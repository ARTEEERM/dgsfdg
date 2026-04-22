# dgsfdg
import tkinter as tk
from tkinter import ttk, messagebox
import json
import random
import os
from datetime import datetime

class RandomTaskGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Task Generator")
        self.root.geometry("600x500")

        # Файл для хранения задач и истории
        self.tasks_file = "tasks.json"
        self.tasks = []
        self.history = []
        self.load_data()

        self.setup_ui()

    def setup_ui(self):
        # Панель добавления новой задачи
        add_frame = ttk.LabelFrame(self.root, text="Добавить новую задачу")
        add_frame.pack(pady=10, padx=10, fill="x")

        ttk.Label(add_frame, text="Задача:").pack(side="left", padx=5)
        self.task_entry = ttk.Entry(add_frame, width=40)
        self.task_entry.pack(side="left", padx=5)

        ttk.Label(add_frame, text="Тип:").pack(side="left", padx=5)
        self.type_var = tk.StringVar(value="учёба")
        type_combo = ttk.Combobox(
            add_frame,
            textvariable=self.type_var,
            values=["учёба", "спорт", "работа"],
            state="readonly",
            width=10
        )
        type_combo.pack(side="left", padx=5)

        add_btn = ttk.Button(add_frame, text="Добавить", command=self.add_task)
        add_btn.pack(side="left", padx=5)

        # Кнопка генерации случайной задачи
        gen_frame = ttk.Frame(self.root)
        gen_frame.pack(pady=10)

        gen_btn = ttk.Button(
            gen_frame,
            text="Сгенерировать случайную задачу",
            command=self.generate_random_task
        )
        gen_btn.pack()

        # Поле отображения текущей задачи
        self.current_task_label = ttk.Label(
            self.root,
            text="Здесь появится случайная задача",
            font=("Arial", 12, "bold"),
            wraplength=500
        )
        self.current_task_label.pack(pady=10)

        # Фильтр по типу задачи
        filter_frame = ttk.LabelFrame(self.root, text="Фильтр по типу")
        filter_frame.pack(pady=10, padx=10, fill="x")

        self.filter_var = tk.StringVar(value="все")
        filter_combo = ttk.Combobox(
            filter_frame,
            textvariable=self.filter_var,
            values=["все", "учёба", "спорт", "работа"],
            state="readonly"
        )
        filter_combo.pack(padx=5, pady=5)
        filter_combo.bind("<<ComboboxSelected>>", self.apply_filter)

        # Список истории сгенерированных задач
        history_frame = ttk.LabelFrame(self.root, text="История задач")
        history_frame.pack(pady=10, padx=10, fill="both", expand=True)

        columns = ("task", "type", "timestamp")
        self.tree = ttk.Treeview(history_frame, columns=columns, show="headings", height=12)

        self.tree.heading("task", text="Задача")
        self.tree.heading("type", text="Тип")
        self.tree.heading("timestamp", text="Время")

        self.tree.column("task", width=300)
        self.tree.column("type", width=100)
        self.tree.column("timestamp", width=150)

        scrollbar = ttk.Scrollbar(history_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        self.refresh_history()

    def load_data(self):
        """Загрузка задач и истории из JSON-файла"""
        if os.path.exists(self.tasks_file):
            try:
                with open(self.tasks_file, "r", encoding="utf-8") as f:
                    data = json.load(f)
                    self.tasks = data.get("tasks", [])
                    self.history = data.get("history", [])
            except (json.JSONDecodeError, IOError):
                self.tasks = []
                self.history = []
        else:
            # Предопределённые задачи
            self.tasks = [
                {"task": "Прочитать статью", "type": "учёба"},
                {"task": "Сделать зарядку", "type": "спорт"},
                {"task": "Написать отчёт", "type": "работа"},
                {"task": "Изучить новую тему", "type": "учёба"},
                {"task": "Пробежать 5 км", "type": "спорт"}
            ]
            self.history = []

    def save_data(self):
        """Сохранение задач и истории в JSON-файл"""
        data = {
            "tasks": self.tasks,
            "history": self.history
        }
        with open(self.tasks_file, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)

    def add_task(self):
        """Добавление новой задачи"""
        task_text = self.task_entry.get().strip()
        task_type = self.type_var.get()

        if not task_text:
            messagebox.showerror("Ошибка", "Задача не может быть пустой!")
            return

        new_task = {"task": task_text, "type": task_type}
        self.tasks.append(new_task)
        self.task_entry.delete(0, tk.END)
        self.save_data()
        messagebox.showinfo("Успех", "Задача добавлена!")


    def generate_random_task(self):
        """Генерация случайной задачи"""
        if not self.tasks:
            messagebox.showwarning("Предупреждение", "Нет задач для генерации!")
            return

        selected_type = self.filter_var.get()
        filtered_tasks = self.tasks

        if selected_type != "все":
            filtered_tasks = [t for t in self.tasks if t["type"] == selected_type]

        if not filtered_tasks:
            messagebox.showwarning("Предупреждение", f"Нет задач типа '{selected_type}'!")
            return

        random_task = random.choice(filtered_tasks)
        timestamp = datetime.now().strftime("%H:%M:%S")

        display_text = f"{random_task['task']} ({random_task['type']})"
        self.current_task_label.config(text=display_text)


        # Добавляем в историю
        history_entry = {
            "task": random_task["task"],
            "type": random_task["type"],
            "timestamp": timestamp
        }
        self.history.append(history_entry)
        self.save_data()
        self.refresh_history()

    def apply_filter(self, event=None):
        """Применение фильтра к истории"""
        self.refresh_history()

    def refresh_history(self):
        """Обновление списка истории"""
        for item in self.tree.get_children():
            self.tree.delete(item)

        selected_filter = self.filter_var.get()

        for entry in self.history:

import sqlite3
import tkinter as tk
from tkinter import messagebox, ttk

# =========================
# DATABASE INITIALIZATION
# =========================

conn = sqlite3.connect("students.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER NOT NULL,
    course TEXT NOT NULL
)
""")
conn.commit()


# =========================
# FUNCTIONS
# =========================


def add_student():
    name = name_entry.get().strip()
    age = age_entry.get().strip()
    course = course_entry.get().strip()

    if not name or not age or not course:
        messagebox.showwarning("Warning", "Please fill in all fields.")
        return

    try:
        age = int(age)
    except ValueError:
        messagebox.showerror("Error", "Age must be a valid number.")
        return

    cursor.execute(
        "INSERT INTO students (name, age, course) VALUES (?, ?, ?)",
        (name, age, course),
    )
    conn.commit()

    messagebox.showinfo("Success", "Student added successfully.")
    clear_fields()
    display_students()


def display_students():
    for item in tree.get_children():
        tree.delete(item)

    cursor.execute("SELECT * FROM students")
    students = cursor.fetchall()

    for idx, student in enumerate(students):
        tag = "evenrow" if idx % 2 == 0 else "oddrow"
        tree.insert("", tk.END, values=student, tags=(tag,))


def update_student():
    selected = tree.selection()

    if not selected:
        messagebox.showwarning("Warning", "Please select a student to update.")
        return

    student_id = tree.item(selected[0])["values"][0]

    name = name_entry.get().strip()
    age = age_entry.get().strip()
    course = course_entry.get().strip()

    if not name or not age or not course:
        messagebox.showwarning("Warning", "Please fill in all fields.")
        return

    try:
        age = int(age)
    except ValueError:
        messagebox.showerror("Error", "Age must be a valid number.")
        return

    cursor.execute(
        """
        UPDATE students
        SET name = ?, age = ?, course = ?
        WHERE id = ?
    """,
        (name, age, course, student_id),
    )

    conn.commit()
    messagebox.showinfo("Success", "Student updated successfully.")
    clear_fields()
    display_students()


def delete_student():
    selected = tree.selection()

    if not selected:
        messagebox.showwarning("Warning", "Please select a student to delete.")
        return

    student_id = tree.item(selected[0])["values"][0]

    if messagebox.askyesno(
        "Confirm Delete", "Are you sure you want to delete this student?"
    ):
        cursor.execute("DELETE FROM students WHERE id = ?", (student_id,))
        conn.commit()
        messagebox.showinfo("Success", "Student deleted successfully.")
        clear_fields()
        display_students()


def clear_fields():
    name_entry.delete(0, tk.END)
    age_entry.delete(0, tk.END)
    course_entry.delete(0, tk.END)
    tree.selection_remove(tree.selection())


def select_student(event):
    selected = tree.selection()
    if selected:
        student = tree.item(selected[0])["values"]
        name_entry.delete(0, tk.END)
        age_entry.delete(0, tk.END)
        course_entry.delete(0, tk.END)

        name_entry.insert(0, student[1])
        age_entry.insert(0, student[2])
        course_entry.insert(0, student[3])


def on_closing():
    conn.close()
    root.destroy()


# =========================
# GUI SETUP
# =========================

root = tk.Tk()
root.title("Student Management System")
root.geometry("700x550")
root.configure(bg="#EAF4FB")
root.protocol("WM_DELETE_WINDOW", on_closing)

# Color Scheme
BG_COLOR = "#F5E427"
TITLE_COLOR = "#F52746"
LABEL_COLOR = "#DA27F5"
ENTRY_BG = "#DDDDDD"

ADD_COLOR = "#3827F5"
UPDATE_COLOR = "#F54927"
DELETE_COLOR = "#27DDF5"
CLEAR_COLOR = "#EB27F5"
BUTTON_TEXT = "black"

# Title
tk.Label(
    root,
    text="Student Management System",
    font=("Arial", 20, "bold"),
    bg=BG_COLOR,
    fg=TITLE_COLOR,
).pack(pady=15)

# Input Frame
input_frame = tk.Frame(root, bg="white", bd=2, relief="groove")
input_frame.pack(pady=5, padx=20, fill="x")

# Inputs
fields = [("Name:", 0), ("Age:", 1), ("Course:", 2)]
entries = {}

for label_text, row_idx in fields:
    tk.Label(
        input_frame,
        text=label_text,
        font=("Arial", 11, "bold"),
        bg="white",
        fg=LABEL_COLOR,
    ).grid(row=row_idx, column=0, padx=10, pady=8, sticky="e")

name_entry = tk.Entry(
    input_frame,
    width=35,
    font=("Arial", 11),
    bg=ENTRY_BG,
    fg="#212121",
    relief="solid",
    bd=1,
)
name_entry.grid(row=0, column=1, padx=10, pady=8)

age_entry = tk.Entry(
    input_frame,
    width=35,
    font=("Arial", 11),
    bg=ENTRY_BG,
    fg="#212121",
    relief="solid",
    bd=1,
)
age_entry.grid(row=1, column=1, padx=10, pady=8)

course_entry = tk.Entry(
    input_frame,
    width=35,
    font=("Arial", 11),
    bg=ENTRY_BG,
    fg="#212121",
    relief="solid",
    bd=1,
)
course_entry.grid(row=2, column=1, padx=10, pady=8)

# Button Frame
button_frame = tk.Frame(root, bg=BG_COLOR)
button_frame.pack(pady=15)

buttons = [
    ("Add", ADD_COLOR, "#1E8449", add_student, 0),
    ("Update", UPDATE_COLOR, "#D68910", update_student, 1),
    ("Delete", DELETE_COLOR, "#C0392B", delete_student, 2),
    ("Clear", CLEAR_COLOR, "#626567", clear_fields, 3),
]

for btn_text, bg_col, active_bg, cmd, col in buttons:
    tk.Button(
        button_frame,
        text=btn_text,
        width=12,
        font=("Arial", 10, "bold"),
        bg=bg_col,
        fg=BUTTON_TEXT,
        activebackground=active_bg,
        activeforeground=BUTTON_TEXT,
        relief="flat",
        cursor="hand2",
        command=cmd,
    ).grid(row=0, column=col, padx=5)

# Treeview Style
style = ttk.Style()
style.theme_use("clam")

style.configure(
    "Treeview.Heading",
    background="#1F618D",
    foreground="white",
    font=("Arial", 10, "bold"),
    padding=8,
)

style.configure(
    "Treeview",
    background="white",
    foreground="#212121",
    rowheight=30,
    fieldbackground="white",
    font=("Arial", 10),
)

style.map(
    "Treeview",
    background=[("selected", "#5DADE2")],
    foreground=[("selected", "white")],
)

# Table
tree = ttk.Treeview(
    root, columns=("ID", "Name", "Age", "Course"), show="headings"
)

tree.heading("ID", text="ID")
tree.heading("Name", text="Name")
tree.heading("Age", text="Age")
tree.heading("Course", text="Course")

tree.column("ID", width=60, anchor="center")
tree.column("Name", width=200)
tree.column("Age", width=80, anchor="center")
tree.column("Course", width=220)

tree.tag_configure("evenrow", background="#F4F9FC")
tree.tag_configure("oddrow", background="#D6EAF8")

tree.pack(fill="both", expand=True, padx=20, pady=10)
tree.bind("<<TreeviewSelect>>", select_student)

display_students()
root.mainloop()


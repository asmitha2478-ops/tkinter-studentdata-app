# tkinter-studentdata-app
import tkinter as tk
from tkinter import ttk, messagebox
import csv
import os
 
DATA_FILE = "student.csv" 
USERNAME = "admin"
PASSWORD = "12345"


def data_file():
    if not os.path.exists(DATA_FILE):
        with open(DATA_FILE, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow(["Register No", "Name", "DOB", "Gender", "Semester", "Contact", "Address", "CGPA"])

def read_data():
    data_file()
    with open(DATA_FILE, newline="") as f:
        return list(csv.DictReader(f))

def write_data(data):
    data_file()
    data.sort(key=lambda x: x["Register No"])

    with open(DATA_FILE, "w", newline="") as f:
        fieldnames = ["Register No", "Name", "DOB", "Gender", "Semester", "Contact", "Address", "CGPA"]
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)

# ---------- Login Page ----------
def login_page(): 
    login = tk.Tk()
    login.title("Student Database Login")
    login.geometry("800x500")
    login.configure(bg="#222")

    tk.Label(login, text="Welcome! Kindly login!", fg="white", bg="#222", font=("Arial", 18, "bold")).pack(pady=20)
    tk.Label(login, text="Username", fg="skyblue", bg="#222", font=("Arial", 16, "bold")).pack()
    username_entry = tk.Entry(login)
    username_entry.pack(pady=5)

    tk.Label(login, text="Password", fg="skyblue", bg="#222",font=("Arial", 16, "bold")).pack()
    password_entry = tk.Entry(login, show="*")
    password_entry.pack(pady=5)

    def check_login():
        if username_entry.get() == USERNAME and password_entry.get() == PASSWORD:
            messagebox.showinfo("Success", "Login successful!")
            login.destroy()
            main_menu()
        else:
            messagebox.showerror("Error", "Invalid username or password")

    tk.Button(login, text="Login", bg="blue", fg="white", command=check_login).pack(pady=20)
    login.mainloop()

# ---------- Main Menu ----------
def main_menu():
    root = tk.Tk()
    root.title("Student Database")
    root.geometry("800x500")
    root.configure(bg="#222")

    tk.Label(root, text="Student Database", font=("Arial", 18, "bold"), fg="white", bg="#222").pack(pady=20)

    def open_add():
        root.destroy()
        add_student()

    def open_view():
        root.destroy()
        view_student()

    def open_edit():
        root.destroy()
        edit_student()

    def open_delete():
        root.destroy()
        delete_student()

    tk.Button(root, text="Add Student", bg="blue", fg="white", width=20, command=open_add).pack(pady=10)
    tk.Button(root, text="View Student", bg="navy", fg="white", width=20, command=open_view).pack(pady=10)
    tk.Button(root, text="Edit Student", bg="teal", fg="white", width=20, command=open_edit).pack(pady=10)
    tk.Button(root, text="Delete Student", bg="gray", fg="white", width=20, command=open_delete).pack(pady=10)

    root.mainloop()

# ---------- Add Student ----------
def add_student():
    add = tk.Tk()
    add.title("Add Student")
    add.geometry("800x500")
    add.configure(bg="#222")

    tk.Label(add, text="Add New Student", font=("Arial", 16, "bold"), fg="white", bg="#222").grid(row=0, column=1, pady=20)
    fields = {}

    labels = ["Register No", "Name", "DOB", "Contact", "Address", "CGPA"]
    for i, label in enumerate(labels, 1):
        tk.Label(add, text=label, fg="white", bg="#222").grid(row=i, column=0, padx=10, pady=5, sticky="w")
        entry = tk.Entry(add, width=30)
        entry.grid(row=i, column=1, pady=5)
        fields[label] = entry
    
    tk.Label(add, text="Gender", fg="white", bg="#222").grid(row=7, column=0, padx=10, pady=5, sticky="w")
    gender_var = tk.StringVar()
    gender_dropdown = ttk.Combobox(add, textvariable=gender_var, values=["Male", "Female"], width=27, state="readonly")
    gender_dropdown.grid(row=7, column=1, pady=5)
    gender_dropdown.current(0)

    tk.Label(add, text="Semester", fg="white", bg="#222").grid(row=8, column=0, padx=10, pady=5, sticky="w")
    sem_var = tk.StringVar()
    sem_dropdown = ttk.Combobox(add, textvariable=sem_var, values=["1st","2nd","3rd","4th","5th","6th","7th","8th"], width=27, state="readonly")
    sem_dropdown.grid(row=8, column=1, pady=5)
    sem_dropdown.current(0)

    def save_student():
        data = read_data()
        reg = fields["Register No"].get().strip()
        if not reg:
            messagebox.showerror("Error", "Register No required")
            return
        for s in data:
            if s["Register No"] == reg:
                messagebox.showerror("Error", "Register No already exists")
                return

        new_student = {k: v.get() for k, v in fields.items()}
        new_student["Gender"] = gender_var.get()
        new_student["Semester"] = sem_var.get()

        data.append(new_student)
        write_data(data)
        messagebox.showinfo("Success", "Student added successfully!")
        add.destroy()
        main_menu()

    tk.Button(add, text="Save Details", bg="blue", fg="white", command=save_student).grid(row=10, column=1, pady=20)
    add.mainloop()

# ---------- View Student ----------
def view_student():
    view = tk.Tk()
    view.title("View Student")
    view.geometry("800x500")
    view.configure(bg="#222")

    tk.Label(view, text="Enter Register No to Search:", fg="white", bg="#222").pack(pady=10)
    reg_entry = tk.Entry(view)
    reg_entry.pack()

    output = tk.Label(view, text="", fg="white", bg="#222", justify="left", font=("Courier", 10))
    output.pack(pady=20)

    def search_student():
        reg = reg_entry.get().strip()
        data = read_data()
        for s in data:
            if s["Register No"] == reg:
                details = "\n".join([f"{k}: {v}" for k, v in s.items()])
                output.config(text=details)
                return
        output.config(text="Student not found!")

    tk.Button(view, text="Search Details", bg="blue", fg="white", command=search_student).pack(pady=10)
    tk.Button(view, text="Back", bg="gray", fg="white", command=lambda: [view.destroy(), main_menu()]).pack()
    view.mainloop()

# ---------- Edit Student ----------
def edit_student():
    edit = tk.Tk()
    edit.title("Edit Student")
    edit.geometry("600x400")
    edit.configure(bg="#222")

    tk.Label(edit, text="Enter Register No to Edit:", fg="white", bg="#222").grid(row=0, column=0, pady=10)
    reg_entry = tk.Entry(edit)
    reg_entry.grid(row=0, column=1, pady=10)

    fields = {}
    labels = ["Name", "DOB", "Contact", "Address", "CGPA"]
    for i, label in enumerate(labels, 1):
        tk.Label(edit, text=label, fg="white", bg="#222").grid(row=i, column=0, padx=10, pady=5, sticky="w")
        entry = tk.Entry(edit, width=30)
        entry.grid(row=i, column=1, pady=5)
        fields[label] = entry

    tk.Label(edit, text="Gender", fg="white", bg="#222").grid(row=7, column=0, padx=10, pady=5, sticky="w")
    gender_var = tk.StringVar()
    gender_dropdown = ttk.Combobox(edit, textvariable=gender_var, values=["Male", "Female"], width=27, state="readonly")
    gender_dropdown.grid(row=7, column=1, pady=5)

    tk.Label(edit, text="Semester", fg="white", bg="#222").grid(row=8, column=0, padx=10, pady=5, sticky="w")
    sem_var = tk.StringVar()
    sem_dropdown = ttk.Combobox(edit, textvariable=sem_var, values=["1st","2nd","3rd","4th","5th","6th","7th","8th"], width=27, state="readonly")
    sem_dropdown.grid(row=8, column=1, pady=5)

    def load_student():
        reg = reg_entry.get().strip()
        data = read_data()
        for s in data:
            if s["Register No"] == reg:
                for k in fields:
                    fields[k].delete(0, tk.END)
                    fields[k].insert(0, s[k])
                gender_var.set(s["Gender"])
                sem_var.set(s["Semester"])
                return
        messagebox.showerror("Error", "Register No not found")

    def update_student():
        reg = reg_entry.get().strip()
        data = read_data()
        for s in data:
            if s["Register No"] == reg:
                for k in fields:
                    s[k] = fields[k].get()
                s["Gender"] = gender_var.get()
                s["Semester"] = sem_var.get()
                write_data(data)
                messagebox.showinfo("Success", "Student updated successfully!")
                edit.destroy()
                main_menu()
                return
        messagebox.showerror("Error", "Register No not found")

    tk.Button(edit, text="Load Data", bg="navy", fg="white", command=load_student).grid(row=0, column=2, padx=10)
    tk.Button(edit, text="Update Student", bg="blue", fg="white", command=update_student).grid(row=10, column=1, pady=20)
    edit.mainloop()

# ---------- Delete Student ----------
def delete_student():
    delete = tk.Tk()
    delete.title("Delete Student")
    delete.geometry("800x500")
    delete.configure(bg="#222")

    tk.Label(delete, text="Enter Register No to Delete:", fg="white", bg="#222").pack(pady=10)
    reg_entry = tk.Entry(delete)
    reg_entry.pack()

    def confirm_delete():
        reg = reg_entry.get().strip()
        data = read_data()
        for s in data:
            if s["Register No"] == reg:
                if messagebox.askyesno("Confirm Delete", f"Are you sure to delete Register No: {reg}?"):
                    data.remove(s)
                    write_data(data)
                    messagebox.showinfo("Deleted", "Student deleted successfully")
                    delete.destroy()
                    main_menu()
                    return
        messagebox.showerror("Error", "Register No not found")

    tk.Button(delete, text="Delete", bg="blue", fg="white", command=confirm_delete).pack(pady=20)
    tk.Button(delete, text="Back", bg="gray", fg="white", command=lambda: [delete.destroy(), main_menu()]).pack()
    delete.mainloop()

data_file()
login_page()

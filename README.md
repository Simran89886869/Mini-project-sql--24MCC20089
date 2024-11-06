# Mini-project-sql--24MCC20089
import sqlite3
from datetime import datetime
import tkinter as tk
from tkinter import messagebox, font

# Database connection
def connect_db():
    return sqlite3.connect('attendance.db')

# Function to clear all data on startup (both students and attendance)
def clear_all_data_on_startup():
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('DELETE FROM student_records')  # Clear student records
    cursor.execute('DELETE FROM attendance_records')  # Clear attendance records
    conn.commit()
    conn.close()

# Functions for database operations
def add_student(student_id, name):
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('SELECT id FROM student_records WHERE id = ?', (student_id,))
    result = cursor.fetchone()
    if result:
        messagebox.showerror("Error", "Student ID already exists. Please enter a unique ID.")
    else:
        cursor.execute('INSERT INTO student_records (id, name) VALUES (?, ?)', (student_id, name))
        conn.commit()
        messagebox.showinfo("Success", f"Student '{name}' added successfully!")
    conn.close()

def mark_attendance(student_id, status):
    conn = connect_db()
    cursor = conn.cursor()
    date = datetime.now().strftime('%Y-%m-%d')
    cursor.execute('INSERT INTO attendance_records (student_id, date, status) VALUES (?, ?, ?)', (student_id, date, status))
    conn.commit()
    conn.close()

def view_attendance():
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('''
        SELECT student_records.name, attendance_records.date, attendance_records.status
        FROM attendance_records
        JOIN student_records ON attendance_records.student_id = student_records.id
        ORDER BY attendance_records.date DESC
    ''')
    records = cursor.fetchall()
    conn.close()
    return records

# GUI Application Class
class AttendanceApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Attendance System")
        self.root.configure(bg="#f0f0f5")
        self.root.geometry("800x600")

        # Title Label
        title_font = font.Font(family="Helvetica", size=18, weight="bold")
        self.title_label = tk.Label(self.root, text="Attendance Management System", font=title_font, bg="#f0f0f5", fg="#333399")
        self.title_label.pack(pady=15)

        # Add Student Section
        self.add_student_frame = tk.LabelFrame(self.root, text="Add Student", font=("Arial", 12, "bold"), fg="#333399", bg="#e6e6fa", padx=20, pady=20)
        self.add_student_frame.pack(pady=10, fill="x", padx=20)

        self.student_id_label = tk.Label(self.add_student_frame, text="Student ID:", bg="#e6e6fa", font=("Arial", 10))
        self.student_id_label.grid(row=0, column=0, padx=5, pady=5, sticky="e")

        self.student_id_entry = tk.Entry(self.add_student_frame, bg="#FFEECC", font=("Arial", 10))
        self.student_id_entry.grid(row=0, column=1, padx=5, pady=5)

        self.student_name_label = tk.Label(self.add_student_frame, text="Student Name:", bg="#e6e6fa", font=("Arial", 10))
        self.student_name_label.grid(row=0, column=2, padx=5, pady=5, sticky="e")

        self.student_name_entry = tk.Entry(self.add_student_frame, bg="#FFEECC", font=("Arial", 10))
        self.student_name_entry.grid(row=0, column=3, padx=5, pady=5)

        self.add_student_button = tk.Button(self.add_student_frame, text="Add Student", command=self.add_student, bg="#4caf50", fg="white", font=("Arial", 10, "bold"))
        self.add_student_button.grid(row=0, column=4, padx=5, pady=5)

        # Mark Attendance Section
        self.mark_attendance_frame = tk.LabelFrame(self.root, text="Mark Attendance", font=("Arial", 12, "bold"), fg="#333399", bg="#e6e6fa", padx=20, pady=20)
        self.mark_attendance_frame.pack(pady=10, fill="x", padx=20)

        self.mark_student_id_label = tk.Label(self.mark_attendance_frame, text="Student ID:", bg="#e6e6fa", font=("Arial", 10))
        self.mark_student_id_label.grid(row=0, column=0, padx=5, pady=5, sticky="e")

        self.mark_student_id_entry = tk.Entry(self.mark_attendance_frame, bg="#FFEECC", font=("Arial", 10))
        self.mark_student_id_entry.grid(row=0, column=1, padx=5, pady=5)

        self.status_label = tk.Label(self.mark_attendance_frame, text="Status:", bg="#e6e6fa", font=("Arial", 10))
        self.status_label.grid(row=0, column=2, padx=5, pady=5, sticky="e")

        self.status_combobox = tk.StringVar()
        self.status_combobox.set("Present")  # Default value
        self.status_options = tk.OptionMenu(self.mark_attendance_frame, self.status_combobox, "Present", "Absent")
        self.status_options.config(bg="#FFEECC", font=("Arial", 10))
        self.status_options.grid(row=0, column=3, padx=5, pady=5)

        self.mark_attendance_button = tk.Button(self.mark_attendance_frame, text="Mark Attendance", command=self.mark_attendance, bg="#2196f3", fg="white", font=("Arial", 10, "bold"))
        self.mark_attendance_button.grid(row=0, column=4, padx=5, pady=5)

        # View Attendance Section
        self.view_attendance_frame = tk.LabelFrame(self.root, text="View Attendance", font=("Arial", 12, "bold"), fg="#333399", bg="#e6e6fa", padx=20, pady=20)
        self.view_attendance_frame.pack(pady=10, fill="x", padx=20)

        self.view_button = tk.Button(self.view_attendance_frame, text="View Attendance", command=self.view_attendance, bg="#FF8C00", fg="white", font=("Arial", 10, "bold"))
        self.view_button.grid(row=0, column=0, padx=5, pady=5)

        self.clear_button = tk.Button(self.view_attendance_frame, text="Clear Attendance", command=self.clear_attendance, bg="#f44336", fg="white", font=("Arial", 10, "bold"))
        self.clear_button.grid(row=0, column=1, padx=5, pady=5)

        # Canvas for Attendance Records
        self.canvas = tk.Canvas(self.view_attendance_frame, width=700, height=250, bg="white")
        self.canvas.grid(row=1, column=0, columnspan=2, padx=5, pady=10)
        self.canvas.bind("<Configure>", self.draw_table)

        self.attendance_data = []

    def add_student(self):
        student_id = self.student_id_entry.get()
        name = self.student_name_entry.get()
        if student_id and name:
            add_student(student_id, name)
            self.student_id_entry.delete(0, tk.END)
            self.student_name_entry.delete(0, tk.END)
        else:
            messagebox.showerror("Error", "Please enter both Student ID and Name.")

    def mark_attendance(self):
        student_id = self.mark_student_id_entry.get()
        status = self.status_combobox.get()
        if student_id and status:
            mark_attendance(student_id, status)
            messagebox.showinfo("Success", "Attendance marked successfully!")
            self.mark_student_id_entry.delete(0, tk.END)
            self.status_combobox.set('Present')  # Reset to default
        else:
            messagebox.showerror("Error", "Please enter both Student ID and Status.")

    def view_attendance(self):
        records = view_attendance()
        self.attendance_data = records  # Store records for drawing
        self.draw_table()  # Draw the table after fetching records

    def clear_attendance(self):
        clear_all_data_on_startup()
        self.attendance_data.clear()  # Clear stored data
        self.canvas.delete("all")  # Clear canvas
        messagebox.showinfo("Success", "All attendance records cleared.")

    def draw_table(self, event=None):
        self.canvas.delete("all")  # Clear previous drawings
        row_height = 30

        # Header Row
        header_color = "#D3D3D3"
        self.canvas.create_rectangle(0, 0, 700, row_height, fill=header_color, outline="")
        self.canvas.create_text(10, 15, anchor="w", text="Student Name", fill="black", font=("Arial", 10, "bold"))
        self.canvas.create_text(200, 15, anchor="w", text="Date", fill="black", font=("Arial", 10, "bold"))
        self.canvas.create_text(400, 15, anchor="w", text="Status", fill="black", font=("Arial", 10, "bold"))

        # Data Rows
        for i, record in enumerate(self.attendance_data):
            y_position = (i + 1) * row_height
            self.canvas.create_text(10, y_position + 15, anchor="w", text=record[0], fill="black", font=("Arial", 10))
            self.canvas.create_text(200, y_position + 15, anchor="w", text=record[1], fill="black", font=("Arial", 10))
            self.canvas.create_text(400, y_position + 15, anchor="w", text=record[2], fill="black", font=("Arial", 10))

if __name__ == "__main__":
    clear_all_data_on_startup()  # Clear data on startup
    root = tk.Tk()
    app = AttendanceApp(root)
    root.mainloop()

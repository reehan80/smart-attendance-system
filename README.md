# smart-attendance-system

Step-By-Step Explanation:

Step 1: Import Required Libraries 

➢ The first step is to import all the necessary libraries that will be used in the code for various 
functionalities such as image processing, face recognition, and to store the attendance.
import cv2
import numpy as np
import face_recognition
import os
from datetime import datetime
from openpyxl import Workbook, load_workbook
import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
import pandas as pd
import pyttsx3

Step 2:Setup Face Encoding:

➢ Create a folder as images which contains the images of the students
path = 'images'
images = []
classnames = []
mylist = os.listdir('C:/Users/darsh/PycharmProjects/pythonProject/images')
for cl in mylist:
 curImg = 
cv2.imread(f'C:/Users/darsh/PycharmProjects/pythonProject/images/mubarak.jpg'
)
 images.append(curImg)
 classnames.append(os.path.splitext(cl)[0])
# Function to encode known images
def findEncodings(images): encodeList = []
 for img in images:
 img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
 encodings = face_recognition.face_encodings(img)
 if encodings:
 encodeList.append(encodings[0])
 return encodeList
encodeListKnown = findEncodings(images)
print('[INFO] Encoding Complete')
Step 3:Setup Exel Worbook:
➢ Next, setup an exel sheet in which we can store the attendance by recognising the faces of the 
students.
def markAttendance(name):
 file_name = 'Attendance.xlsx'
 if os.path.exists(file_name):
 workbook = load_workbook(file_name)
 sheet = workbook.active
 else:
 workbook = Workbook()
 sheet = workbook.active
 sheet.append(["Name", "Date", "Time"])
 now = datetime.now()
 date = now.strftime('%Y-%m-%d')
 time = now.strftime('%H:%M:%S')
 names_in_sheet = [cell.value for cell in sheet['A'] if cell.value != 
"Name"]
 if name not in names_in_sheet:
 sheet.append([name, date, time])
 workbook.save(file_name)
 print(f"[INFO] Attendance marked for {name}") else:
 print(f"[INFO] {name} already marked today.")
 
Step 3: Setup Text-To-Speech:

➢ Here, we have written some text which will speak when the attendance is taken and also for if a 
unknown person stand infront of webcam.
engine = pyttsx3.init()
def speak(text):
 engine.say(text)
 engine.runAndWait()
 
Step 4: Smart Attendance Logic :

➢ Logic for the smart attendance using face recognition.
def smart_attendance():
 cap = cv2.VideoCapture(0)
 already_marked = set()
 while True:
 success, img = cap.read()
 if not success:
 break
 imgS = cv2.resize(img, (0, 0), None, 0.25, 0.25)
 imgS = cv2.cvtColor(imgS, cv2.COLOR_BGR2RGB)
 facesCurFrame = face_recognition.face_locations(imgS)
 encodesCurFrame = face_recognition.face_encodings(imgS, 
facesCurFrame)
 for encodeFace, faceLoc in zip(encodesCurFrame, facesCurFrame):
 matches = face_recognition.compare_faces(encodeListKnown, 
encodeFace) faceDis = face_recognition.face_distance(encodeListKnown, 
encodeFace)
 matchIndex = np.argmin(faceDis)
 y1, x2, y2, x1 = faceLoc
 y1, x2, y2, x1 = y1*4, x2*4, y2*4, x1*4
 if matches[matchIndex] and faceDis[matchIndex] < 0.6:
 name = classnames[matchIndex].upper()
 cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), 2)
 cv2.putText(img, name, (x1, y1-10), cv2.FONT_HERSHEY_SIMPLEX, 
1, (255, 255, 255), 2)
 if name not in already_marked:
 markAttendance(name)
 already_marked.add(name)
 speak(f"Attendance marked for {name}")
 else:
 cv2.rectangle(img, (x1, y1), (x2, y2), (0, 0, 255), 2)
 cv2.putText(img, "New Student Detected", (x1, y1-10), 
cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
 speak("Unknown person detected. Please register and try 
again.")
 cv2.imshow('Webcam - Press Q to Exit', img)
 if cv2.waitKey(1) & 0xFF == ord('q'):
 break
 cap.release()
 cv2.destroyAllWindows()
 
 Step 5: Setup Professional Tkinter GUI :
 
➢ It is used to display a popup box for starting and viewing attendance.
def main_gui():
 window = tk.Tk()
 window.title("Smart Attendance System")
 window.configure(bg="#f0f0f0")
 window.geometry("400x300")
 window.resizable(False, False)
 # Center window
 window.update_idletasks()
 width = window.winfo_width()
 height = window.winfo_height()
 x = (window.winfo_screenwidth() // 2) - (width // 2)
 y = (window.winfo_screenheight() // 2) - (height // 2)
 window.geometry(f'{width}x{height}+{x}+{y}')
 # Title label
 title_label = tk.Label(window, text="Smart Attendance System", 
font=("Helvetica", 16, "bold"), bg="#f0f0f0", fg="#333")
 title_label.pack(pady=20)
 # Style buttons
 style = ttk.Style()
 style.configure("TButton", font=("Helvetica", 12), padding=10)
 def open_attendance():
 try:
 df = pd.read_excel('Attendance.xlsx')
 messagebox.showinfo("Attendance Sheet", 
df.to_string(index=False))
 except FileNotFoundError:
 messagebox.showerror("Error", "Attendance file not found.") ttk.Button(window, text="Start Smart Attendance", 
command=smart_attendance).pack(pady=10)
 ttk.Button(window, text="View Attendance", 
command=open_attendance).pack(pady=10)
 ttk.Button(window, text="Exit", command=window.destroy).pack(pady=10)
 window.mainloop()
 
Step 6: Run Main GUI:

➢ Here, we are running the GUI .
main_gui()

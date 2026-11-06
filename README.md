# ============================================================
# FRESHMEN BSIT STUDENT ATTENDANCE MONITORING
# PHINMA Cagayan de Oro City
# ============================================================
# Beginner-friendly version using basic Python only.
# Note: Records stay in memory and reset when the program closes.
from datetime import datetime, timedelta

students = []
attendance_records = []
active_session = []

def show_line():
    print("------------------------------------------------------------")

def ask_required(prompt):
    value = input(prompt).strip()
    while value == "":
        print("This field cannot be empty. Please try again.")
        value = input(prompt).strip()
    return value

def find_student(student_id, student_list):
    result = -1
    for num in range(len(student_list)):
        if student_list[num][0] == student_id:
            result = num
    return result

def find_attendance(student_id, subject, attendance_date, record_list):
    result = -1
    for num in range(len(record_list)):
        if (record_list[num][0] == student_id and
            record_list[num][1].lower() == subject.lower() and
            record_list[num][2] == attendance_date):
            result = num
    return result

def show_student_details(student):
    show_line()
    print("Student ID :", student[0])
    print("Full Name  :", student[1])
    print("Course     :", student[2])
    print("Year Level :", student[3])
    print("Section    :", student[4])
    show_line()

def show_students(student_list):
    print("\n==================== STUDENT LIST ====================")
    if len(student_list) == 0:
        print("No students registered yet.")
    else:
        print("Total Students:", len(student_list))
        for num in range(len(student_list)):
            print("\nStudent Number:", num + 1)
            show_student_details(student_list[num])

def add_student(student_list):
    print("\n================ REGISTER NEW STUDENT ================")
    student_id = ask_required("Enter Student ID: ")
    position = find_student(student_id, student_list)
    if position != -1:
        print("That Student ID is already registered.")
    else:
        name = ask_required("Enter Full Name: ")
        course = "BSIT"
        year_level = "1st Year"
        section = ask_required("Enter Section: ")
        student_list.append([student_id, name, course, year_level, section])
        print("Freshman student registered successfully.")
        show_student_details(student_list[len(student_list) - 1])

def search_student(student_list):
    print("\n==================== SEARCH STUDENT ====================")
    student_id = ask_required("Enter Student ID: ")
    position = find_student(student_id, student_list)
    if position == -1:
        print("Student not found. Check the Student ID and try again.")
    else:
        print("Student found.")
        show_student_details(student_list[position])

def edit_student(student_list):
    print("\n===================== EDIT STUDENT =====================")
    student_id = ask_required("Enter Student ID to edit: ")
    position = find_student(student_id, student_list)
    if position == -1:
        print("Student not found.")
    else:
        student = student_list[position]
        show_student_details(student)
        print("Press Enter to keep the current value.")
        name = input("Full Name [" + student[1] + "]: ").strip()
        section = input("Section [" + student[4] + "]: ").strip()
        if name != "":
            student[1] = name
        if section != "":
            student[4] = section
        print("Student information updated successfully.")
        show_student_details(student)

def close_expired_session(student_list, record_list):
    if len(active_session) == 0:
        return
    if datetime.now() >= active_session[3]:
        subject = active_session[0]
        attendance_date = active_session[1]
        for student in student_list:
            if find_attendance(student[0], subject, attendance_date, record_list) == -1:
                record_list.append([student[0], subject, attendance_date, "Absent", "Automatically marked absent after the attendance time limit."])
        print("\nAttendance time limit has ended.")
        print("Students who did not check in were marked Absent.")
        active_session.clear()

def start_attendance_session():
    print("\n================ START ATTENDANCE SESSION ================")
    subject = ask_required("Enter Subject Name: ")
    attendance_date = ask_required("Enter Date (YYYY-MM-DD): ")
    while True:
        duration_text = input("Enter attendance time limit in minutes: ").strip()
        if duration_text.isdigit() and int(duration_text) > 0:
            duration = int(duration_text)
            break
        print("Please enter a whole number greater than zero.")
    start_time = datetime.now()
    end_time = start_time + timedelta(minutes=duration)
    active_session.clear()
    active_session.append(subject)
    active_session.append(attendance_date)
    active_session.append(start_time)
    active_session.append(end_time)
    print("\nAttendance session created.")
    print("Subject :", subject)
    print("Date    :", attendance_date)
    print("Start   :", start_time.strftime("%H:%M:%S"))
    print("Deadline:", end_time.strftime("%H:%M:%S"))
    print("Time Limit:", duration, "minute(s)")
    print("Students can now enter ONLY their Student ID.")
    print("Unchecked students will be marked Absent after the deadline.")

def show_active_session():
    if len(active_session) == 0:
        print("No active attendance session.")
    else:
        close_expired_session(students, attendance_records)
        if len(active_session) > 0:
            print("\nActive Attendance Session")
            print("Subject :", active_session[0])
            print("Date    :", active_session[1])
            print("Start   :", active_session[2].strftime("%H:%M:%S"))
            print("Deadline:", active_session[3].strftime("%H:%M:%S"))
            remaining = active_session[3] - datetime.now()
            print("Time remaining:", str(remaining).split(".")[0])

def student_check_in(student_list, record_list):
    close_expired_session(student_list, record_list)
    print("\n================ AUTOMATIC ATTENDANCE =================")
    if len(active_session) == 0:
        print("No attendance session is currently open.")
        return
    if datetime.now() < active_session[2]:
        print("Attendance session has not started yet.")
        return
    if datetime.now() >= active_session[3]:
        close_expired_session(student_list, record_list)
        print("Attendance time limit has ended. Check-in is closed.")
        return
    student_id = ask_required("Enter Student ID: ")
    if datetime.now() >= active_session[3]:
        close_expired_session(student_list, record_list)
        print("Time limit ended before check-in could be recorded.")
        return
    student_position = find_student(student_id, student_list)
    if student_position == -1:
        print("Student ID not found. Attendance was not recorded.")
        return
    subject = active_session[0]
    attendance_date = active_session[1]
    if find_attendance(student_id, subject, attendance_date, record_list) != -1:
        print("Attendance already recorded for this student and session.")
        return
    record_list.append([student_id, subject, attendance_date, "Present", "Automatic ID check-in at " + datetime.now().strftime("%H:%M:%S")])
    print("Attendance recorded successfully.")
    print("Student :", student_list[student_position][1])
    print("Status  : Present")
    print("Time    :", datetime.now().strftime("%H:%M:%S"))

def choose_status():
    print("\nChoose Attendance Status:")
    print("1. Present")
    print("2. Absent")
    print("3. Excuse")
    print("4. Suspended Classes")
    choice = input("Enter status number: ").strip()
    if choice == "1":
        result = "Present"
    elif choice == "2":
        result = "Absent"
    elif choice == "3":
        result = "Excuse"
    elif choice == "4":
        result = "Suspended Classes"
    else:
        result = ""
    return result

def record_attendance(student_list, record_list):
    print("\n================ MANUAL ATTENDANCE ===================")
    print("Use this for teacher-entered attendance or exceptions.")
    student_id = ask_required("Enter Student ID: ")
    position = find_student(student_id, student_list)
    if position == -1:
        print("Student not found. Register the student first.")
        return
    subject = ask_required("Enter Subject Name: ")
    attendance_date = ask_required("Enter Date (YYYY-MM-DD): ")
    if find_attendance(student_id, subject, attendance_date, record_list) != -1:
        print("Attendance already exists for this student, subject, and date.")
        return
    status = choose_status()
    if status == "":
        print("Invalid status. Attendance was not recorded.")
        return
    remarks = input("Enter Remarks (press Enter if none): ").strip()
    record_list.append([student_id, subject, attendance_date, status, remarks])
    print("Attendance recorded successfully.")

def show_all_attendance(record_list):
    print("\n================ ALL ATTENDANCE RECORDS ================")
    if len(record_list) == 0:
        print("No attendance records available.")
    else:
        print("Total Attendance Records:", len(record_list))
        for num in range(len(record_list)):
            record = record_list[num]
            print("\nRecord Number:", num + 1)
            print("Student ID :", record[0])
            print("Subject    :", record[1])
            print("Date       :", record[2])
            print("Status     :", record[3])
            if record[4] == "":
                print("Remarks    : None")
            else:
                print("Remarks    :", record[4])
            show_line()

def student_attendance_history(student_list, record_list):
    print("\n============== STUDENT ATTENDANCE HISTORY ==============")
    student_id = ask_required("Enter Student ID: ")
    position = find_student(student_id, student_list)
    if position == -1:
        print("Student not found.")
    else:
        print("Attendance History for:", student_list[position][1])
        found = 0
        for record in record_list:
            if record[0] == student_id:
                print("Subject:", record[1], "| Date:", record[2],
                      "| Status:", record[3], "| Remarks:", record[4])
                found = found + 1
        if found == 0:
            print("No attendance records found for this student.")
        else:
            print("Total records found:", found)

def search_attendance_by_date(record_list):
    print("\n================ SEARCH ATTENDANCE BY DATE ================")
    attendance_date = ask_required("Enter Date (YYYY-MM-DD): ")
    found = 0
    for record in record_list:
        if record[2] == attendance_date:
            print("Student ID:", record[0], "| Subject:", record[1],
                  "| Status:", record[3], "| Remarks:", record[4])
            found = found + 1
    if found == 0:
        print("No attendance records found for that date.")
    else:
        print("Total records for", attendance_date, ":", found)

def attendance_summary(student_list, record_list):
    print("\n================ STUDENT ATTENDANCE SUMMARY ================")
    student_id = ask_required("Enter Student ID: ")
    position = find_student(student_id, student_list)
    if position == -1:
        print("Student not found.")
    else:
        present = 0
        absent = 0
        excuse = 0
        suspended = 0
        total = 0
        for record in record_list:
            if record[0] == student_id:
                total = total + 1
                if record[3] == "Present":
                    present = present + 1
                elif record[3] == "Absent":
                    absent = absent + 1
                elif record[3] == "Excuse":
                    excuse = excuse + 1
                elif record[3] == "Suspended Classes":
                    suspended = suspended + 1
        print("Student Name       :", student_list[position][1])
        print("Student ID         :", student_id)
        print("Total Records      :", total)
        print("Present            :", present)
        print("Absent             :", absent)
        print("Excuse             :", excuse)
        print("Suspended Classes  :", suspended)
        if total > 0:
            result = present / total * 100
            print("Present Percentage :", round(result, 2), "%")
        else:
            print("No attendance records to calculate a percentage.")

def all_students_summary(student_list, record_list):
    print("\n================ SUMMARY OF ALL STUDENTS ================")
    if len(student_list) == 0:
        print("No students registered yet.")
    else:
        for student in student_list:
            present = 0
            absent = 0
            excuse = 0
            suspended = 0
            total = 0
            for record in record_list:
                if record[0] == student[0]:
                    total = total + 1
                    if record[3] == "Present":
                        present = present + 1
                    elif record[3] == "Absent":
                        absent = absent + 1
                    elif record[3] == "Excuse":
                        excuse = excuse + 1
                    elif record[3] == "Suspended Classes":
                        suspended = suspended + 1
            print("\nStudent:", student[1], "| ID:", student[0])
            print("Total:", total, "| Present:", present, "| Absent:", absent,
                  "| Excuse:", excuse, "| Suspended:", suspended)
            if total > 0:
                result = present / total * 100
                print("Present Percentage:", round(result, 2), "%")
            else:
                print("Present Percentage: No records")

def report_menu(student_list, record_list):
    choice = ""
    while choice != "6":
        print("\n================ ATTENDANCE REPORTS ================")
        print("1. View All Attendance Records")
        print("2. View One Student's Attendance Summary")
        print("3. View Student Attendance History")
        print("4. Search Attendance by Date")
        print("5. View Summary of All Students")
        print("6. Return to Teacher Menu")
        choice = input("Choose an option: ").strip()
        if choice == "1":
            show_all_attendance(record_list)
        elif choice == "2":
            attendance_summary(student_list, record_list)
        elif choice == "3":
            student_attendance_history(student_list, record_list)
        elif choice == "4":
            search_attendance_by_date(record_list)
        elif choice == "5":
            all_students_summary(student_list, record_list)
        elif choice == "6":
            print("Returning to Teacher Menu.")
        else:
            print("Invalid option. Please choose from 1 to 6.")

def teacher_login(student_list, record_list):
    print("\n===================== TEACHER LOGIN =====================")
    teacher_id = ask_required("Enter Teacher ID (TCH-001 to TCH-010): ")
    valid_teacher = False
    for num in range(1, 11):
        if teacher_id == "TCH-00" + str(num):
            valid_teacher = True
    if valid_teacher:
        print("Teacher login successful.")
        teacher_menu(student_list, record_list)
    else:
        print("Invalid Teacher ID. Access denied.")

def teacher_menu(student_list, record_list):
    choice = ""
    while choice != "9":
        close_expired_session(student_list, record_list)
        print("\n==================== TEACHER MENU ====================")
        print("1. Register Student")
        print("2. View All Students")
        print("3. Search Student")
        print("4. Edit Student Information")
        print("5. Start Attendance Session")
        print("6. Manual Attendance / Exceptions")
        print("7. Attendance Reports")
        print("8. View Active Session")
        print("9. Return to Main Menu")
        choice = input("Choose an option: ").strip()
        if choice == "1":
            add_student(student_list)
        elif choice == "2":
            show_students(student_list)
        elif choice == "3":
            search_student(student_list)
        elif choice == "4":
            edit_student(student_list)
        elif choice == "5":
            start_attendance_session()
        elif choice == "6":
            record_attendance(student_list, record_list)
        elif choice == "7":
            report_menu(student_list, record_list)
        elif choice == "8":
            show_active_session()
        elif choice == "9":
            print("Returning to Main Menu.")
        else:
            print("Invalid option. Please choose from 1 to 9.")

def student_menu(student_list, record_list):
    student_check_in(student_list, record_list)

def main():
    choice = ""
    while choice != "3":
        print("\n============================================================")
        print("       FRESHMEN BSIT STUDENT ATTENDANCE MONITORING")
        print("              PHINMA Cagayan de Oro City")
        print("============================================================")
        print("1. Teacher Login")
        print("2. Student Menu")
        print("3. Exit System")
        choice = input("Choose an option: ").strip()
        if choice == "1":
            teacher_login(students, attendance_records)
        elif choice == "2":
            student_menu(students, attendance_records)
        elif choice == "3":
            print("\nThank you for using the Attendance Monitoring System.")
            print("Program closed.")
        else:
            print("Invalid option. Please choose 1, 2, or 3.")

main()

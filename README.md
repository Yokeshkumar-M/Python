# Python
# Badminton club management system

import sqlite3
import os
import csv
import re as reg
from sqlite3 import Error
from datetime import datetime
import tkinter as tk
from tkinter import *

def sql_connection():

    try:
        print("Badminton Club Membership Management System")
        print(" Database name : club_database")
        conn = sqlite3.connect('club_database.db')
        return conn
    except Error:
        print(Error)

def sql_table(conn):

    cursorObj = conn.cursor()

    columnNames = []
    checktable = cursorObj.execute(f"SELECT name FROM sqlite_schema WHERE type='table' AND name='club_table';").fetchall()
    if checktable == []:
        cursorObj.execute(f"""CREATE TABLE club_table(  MEMBERSHIP_ID INTEGER PRIMARY KEY AUTOINCREMENT , 
                                                        NAME char(30) NOT NULL, 
                                                        PHONE_NUMBER int(15) NOT NULL, 
                                                        AGE int(3) NOT NULL,
                                                        GENDER char(10) NOT NULL,
                                                        DATE_OF_BIRTH char(15) NOT NULL,
                                                        DATE_JOINED char(20) );""")
    print("Database table created successfully.")

    data = cursorObj.execute(f"SELECT * FROM club_table")
    for column in data.description:
        columnNames.append(column[0])

    def name():

        print("Enter name :")
        name_input = input()

        if reg.fullmatch('[A-Za-z]{2,25}( [A-Za-z]{2,25})?', name_input):
            return name_input
        else:
            print("Enter valid name..")
            name()

    def phone_no():

        print("Enter phone number :")
        phone_no_input = input()

        if reg.fullmatch('(0|91)?[6-9][0-9]{9}',phone_no_input):
            return phone_no_input
        else:
            print("Enter valid phone number..")
            phone_no()

    def age():

        print("Enter age :")
        try:
            age_input = int(input())
            if 12 <= age_input <= 60:
                return age_input
            else:
                print("Invalid age..Enter between 12 and 60..")
                age()
        except:
            print("Invalid age input")
            age()

    def gender():

        global gender_data
        print("Gender => [ MALE --> m , FEMALE --> f ]")
        print("Enter choice..")
        gender_input = input()

        if gender_input == "m":
            gender_data = 'MALE'
        elif gender_input == "f":
            gender_data = 'FEMALE'
        else:
            print("Invalid gender input..Enter value [ m or f ]..")
            gender()
        return gender_data

    def date():

        global date_of_member
        try:
            print("Date input..")
            print("Enter month in numerical format [ 1 to 12 ] :")
            month = int(input())
            print("Enter day in numerical format :")
            day = int(input())
            print("Enter year in numerical format :")
            year = int(input())
            try:
                date_of_member = datetime.strptime(f'{year}-{month}-{day}', '%Y-%m-%d')
                date_of_member = date_of_member.strftime('%Y-%m-%d')
                return date_of_member
            except:
                print("Input date is not valid..")
                date()
        except:
            print("Invalid date input..")
            date()

    def joindate():

        now = datetime.now()
        joining_date = now.strftime('%Y-%m-%d')
        return joining_date

    def re_enter():

            cursorObj.execute("SELECT * FROM club_table ORDER BY MEMBERSHIP_ID DESC LIMIT 1")
            noneCheck = cursorObj.fetchall()

            for row in noneCheck:
                if 'None' in row:
                    print("Technical Error in user data insertion..Requested to re-enter data..")
                    print(f"Error in data parsing || {row} ||.")
                    cursorObj.execute("DELETE FROM club_table WHERE MEMBERSHIP_ID in(SELECT MAX(MEMBERSHIP_ID) FROM club_table)")
                    print("Requested to re-enter data..")
                    conn.commit()
                    insert()
                else:
                    return

    def members():

        members_id = []
        memberIdData = cursorObj.execute("SELECT Membership_id FROM club_table ")

        for row in memberIdData:
            memb = str(row)
            members_id.append(int(memb.replace("(", "").replace(",", "").replace(")", "")))
        return members_id

    def gui_check():

        print("Do you wish to see in GUI [ yes --> 1 , no --> 2 ]")
        try:
            guiCheck = int(input())
            if guiCheck == 1:
                gui_db()
            elif guiCheck == 2:
                return
            else:
                print("Invalid input.. Enter value [ 1 or 2 ]..")
                gui_check()
        except:
            print("Invalid input..")
            gui_check()

    def gui_db():

        member_data = tk.Tk()
        member_data.geometry("1024x480")

        e = Label(member_data, width=17, text='MEMBERSHIP ID', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=0)
        e = Label(member_data, width=17, text='NAME', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=1)
        e = Label(member_data, width=17, text='PHONE NUMBER', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=2)
        e = Label(member_data, width=17, text='AGE', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=3)
        e = Label(member_data, width=17, text='GENDER', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=4)
        e = Label(member_data, width=17, text='DATE OF BIRTH', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=5)
        e = Label(member_data, width=17, text='DATE JOINED', borderwidth=2, relief='ridge', anchor='center', bg='yellow')
        e.grid(row=0, column=6)

        total_data = cursorObj.execute('SELECT * FROM club_table ')
        i = 1
        for membdata in total_data:
            for j in range(len(membdata)):
                e = Entry(member_data, width=20, fg='black')
                e.grid(row=i, column=j)
                e.insert(END, membdata[j])
            i = i + 1
        member_data.mainloop()

    def insert():

        print("Insert data ")

        print("Name..")
        nameData = name()

        print("Phone number..")
        phoneNoData = phone_no()

        print("Age..")
        ageData = age()

        print("Enter gender..")
        genderData = gender()

        print("Date of birth..")
        dobMemb = date()

        print("Registering join date..")
        joinDate = joindate()

        cursorObj.execute(f"""INSERT INTO club_table (NAME,PHONE_NUMBER,AGE,GENDER,DATE_OF_BIRTH,DATE_JOINED) 
                            VALUES('{nameData}','{phoneNoData}','{ageData}','{genderData}','{dobMemb}','{joinDate}');""")
        conn.commit()
        re_enter()
        print("Data inserted successfully")

    def update():

        print("Update data")

        print("Enter Membership Id :")

        try:
            membId = int(input())

            if membId in members():

                def update_data():

                    print("What do want to update :")
                    print(" 1 --> Update Name")
                    print(" 2 --> Update Phone_no")
                    print(" 3 --> Update Age")
                    print(" 4 --> Update Gender")
                    print(" 5 --> Update Date_of_birth")
                    print("Enter your choice :")

                    try:

                        user_update_choice = int(input())

                        if user_update_choice == 1:

                            name_update = name()
                            cursorObj.execute(f"UPDATE club_table SET NAME = '{name_update}' WHERE Membership_id = {membId}; ")  # Update records
                            conn.commit()
                            print("Database table updated successfully")

                        elif user_update_choice == 2:

                            phoneno_update = phone_no()
                            cursorObj.execute(f"UPDATE club_table SET PHONE_NUMBER = '{phoneno_update}' WHERE Membership_id = {membId}; ")  # Update records
                            conn.commit()
                            print("Database table updated successfully")

                        elif user_update_choice == 3:

                            age_update = age()
                            cursorObj.execute(f"UPDATE club_table SET AGE = '{age_update}' WHERE Membership_id = {membId}; ")  # Update records
                            conn.commit()
                            print("Database table updated successfully")

                        elif user_update_choice == 4:

                            gender_update = gender()
                            cursorObj.execute(f"UPDATE club_table SET GENDER = '{gender_update}' WHERE Membership_id = {membId}; ")  # Update records
                            conn.commit()
                            print("Database table updated successfully")

                        elif user_update_choice == 5:

                            dob_memb = date()
                            cursorObj.execute(f"UPDATE club_table SET DATE_OF_BIRTH = '{dob_memb}' WHERE Membership_id = {membId}; ")  # Update records
                            conn.commit()
                            print("Database table updated successfully")

                        else:

                            print("Invalid input number..")
                            update_data()

                    except:

                        print("Invalid input")
                        update_data()

                update_data()

            else:
                print("Invalid Membership Id ")
                update()

        except:
            print("Invalid input")
            update()

    def delete():

        print("Deleting row of table")

        print("Enter Membership Id = ")
        try:
            membId = int(input())

            if membId in members():

                cursorObj.execute(f"DELETE FROM club_table WHERE Membership_id = {membId};")
                conn.commit()
                print("Database table deleted successfully")

            else:

                print("Invalid Membership Id")
                delete()

        except:
            print("Invalid Membership Id")
            delete()

    def show_db():

        if members():

            cursorObj.execute(f"SELECT * FROM club_table")
            rows = cursorObj.fetchall()

            print("Badminton Club Member details:")
            print(tuple(columnNames))
            for row in rows:
                print(row)

            gui_check()

        else:
            print("No record in database table..")

    def export_to_excel():

        print("Do you want to export your table into excel sheet [ yes --> 1 , no --> 2 ]")
        checkExport = int(input())

        if checkExport == 1:

                print("Exporting data into excel sheet............")
                cursorObj.execute(f"SELECT * FROM club_table")
                with open(f"badminton_club_membership_details.csv", "w", newline="") as csv_file:
                  csv_writer = csv.writer(csv_file,delimiter=",")
                  csv_writer.writerow([i[0] for i in cursorObj.description])
                  csv_writer.writerows(cursorObj)
                dirpath = os.getcwd() + f"\ badminton_club_membership_details.csv"
                print("Data exported Successfully into {}".format(dirpath))

        elif checkExport == 2:

            return

        else:

            print("Wrong input..")
            export_to_excel()

    def db_manupulation():

        print(" 1 ---> Insert data into table")
        print(" 2 ---> Update the data")
        print(" 3 ---> Delete data from table")
        print(" 4 ---> Show database table")
        print(" 5 ---> Export database table to excel")
        print(" Enter your choice :")

        try:
            result = int(input())
            if result == 1:
                insert()
            elif result == 2:
                update()
            elif result == 3:
                delete()
            elif result == 4:
                show_db()
            elif result == 5:
                export_to_excel()
            else:
                print("Invalid input number..")
                db_manupulation()
        except:
            print("Invalid input")
            db_manupulation()

    def continue_again():

        try:
            print("Do you want to continue : [ yes --> 1 , no --> 2 ]")
            continueResult = int(input())

            if continueResult == 1:
                db_manupulation()
            elif continueResult == 2:
                return
            else:
                print("Wrong input")
                continue_again()
        except:
            print("Invalid input")
            continue_again()

        continue_again()

    db_manupulation()

    continue_again()

sqllite_conn = sql_connection()
sql_table(sqllite_conn)

if sqllite_conn:
    sqllite_conn.close()
    print("\nThe Database connection is closed.")
    print("Process ended.....")

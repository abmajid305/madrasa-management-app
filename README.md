# madrasa-management-app
madrasa management to mange and calculate mark , attendance ,monthly fee and summery of each class to head of madrasa 
📘 Madrasa / School Management System

A fully digital management system designed for Madrasas and Schools to simplify daily administrative work.
This system is built using Google Apps Script and Google Sheets, with a lightweight HTML/CSS/JS frontend, making it easy to deploy, maintain, and extend.

The system automates attendance tracking, exam results, fee management, student ranking, and class-wise summaries — all connected to Google Sheets for real-time data updates.

🌟 Features
🧑‍🎓 Student Management

Student login with class selection

Class-wise dashboards

Arabic–English bilingual interface (optional)

📊 Attendance System

Daily attendance entry (1 = Present, 0 = Absent, blank = Not Recorded)

Automatic Attendance Ranking for every student

Dashboard with:

Today’s attendance status

Monthly attendance

Yearly attendance summary

📝 Marks & Examination

Supports:

Quarterly exams

Half-yearly exams

Final exams

Automatic result calculations

Shows:

Rank in class

Rank in attendance

Term-wise marks

Total marks & grade

When a teacher enters marks in specific sheet columns,
➝ The system automatically calculates results
(Using Google Sheets formulas, so this logic is NOT in the backend code.)

💰 Fee Management

Monthly fee tracking

Payment status

Overdue and pending fees

Class-level financial summaries

🧾 Class Summary for Head of Madrasa / School

Student strength summary

Attendance summary

Fee collection summary

Exam performance summary

Automatically updates as teachers enter data in Google Sheets

⚙️ Automation

A major portion of the automation is handled through:

Google Sheets formulas

ARRAYFORMULA

Custom ranking formulas

Conditional formatting

These formulas run independently and are not included in the Apps Script backend.

🧩 Technology Stack
Frontend

HTML

CSS

JavaScript

Backend

Google Apps Script

Web App Deployment (doGet, doPost)

API-like endpoints for login and data retrieval

Database

Google Sheets

Sheet-per-class student credentials

Sheet for attendance

Sheets for marks, fees, summaries

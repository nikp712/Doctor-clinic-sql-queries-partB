# Doctor’s Clinic SQL Queries – Part B

## Table of Contents
1. [Overview](#overview)
2. [Project Features](#project-features)
3. [Database Summary](#database-summary)
4. [Preprocessing Steps](#preprocessing-steps)
5. [SQL Queries and Analysis](#sql-queries-and-analysis)
6. [Key Insights](#key-insights)
7. [How to Run the Project](#how-to-run-the-project)
8. [Results](#results)
9. [Future Improvements](#future-improvements)
10. [Acknowledgments](#acknowledgments)

---

## Overview
This project demonstrates advanced SQL querying for a structured medical clinic database (Doctor’s SuperClinic case study, RMIT 2022). Each query answers a real-world healthcare/business question, such as appointments, diagnoses, and staff activity. The project provides modular SQL scripts and visualisation examples, with the database schema and data kept private for compliance.

---

## Project Features
- **12 Realistic SQL Queries:** Answers operational and analytic questions for a clinic business scenario.
- **View Creation:** Example of SQL views for live reporting and staff workload.
- **Aggregations and Joins:** Combines tables for clinical, HR, and patient insights.
- **Data Visualisation:** Code and recommendations for plotting query results.
- **Portfolio-Ready:** Clean structure and documentation for recruiters or instructors.

---

## Database Summary
- **Dataset:** Doctor’s SuperClinic (ISYS2421, RMIT, 2022).
- **Key Tables:** `appointment`, `doctor`, `patient`, `room`, `request`, `disease`, etc.
- **Privacy Notice:** Database and data are not included due to privacy. Queries are written based on the provided schema in the assignment PDF.

---

## Preprocessing Steps
1. Connected to the sample database (MySQL) provided by the course.
2. Reviewed the case study, schema, and ER diagram.
3. Prepared queries with proper joins, filtering, and groupings.

---

## SQL Queries and Analysis

**Example queries**

1. **View of today’s appointments**
   ```sql
   CREATE VIEW todays_appointments AS
   SELECT CONCAT(a.dateOfAppointment, " ", TIME_FORMAT(a.timeOfAppointment, '%H:%i')) AS DateAndTime,
   CONCAT(d.given, " ", d.surname) AS Doctor_name,
   CONCAT(p.given, " ", p.surname) AS Patient_name, 
   p.Phone, Done AS STATUS
   FROM appointment a, doctor d, patient p
   WHERE d.doctorID = a.doctorID
   AND p.patientID = a.patientID
   AND YEAR(a.dateOfAppointment) = YEAR(SYSDATE())
   AND MONTH(a.dateOfAppointment) = MONTH(SYSDATE())
   AND DAY(a.dateOfAppointment) = DAY(SYSDATE())
   ORDER BY Doctor_name, dateofappointment, timeofappointment;
   ```

2. **Appointments for March 2022 (doctor/patient, disease, status)**
   ```sql
   SELECT CONCAT(d.given," ",d.surname)AS Doctor_name, CONCAT(p.given,"
   ",p.surname) AS Patient_name,
   CONCAT(DATE_FORMAT(a.dateOfAppointment,'%Y-%m-%d')," ",TIME_FORMAT(a.timeOfAppointment,'%H:%i')) AS DateAndTime,
   a. RoomNo, a.Done AS STATUS, disease.diseasename AS Disease_name
   FROM doctor d, patient p, appointment a
   LEFT JOIN request ON request.appointmentID=a.appointmentID
   LEFT JOIN disease ON disease.DiseaseID = request.DiseaseID
   WHERE MONTH(a.dateOfAppointment)=3
   AND YEAR(a.dateOfAppointment)=2022
   AND d.doctorID = a.doctorID
   AND p.patientID = a.patientID
   ORDER BY a.dateOFAppointment, a.timeOfAppointment;
   ```

3. **Appointments not in Richmond in 2021**
   ```sql
   SELECT CONCAT(p.given," ",p.surname) AS Patient_name,
   CONCAT(a.dateOfAppointment," ",TIME_FORMAT(a.timeOfAppointment,'%H:%i'))
   AS DateAndTime
   FROM appointment a, patient p
   WHERE YEAR(dateOfAppointment) = 2021
   AND p.suburb = 'RICHMOND'
   AND p.patientID = a.patientID
   AND appointmentid NOT IN (SELECT appointmentid FROM request WHERE requesttype="D");
   ```

4. **Oldest and youngest male patient (with calculated age)**
   ```sql
   SELECT surname,given,TRUNCATE(DATEDIFF(SYSDATE(),Dob)/365.25,0) AS Age,Sex
   FROM patient
   WHERE (dob=(SELECT MAX(dob) FROM patient WHERE sex = "M")
   OR dob=(SELECT MIN(dob) FROM patient WHERE sex = "M"))
   AND sex = "M"
   ```
   
5. **Room usage for completed appointments**
   ```sql
   SELECT a.RoomNo,roomname, COUNT(*)
   FROM appointment a, room r
   WHERE a.done='Y'
   AND a.roomno = r.roomno
   GROUP BY a.RoomNo;
   ```

6. **Doctors with appointments for >5 diseases**
   ```sql
   SELECT CONCAT(p.given, " ", p.surname) AS patient,
   CONCAT("Dr ",o.given, " ", o.surname) AS doctor,
   dateofappointment, roomno, COUNT(*)
   FROM appointment a, request r, disease d, doctor o, patient p
   WHERE a.appointmentid = r.appointmentid AND r.diseaseid = d.diseaseid
   AND o.doctorid = a.doctorid AND a.patientid = p.patientid
   GROUP BY a.appointmentid
   HAVING COUNT(*) > 5
   ORDER BY o.surname, a.dateofappointment;
   ```

7. **Diseases diagnosed less than 10 times**
   ```sql
   SELECT e.diseaseID, e.diseasename, COUNT(d.diseaseID)
   FROM disease e LEFT JOIN request d ON e.DiseaseID=d.DiseaseID
   GROUP BY e.DiseaseID, e.diseasename
   HAVING COUNT(*) < 10;
   or
   SELECT diseasename, 0 AS timesDiagnosed
   FROM disease
   WHERE diseaseID NOT IN (SELECT DISTINCT diseaseid FROM request WHERE diseaseid IS NOT NULL)
   UNION
   SELECT diseasename,COUNT(*)
   FROM disease, request
   WHERE disease.diseaseID=request.DiseaseID
   GROUP BY diseasename
   HAVING COUNT(*) < 10;
   ```

8. **Patient full name and request details, if more than 130 appointments**
   ```sql
   SELECT p.patientID, CONCAT(p.surname, ",", p.given) AS full_name,
   COUNT(DISTINCT a.appointmentid) AS TimeVisited, COUNT(DISTINCT r.testid) AS tests_ordered,
   COUNT(DISTINCT diseaseid) AS diagnosed_disease, COUNT(DISTINCT drugid) AS drugs_prescribed
   FROM patient p, appointment a LEFT JOIN request r ON
   r.appointmentid=a.appointmentid
   WHERE p.patientID = a.patientID
   GROUP BY p.patientID, p.surname, p.given
   HAVING COUNT(DISTINCT a.appointmentid) > 130
   ORDER BY p.surname;
   or DONE = 'Y' depends on the interpretation of "visits"
   ```

9. **Doctor and patient info for all 2022 appointments**
   ```sql
   SELECT (CONCAT('D', d.doctorID)) AS ID, CONCAT(d.surname, ", Dr ", d.given) AS full_name, d.address,d.suburb,d.postcode
   FROM doctor d
   WHERE d.doctorID IN (SELECT DISTINCT doctorID FROM appointment WHERE
   YEAR(dateOfAppointment) ='2022')
   UNION
   SELECT (CONCAT('P', p.patientID)) AS ID, CONCAT(p.surname, ",", p.given)
   AS full_name, p.address,p.suburb,p.postcode
   FROM patient p
   WHERE patientID IN (SELECT DISTINCT patientID FROM appointment WHERE
   YEAR(dateOfAppointment) ='2022')
   ORDER BY suburb, full_name;
   ```

10. **Female doctors with supervisor info and years worked**
    ```sql
    SELECT CONCAT(d.surname," ",d.given) AS Doctor_name, CONCAT(s.surname," ",s.given) AS Supervisor_name,
    TRUNCATE((DATEDIFF(IFNULL(d.resigned,SYSDATE()),d.joined)/365.25),0) AS years_worked
    FROM doctor d, doctor s
    WHERE d.supervisorID = s.DoctorID
    AND TRUNCATE((DATEDIFF(IFNULL(d.resigned,SYSDATE()),d.joined)/365.25),0)
    BETWEEN 1 AND 12
    AND d.sex='F';
    ```

11. **Create view for number of appointments attended by each doctor**
    ```sql
    CREATE VIEW AttendedCount AS
    SELECT doctorID, COUNT(*) as timeVisitedPatient
    FROM appointment
    GROUP BY doctorID;
    SELECT *
    FROM AttendedCount, Doctor
    WHERE AttendedCount.doctorID = Doctor.doctorID
    AND ( timeVisitedPatient = (SELECT MAX(timeVisitedPatient) FROM AttendedCount)
    OR timeVisitedPatient = (SELECT MIN(timeVisitedPatient) FROM dAttendedCount) );
    ```

12. **Data Visualization: Patient Age Distribution**

![Screenshot (52)](https://github.com/user-attachments/assets/bdaec3e5-4c6d-4842-b366-02fceaa8993f)


13. **What does your histogram show, and why is it useful for business decision-making?**
    - The histogram visualizes the distribution of patient ages across the database. Each bar represents a range of ages (for example, 32–36 years, 36–41 years, etc.), and the height of each bar indicates the number of patients in each age group.
    
    **Business Value & Insights:**
    - Patient Demographics: The business can easily see which age groups are most and least represented in the patient population.
    - Targeted Services: Knowing the most common age brackets helps the clinic design tailored healthcare packages or communication for different age groups.
    - Resource Planning: Age distribution can inform staffing and service availability. E.g. pediatric vs. geriatric care.
    - Marketing & Engagement: Marketing efforts can be better aligned with the demographics, focusing on age groups with the highest patient counts.
    - Overall, the histogram transforms raw data into actionable insight, making it much easier for business leaders to understand their patient base and make strategic decisions.
      
---

## Key Insights

- SQL views simplify reporting for clinic operations.
- Advanced joins and subqueries support complex business logic.
- Queries are adaptable for any medical or business database with a similar structure.

---

## How to Run the Project

1. **Clone or download** this repository to your computer.
2. **Set up** a MySQL or MariaDB instance (local or cloud).
3. **Import the sample database** (from your instructor or assignment kit).
4. **Check your database schema** (it may differ—adjust queries if needed).
5. **Open** `SQL_Queries.sql` in SQL Workbench or MySQL.
6. **Run each query independently** and review the results.

> **Note:**  
> The original database is not included due to privacy and copyright.  
> Adjust table/column names as needed for your database schema.

---

## Results

- All queries are ready to run and have been tested against the provided schema.
- Visualisations and reports help with business analytics and clinic operations.
- Example result sets can be saved/exported for further analysis.

---

## Future Improvements

- Add parameterised reporting (date ranges, doctors, etc.).
- Develop dashboards for operational managers.
- Explore automation with stored procedures.

---

## Acknowledgments

- Case study and database schema: RMIT University.
- SQL tools: MySQL, SQL Workbench.

---

## Author

Nhi Phan | RMIT University  
Final Student Year, Bachelor of Business Information Systems (Expected November 2025)

---

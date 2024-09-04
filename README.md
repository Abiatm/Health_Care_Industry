
# Retail_Inventory_Report
## Table of contents 
- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Data Source](#data-source)
- [Tools Used](#tools-used)
- [Data filtering_OR_Data Analysis](#Data_filtering_OR-Data_analysis)
- [Query](#Query)
- [Image](#Image)
- [Results](#results)
- [Recommendations](#recommendations)
- [Conclusion](#Conclusion)
### Project Overview
#### Healthcare Patient Database Analysis
This report provides a comprehensive analysis of the Healthcare Patient database. The database captures information about patients, doctors, states, and departments, enabling easy acess to explore relationships and patterns within the healthcare system.
### Problem Statement
- Fetch patients with the same age.
- Find the second oldest patient and their doctor and department.
- Get the maximum age per department and the patient name.
- Doctor-wise count of patients sorted by count in descending order.
- Fetch only the first name from the PatientName and append the age.
- Fetch patients with odd ages.
- Create a view to fetch patient details with an age greater than 50.
- Create a procedure to update the patient's age by 10% where the department is 'Cardiology' and the doctor is not 'Dr. Smith'.
- Create a stored procedure to fetch patient details along with their doctor, department, and state, including error handling.
### Data Source
Healthcare industry
### Tools Used
- SSMS
- MYSQL
### Data filtering_OR_Data Analysis
Before the analysis was done creation of database ,tables and inserting of their data was doen into sql database system so as to have a single documentation of the health care data,after which the required questions of needs was obtained and with use of views and stored procedures data was stored for future needs,the sql database,database creation,tables,query is shown in 'Query' below
### Query
```
CREATE DATABASE HealthcarePatientDb;

USE HealthcarePatientDb;

CREATE TABLE Patient (
    PatientID NVARCHAR(10) PRIMARY KEY,
    PatientName NVARCHAR(50),
    Age INT,
    Gender NVARCHAR(10),
    DoctorID NVARCHAR(10),
    StateID NVARCHAR(10)
);
INSERT INTO Patient (PatientID, PatientName, Age, Gender, DoctorID, StateID)
VALUES
    ('PT01', 'John Doe', 45, 'M', '1', '101'),
    ('PT02', 'Jane Smith', 30, 'F', '2', '102'),
    ('PT03', 'Mary Johnson', 60, 'F', '3', '103'),
    ('PT04', 'Michael Brown', 50, 'M', '4', '104'),
    ('PT05', 'Patricia Davis', 40, 'F', '1', '105'),
    ('PT06', 'Robert Miller', 55, 'M', '2', '106'),
    ('PT07', 'Linda Wilson', 35, 'F', '3', '107'),
    ('PT08', 'William Moore', 65, 'M', '4', '108'),
    ('PT09', 'Barbara Taylor', 28, 'F', '1', '109'),
    ('PT10', 'James Anderson', 70, 'M', '2', '110');


CREATE TABLE Doctor (
    DoctorID NVARCHAR(10),
    DoctorName NVARCHAR(50),
    Specialization NVARCHAR(50)
);

INSERT INTO Doctor (DoctorID, DoctorName, Specialization)
VALUES
    ('1', 'Dr. Smith', 'Cardiology'),
    ('2', 'Dr. Adams', 'Neurology'),
    ('3', 'Dr. White', 'Orthopedics'),
    ('4', 'Dr. Johnson', 'Dermatology');

CREATE TABLE StateMaster (
    StateID NVARCHAR(10),
    StateName NVARCHAR(50)
);

INSERT INTO StateMaster (StateID, StateName)
VALUES
    ('101', 'Lagos'),
    ('102', 'Abuja'),
    ('103', 'Kano'),
    ('104', 'Delta'),
    ('105', 'Ido'),
    ('106', 'Ibadan'),
    ('107', 'Enugu'),
    ('108', 'Kaduna'),
    ('109', 'Ogun'),
    ('110', 'Anambra');

CREATE TABLE Department (
    DepartmentID NVARCHAR(10),
    DepartmentName NVARCHAR(50)
);

INSERT INTO Department (DepartmentID, DepartmentName)
VALUES
    ('1', 'Cardiology'),
    ('2', 'Neurology'),
    ('3', 'Orthopedics'),
    ('4', 'Dermatology');



--QUE1-Fetch patients with the same age.

SELECT * FROM [HealthcarePatientDb].[dbo].[Patient]
WHERE Age IN
(
SELECT Age FROM [HealthcarePatientDb].[dbo].[Patient]
GROUP BY Age
HAVING COUNT(Age)>1)

--QUE 2-Find the second oldest patient and their doctor and department.
SELECT P.PatientName,P.Age,D.DoctorName,DE.DepartmentName FROM [HealthcarePatientDb].[dbo].[Patient] P
LEFT JOIN [HealthcarePatientDb].[dbo].[Doctor] D
ON P.DoctorID=D.DoctorID
LEFT JOIN [HealthcarePatientDb].[dbo].[Department] DE
ON DE.DepartmentID=D.DoctorID
 ORDER BY P.Age DESC
 OFFSET 1 ROW     --jumps the two hhighest rows
 FETCH NEXT 1 ROW ONLY;

 --QUE 3-Get the maximum age per department and the patient name.
WITH RANKING AS(
SELECT P.PatientName,P.Age,D.DepartmentName,DENSE_RANK() OVER (PARTITION BY D.DepartmentName ORDER BY Age DESC) RANKINGS FROM [HealthcarePatientDb].[dbo].[Department] D
LEFT JOIN  [HealthcarePatientDb].[dbo].[Patient] P
ON D.DepartmentID=P.DoctorID)
SELECT PatientName,Age,DepartmentName FROM RANKING
WHERE RANKINGS=1

--QUE 4-Doctor-wise count of patients sorted by count in descending order.
SELECT D.DoctorName,COUNT(P.PatientName) Count_by_Doc  FROM [HealthcarePatientDb].[dbo].[Patient] P
LEFT JOIN [HealthcarePatientDb].[dbo].[Doctor] D
ON P.DoctorID=D.DoctorID
GROUP BY D.DoctorName 
ORDER BY Count_by_Doc

--QUE 5 Fetch only the first name from the PatientName and append the age.
SELECT PARSENAME(REPLACE(PatientName, ' ', '.'), 2) + '_' + CAST(Age AS VARCHAR) AS FIRSTNAME_Age
FROM [HealthcarePatientDb].[dbo].[Patient] ;

--QUE 6 Fetch patients with odd ages.
SELECT * FROM  [HealthcarePatientDb].[dbo].[Patient]
  WHERE Age%2 !=0
  
  --QUE 7 Create a view to fetch patient details with an age greater than 50.
CREATE VIEW vw_AgeGreaterThan50 AS
  SELECT PatientName,Age FROM  [HealthcarePatientDb].[dbo].[Patient]
  WHERE Age >50 

  
--QUE 8 Create a procedure to update the patient's age by 10% where the department is 'Cardiology' and the doctor is not 'Dr. Smith'.

SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE sp_AgePerIncreases10
AS
BEGIN
UPDATE  P
 SET P.Age=1.10 * P.Age FROM [HealthcarePatientDb].[dbo].[Patient]   P
LEFT JOIN [HealthcarePatientDb].[dbo].[Department]   D
ON P.DoctorID=D.DepartmentID
LEFT JOIN [HealthcarePatientDb].[dbo].[Doctor]   DO
ON DO.DoctorID=D.DepartmentID
 
 WHERE D.DepartmentName='Cardiology' AND DO.DoctorName NOT IN ('Dr. Smith')



END
GO


--QUE 9-Create a stored procedure to fetch patient details along with their doctor, department, and state, including error handling.

CREATE PROCEDURE sp_PatDetails
AS
BEGIN
BEGIN TRY
SELECT P.PatientName,P.Age,D.DoctorName,DE.DepartmentName,S.StateName FROM [HealthcarePatientDb].[dbo].[Patient] P
LEFT JOIN [HealthcarePatientDb].[dbo].[Doctor] D
ON P.DoctorID=D.DoctorID
LEFT JOIN [HealthcarePatientDb].[dbo].[Department] DE
ON DE.DepartmentID=D.DoctorID
LEFT JOIN [HealthcarePatientDb].[dbo].[StateMaster] S
ON P.StateID=S.StateID
   END TRY
   BEGIN CATCH
   DECLARE @ERRORMESSAGE NVARCHAR(4000)
    
   SET @ERRORMESSAGE =ERROR_MESSAGE()
   
 --  RAISEERROR (@ERRORMESSAGE,16,1);
   END CATCH
   END

```

### Image
![HEALTH_PORT](https://github.com/user-attachments/assets/ef0ee033-47fb-4793-8822-a71ee5ea6329)

### Results
- Age Distribution: The patient data reveals a diverse age range, with individuals spanning from their 20s to their 70s. This shows that different age range from patients seeking healthcare services.

- Gender Balance: The database exhibits a relatively balanced gender distribution, with both male and female patients represented. This suggests that the healthcare system is is for both gender.

- Doctor Specialization: The analysis highlights the varying specializations of doctors, including Cardiology, Neurology, Orthopedics, and Dermatology. This demonstrates the availability of specialized care for different medical conditions.

- State Representation: Patients are distributed across multiple states, indicating a wide geographical reach of the healthcare services. This suggests that the system is serving patients from various states or region.

- Departmental Distribution: The data reveals a balanced distribution of patients across different departments, suggesting a well-structured healthcare system with various specialties.

### Recommendations
- From the age bracket it shows that from 45 and above goes for health care services,therefore more health facilities should be provided for such age bracketto met their needs
- Availability in more states should be encouraged for easy accesibility 
- More equipments shoud be provided as it has many specializations and also their is need for paient care
### Conclusion
The analysis of the Healthcare Patient database provides valuable insights into the patient population, doctor specializations, geographical distribution, and departmental structure. These findings helps for strategic planning, resource allocation, and service improvements to enhance the overall quality of healthcare delivery.


# Company Database Schema

A relational database schema for managing a company's organizational structure, tracking locations, departments, job salary grades, and employee hierarchies.

## Database Overview

This database consists of 4 primary tables designed to model an enterprise structure:
* **Locations**: Stores physical office addresses globally.
* **Departments**: Tracks organizational units, their physical locations, and their managers.
* **JobGrades**: Defines pay scale ranges (minimum and maximum salaries) per grade level.
* **Employees**: Contains complete staff records, structural department alignments, salary info, and management hierarchies (self-referencing).

---

## Entity-Relationship Diagram (ERD) Outline

* `Locations` (1) ─── (0..*) `Departments`
* `Departments` (1) ─── (0..*) `Employees`
* `JobGrades` (1) ─── (0..*) `Employees`
* `Employees` (1) ─── (0..*) `Employees` *(Self-referencing Manager relationship)*

---

## SQL Deployment Script

You can copy and run the following standard SQL script to initialize and build the database structure.

### 1. Database Creation & Initialization
```sql
-- Create Database
CREATE DATABASE CompanyDB;
GO

-- Use Database
USE CompanyDB;
GO
```

### 2. Table Creation Scripts

```sql
-- 1. Location Table
CREATE TABLE Locations (
    LocationID INT PRIMARY KEY,
    LocationName VARCHAR(255) NOT NULL,
    StreetAddress VARCHAR(255),
    City VARCHAR(100) NOT NULL,
    StateProvince VARCHAR(100),
    CountryID VARCHAR(2), -- ISO 2-letter country code
    PostalCode VARCHAR(20)
);

-- 2. JobGrades Table
CREATE TABLE JobGrades (
    GradeID INT PRIMARY KEY,
    GradeLevel VARCHAR(50) NOT NULL,
    MinSalary DECIMAL(10,2) NOT NULL,
    MaxSalary DECIMAL(10,2) NOT NULL
);

-- 3. Departments Table
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100) NOT NULL,
    LocationID INT,
    ManagerID INT, 
    FOREIGN KEY (LocationID) REFERENCES Locations(LocationID)
    -- Note: ManagerID Foreign Key is typically added via ALTER TABLE 
    -- after the Employees table is created to avoid circular references.
);

-- 4. Employee Table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    PhoneNumber VARCHAR(20),
    JobID INT, -- Placeholder for an external Jobs catalog
    GradeID INT,
    DepartmentID INT,
    ManagerID INT,  -- Self-referencing Foreign Key
    Salary DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (GradeID) REFERENCES JobGrades(GradeID),
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID),
    FOREIGN KEY (ManagerID) REFERENCES Employees(EmployeeID)
);

-- 5. Add Circular Foreign Key Constraint safely
ALTER TABLE Departments 
ADD CONSTRAINT FK_Department_Manager 
FOREIGN KEY (ManagerID) REFERENCES Employees(EmployeeID);
```

---

## Features & Constraints

* **Data Integrity**: Columns like `Email` are strictly declared `UNIQUE` to prevent duplicated staff profile records.
* **Organizational Hierarchy**: The `Employees` table implements a self-referencing relationship mapping `ManagerID` directly back to `EmployeeID`.
* **Salary Rules Engine**: The `JobGrades` structural breakdown lets HR monitor budget constraints by pairing individual employee salaries with predefined `MinSalary` and `MaxSalary` limits.

## Contributions
Feel free to open an issue or submit a pull request if you want to extend this schema to include `Jobs`, `Projects`, or `Dependent` tables!

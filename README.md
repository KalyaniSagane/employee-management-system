# Employee Management System — AWS Full-Stack Deployment

A full-stack Employee Management System with a Java Spring Boot REST API, MySQL database, and static frontend — architected for deployment on AWS using EC2, RDS, S3, an Application Load Balancer, and Auto Scaling.

## Overview

This project demonstrates a complete, production-style AWS deployment: a highly available backend behind a load balancer, a managed relational database in a private subnet, and a decoupled static frontend served from S3 — with full CRUD functionality end to end.

## Tech Stack

- **Backend:** Java 17, Spring Boot 3, Spring Data JPA
- **Database:** Amazon RDS (MySQL)
- **Frontend:** HTML/CSS/JavaScript (static, hosted on S3)
- **Infrastructure:** AWS VPC, EC2, Application Load Balancer, Auto Scaling Group, S3
- **Containerization:** Docker

## AWS Architecture

- Custom **VPC** with public and private subnets across 2 Availability Zones
- **Application Load Balancer** distributing traffic across EC2 instances
- **Auto Scaling Group** for high availability and fault tolerance
- **Amazon RDS (MySQL)** in a private subnet, accessible only from application servers
- **S3** static website hosting for the frontend
- Security groups scoped so only the ALB is internet-facing — see [`AWS_INFRASTRUCTURE.md`](./AWS_INFRASTRUCTURE.md) for the full architecture, security group rules, and deployment steps

## Features / CRUD Operations

- `GET /api/employees` — list all employees
- `GET /api/employees/{id}` — get a single employee
- `POST /api/employees` — add a new employee
- `PUT /api/employees/{id}` — update an employee
- `DELETE /api/employees/{id}` — remove an employee
- `GET /api/employees/health` — health check endpoint

## Results

- Demonstrated end-to-end AWS architecture handling real-time database operations with zero-downtime scaling
- Decoupled frontend/backend architecture allows independent scaling and deployment
- Security-group isolation ensures the database is never directly exposed to the internet

## Project Structure

```
employee-management-system/
├── src/main/java/com/kalyani/ems/
│   ├── EmployeeManagementSystemApplication.java
│   ├── controller/EmployeeController.java
│   ├── model/Employee.java
│   ├── repository/EmployeeRepository.java
│   └── service/EmployeeService.java
├── src/main/resources/application.properties
├── frontend/index.html
├── Dockerfile
├── AWS_INFRASTRUCTURE.md
├── pom.xml
└── README.md
```

## How to Run Locally

```bash
# 1. Start a local MySQL instance (or point to RDS)
#    create database: employee_db

# 2. Set environment variables (or edit application.properties)
export RDS_ENDPOINT=localhost
export RDS_USERNAME=root
export RDS_PASSWORD=yourpassword

# 3. Build and run
mvn clean package
java -jar target/employee-management-system.jar

# 4. Open frontend/index.html in a browser
#    (make sure API_BASE in index.html points to http://localhost:8080/api/employees)
```

## Running with Docker

```bash
docker build -t employee-management-system .
docker run -d -p 8080:8080 \
  -e RDS_ENDPOINT=<your-rds-endpoint> \
  -e RDS_USERNAME=<username> \
  -e RDS_PASSWORD=<password> \
  employee-management-system
```

## Author

**Kalyani Ghogare**
[LinkedIn](https://linkedin.com/in/kalyani-377a12211) · saganekalyani@gmail.com

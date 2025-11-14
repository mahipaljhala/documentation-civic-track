# **CivicTrack - Tech Stack Documentation (Final with AWS RDS)**


## **1. System Architecture Overview**

| **Layer** | **Technology / Tool** | **Purpose** |
|------------|------------------------|--------------|
| **Frontend** | React.js (Vite) + Tailwind CSS + Axios | Provides an interactive user interface for all roles (Citizen, Authority, Admin). |
| **Backend** | Node.js + Express.js + Sequelize ORM | Handles REST APIs, authentication, and communicates with PostgreSQL (via RDS). |
| **Database** | **AWS RDS (PostgreSQL)** | Managed relational database for users, issues, logs, and categories. |
| **Storage** | AWS S3 | Stores uploaded issue images securely. |
| **Deployment** | Docker + Docker Compose (on AWS EC2) | Orchestrates containerized backend and frontend. |
| **Security** | JWT (HTTP-only Cookies), bcrypt, Helmet, SSL/TLS | Ensures authentication, encryption, and secure communication. |
| **SSL/TLS** | AWS Certificate Manager | Enables HTTPS for secure frontend and backend access. |

---

## **2. Frontend Stack**

| **Tool / Library** | **Purpose** |
|----------------------|-------------|
| **React.js (Vite)** | Lightweight, modular UI for dynamic page rendering. |
| **Tailwind CSS** | Provides utility-first responsive styling. |
| **Axios** | Handles API calls using secure cookies for auth. |
| **React Router DOM** | Manages routing between role-based dashboards. |
| **PWA Support** | Allows users to install the app on mobile as a web application. |

**Reason for Choice:**  
React and Vite provide fast performance, modular structure, and easy scalability. Tailwind ensures consistent, responsive design across devices.

---

## **3. Backend Stack**

| **Tool / Library** | **Purpose** |
|----------------------|-------------|
| **Node.js + Express.js** | Provides the REST API layer for authentication, issue management, and admin operations. |
| **Sequelize ORM** | Simplifies PostgreSQL interactions using model-based queries. |
| **JWT (HTTP-only Cookie)** | Manages secure authentication sessions. |
| **Multer + AWS SDK** | Handles image uploads directly to S3. |
| **bcrypt** | Secures user passwords through hashing. |
| **Helmet + CORS + Express Validator** | Adds security, sanitization, and validation layers. |

**Reason for Choice:**  
Express and Sequelize provide flexibility and maintainable API structure. Using JWT with HTTP-only cookies prevents XSS token leaks.

---

## **4. Database Layer — AWS RDS (PostgreSQL)**

| **Component** | **Technology** | **Purpose** |
|----------------|----------------|--------------|
| **Database Engine** | AWS RDS (PostgreSQL) | Managed relational database service for high reliability. |
| **Indexes** | BTREE, UNIQUE, COMPOSITE| Optimizes relational queries. |
| **Backup** | AWS Automated Backups | RDS-managed daily snapshot and recovery points. |

**Reason for Choice:**  
Using **AWS RDS** offloads database management, backups, and scaling. It provides reliability, encryption, and monitoring without requiring local containers.

---

## **5. Cloud & Infrastructure (AWS)**

| **Service** | **Purpose** |
|--------------|-------------|
| **AWS EC2** | Hosts the Dockerized backend and frontend containers. |
| **AWS RDS (PostgreSQL)** | Stores all structured application data. |
| **AWS S3** | Manages image uploads. |
| **AWS Certificate Manager (ACM)** | Issues SSL certificates for HTTPS encryption. |
| **AWS Route53** | DNS. |

**RDS Networking Setup:**
- EC2 and RDS are deployed within the **same VPC** for secure, private communication.  
- RDS Security Group allows inbound connections **only** from the EC2 instance’s private IP.  
- Database credentials are stored securely in the `.env` file and loaded at runtime.  

---

## **6. Deployment & DevOps Setup**

| **Tool / Process** | **Purpose** |
|----------------------|-------------|
| **Docker** | Containerizes both backend and frontend for consistent environments. |
| **Docker Compose** | Orchestrates all services (frontend, backend) together. |
| **Environment Variables (.env)** | Holds credentials for JWT, AWS, and database. |
| **Restart Policy** | Docker automatically restarts containers (`restart: always`). |


---

## **8. Security & Best Practices**

| **Feature** | **Implementation** |
|--------------|--------------------|
| **Authentication** | JWT stored in HTTP-only cookies. |
| **Password Protection** | bcrypt for salted password hashing. |
| **Validation** | Express Validator for input sanitization. |
| **Encryption** | HTTPS with AWS ACM certificates. |
| **Access Control** | Role-based access (Citizen, Authority, Admin). |

**Security Measures in AWS RDS:**
- Encryption at rest using AWS KMS.  
- Connection limited to private subnet within VPC.  
- Parameter groups configured for optimized performance.  

---

## **9. System Deployment Workflow**

### **Step-by-Step Deployment Flow**
1. Code pushed to GitHub repository.  
2. EC2 instance configured with Docker and Docker Compose.  
3. `.env` file includes RDS credentials (`DB_HOST`, `DB_USER`, `DB_PASS`, `DB_NAME`, `DB_PORT`).  
4. `docker-compose up -d` starts services:  
   - `backend` (Express API)  
   - `frontend` (React.js PWA)  
5. Backend connects securely to RDS PostgreSQL instance.  
6. Image uploads handled by S3 with public read access.  
7. AWS ACM ensures HTTPS for all communication.  

**Ports Configuration:**.   

| Service | Port | Description |
|----------|------|--------------|
| Frontend | 3000 | React web application |
| Backend | 5000 | Node.js REST API |
| RDS (PostgreSQL) | 5432 | Managed database connection |

---
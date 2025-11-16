# **CivicTrack Database Schema Design Document**

## **2. Entity Relationship Diagram (ERD)**

![ERD](../images/the-erd.png)

The ERD represents relationships between **users**, **roles**, **issues**, **authorities**, **flags**, and **logs**.  
It forms the backbone of the data model for CivicTrack.

---

## **3. Core Tables and Relationships**

#### **3.1 users**

Stores all registered platform users.

| Column | Type | Constraints | Description |
|--------|------|-------------|--------------|
| `id` | BIGINT | PK | Unique user ID |
| `name` | VARCHAR(100) | NOT NULL | User’s full name |
| `email` | VARCHAR(150) | UNIQUE, NOT NULL | User’s email for login |
| `password_hash` | VARCHAR(255) | NOT NULL | Encrypted password (bcrypt) |
| `created_at` | TIMESTAMPTZ | DEFAULT now() | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() | Last modification |
| `deleted_at` | TIMESTAMPTZ | NULLABLE | Soft delete marker |

---

### **3.2 roles**

Defines all available roles within the system.

| Column | Type | Constraints | Description |
|--------|------|-------------|--------------|
| `id` | BIGINT | PK | Unique role ID |
| `name` | VARCHAR(50) | UNIQUE, NOT NULL | Role name (citizen, authority, admin) |
| `description` | TEXT | NULL | Description of the role |
| `is_active` | BOOLEAN | DEFAULT TRUE | Role availability |
| `created_at` | TIMESTAMPTZ | DEFAULT now() | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() | Last update timestamp |

---

### **3.3 user_role**

Mapping table linking users with their assigned roles.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `user_id` | BIGINT | FK → `users.id`, NOT NULL |
| `role_id` | BIGINT | FK → `roles.id`, NOT NULL |
| `created_at` | TIMESTAMPTZ | DEFAULT now() |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() |

---

### **3.4 authorities**

Stores each civic department or administrative authority.

| Column | Type | Constraints | Description |
|--------|------|-------------|--------------|
| `id` | BIGINT | PK | Authority ID |
| `name` | VARCHAR(150) | NOT NULL | Authority or department name |
| `city` | VARCHAR(100) | NOT NULL | City or jurisdiction |
| `region` | VARCHAR(100) | NOT NULL | Area of operation |
| `department_id` | BIGINT | FK → `departments.id` | Linked department |
| `address` | TEXT | NULL | Physical address |
<!-- | `coverage_area` | GEOGRAPHY(Polygon) | NULL | Geo-boundary | -->
| `created_at` | TIMESTAMPTZ | DEFAULT now() | Created date |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() | Updated date |

---

### **3.5 authority_user**

Links specific users to their respective authorities.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `authority_id` | BIGINT | FK → `authorities.id`, NOT NULL |
| `user_id` | BIGINT | FK → `users.id`, UNIQUE, NOT NULL |
| `created_at` | TIMESTAMPTZ | DEFAULT now() |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() |

---

### **3.6 issues**

Stores predefined issue categories (e.g., Roads, Water, Lighting).

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `name` | VARCHAR(100) | UNIQUE, NOT NULL |
| `slug` | VARCHAR(100) | UNIQUE, NOT NULL |
| `description` | TEXT | NULL |

---

### **3.7 user_issue**

Stores user-submitted civic issues.

| Column | Type | Constraints | Description |
|--------|------|-------------|--------------|
| `id` | BIGINT | **PK** | Unique issue ID. |
| `title` | VARCHAR(255) | **NOT NULL** | Title of the reported issue. |
| `description` | TEXT | **NOT NULL** | Detailed description of the issue. |
| `issue_id` | BIGINT | **FK → issues.id**, NOT NULL | References the type/category of the issue. |
| `reporter_id` | BIGINT | **FK → users.id**, NOT NULL | ID of the user who reported the issue. |
| `authority_id` | BIGINT | **FK → authorities.id** | ID of the authority assigned to handle this issue. |
| `latitude` | DOUBLE PRECISION | NULL | Latitude coordinate of the issue’s location. |
| `longitude` | DOUBLE PRECISION | NULL | Longitude coordinate of the issue’s location. |
| `region` | VARCHAR(100) | NULL | Optional city or area name. |
| `status` | ENUM('reported', 'in_progress', 'resolved') | DEFAULT `'reported'` | Represents the issue’s lifecycle stage. |
| `is_hidden` | BOOLEAN | DEFAULT FALSE | Marks whether the issue is hidden due to admin moderation or flags. |
| `created_at` | TIMESTAMPTZ | DEFAULT `now()` | Timestamp of when the issue was created. |
| `updated_at` | TIMESTAMPTZ | DEFAULT `now()` | Timestamp of the most recent update. |
| `deleted_at` | TIMESTAMPTZ | NULLABLE | Soft deletion timestamp for archival or recovery. |

---

### **3.8 issue_images**

Stores image URLs related to user-reported issues.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `report_id` | BIGINT | FK → `user_issue.id`, NOT NULL |
| `url` | TEXT | NOT NULL |
| `created_at` | TIMESTAMPTZ | DEFAULT now() |

---

### **3.9 flags**

Defines flag categories for moderation (e.g., spam, duplicate).

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `name` | VARCHAR(100) | UNIQUE, NOT NULL |
| `description` | TEXT | NULL |

---

### **3.10 user_issue_flag**

Tracks flags applied by users to reported issues.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `report_id` | BIGINT | FK → `user_issue.id`, NOT NULL |
| `user_id` | BIGINT | FK → `users.id`, NOT NULL |
| `flag_id` | BIGINT | FK → `flags.id`, NOT NULL |
| `created_at` | TIMESTAMPTZ | DEFAULT now() |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() |

---

### **3.11 logs**

Stores historical changes for issue status updates and comments.

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | BIGINT | PK |
| `issue_id` | BIGINT | FK → `user_issue.id`, NOT NULL |
| `updated_by` | BIGINT | FK → `users.id`, NOT NULL |
| `from_status` | VARCHAR(30) | NULL |
| `to_status` | VARCHAR(30) | NULL |
| `comment` | TEXT | NULL |
| `created_at` | TIMESTAMPTZ | DEFAULT now() |
| `updated_at` | TIMESTAMPTZ | DEFAULT now() |
| `deleted_at` | TIMESTAMPTZ | NULLABLE | Soft delete marker |

---

## **4. ENUMs and Status Definitions**

| Enum Type | Values | Description |
|------------|---------|-------------|
| `issue_status` | `('reported', 'in_progress', 'resolved')` | Defines the issue lifecycle |


#### **Example SQL**
```sql
CREATE TYPE issue_status AS ENUM ('reported', 'in_progress', 'resolved');
```


---
## **6. Index Summary**

<!-- | **Table** | **Index Name** | **Index Type** | **Columns** | **Purpose / Use Case** | **SQL Example** |
|-----------|----------------|----------------|-------------|-------------------------|------------------|
| `users` | `idx_users_email_unique` | UNIQUE (BTREE) | `email` | Prevent duplicate user accounts and speed up authentication | `CREATE UNIQUE INDEX idx_users_email_unique ON users(email);` |
| `user_issue` | `idx_user_issue_status` | BTREE | `status` | Quickly filter issues by status (`reported`, `in_progress`, `resolved`) | `CREATE INDEX idx_user_issue_status ON user_issue(status);` |
| `user_issue` | `idx_user_issue_authority_status` | COMPOSITE (BTREE) | `authority_id, status` | Speed up authority dashboards when filtering issues by both status and assigned authority | `CREATE INDEX idx_user_issue_authority_status ON user_issue(authority_id, status);` |
| `roles` | `idx_roles_name_unique` | UNIQUE (BTREE) | `name` | Ensure no duplicate roles exist in the system | `CREATE UNIQUE INDEX idx_roles_name_unique ON roles(name);` |
| `issues` | `idx_issues_slug_unique` | UNIQUE (BTREE) | `slug` | Ensure issue category slugs are unique and URL-safe | `CREATE UNIQUE INDEX idx_issues_slug_unique ON issues(slug);` |
| `user_role` | `idx_user_role_unique` | UNIQUE COMPOSITE (BTREE) | `user_id, role_id` | Prevent assigning the same role to a user more than once | `CREATE UNIQUE INDEX idx_user_role_unique ON user_role(user_id, role_id);` |
| `authority_user` | `idx_authority_user_unique` | UNIQUE COMPOSITE (BTREE) | `authority_id, user_id` | Enforce one-to-one mapping between authority and user account | `CREATE UNIQUE INDEX idx_authority_user_unique ON authority_user(authority_id, user_id);` |
| `logs` | `idx_logs_issue_id` | BTREE | `issue_id` | Fetch issue logs quickly for timelines and history views | `CREATE INDEX idx_logs_issue_id ON logs(issue_id);` | -->





| **Table**         | **Index Name**                  | **Index Type** | **Columns**    | **Purpose / Use Case**                                                        |
| ----------------- | ------------------------------- | -------------- | -------------- | ----------------------------------------------------------------------------- |
| `users`           | `idx_users_email_unique`        | UNIQUE (BTREE) | `email`        | Prevent duplicate user accounts and speed up authentication                   |
| `user_issue`      | `idx_user_issue_status`         | BTREE          | `status`       | Quickly filter issues by their status (`reported`, `in_progress`, `resolved`) |
| `user_issue`      | `idx_user_issue_authority`      | BTREE          | `authority_id` | Efficiently fetch issues assigned to a specific authority                     |
| `logs`            | `idx_logs_issue_id`             | BTREE          | `issue_id`     | Retrieve activity logs for a particular issue quickly                         |
| `user_issue_flag` | `idx_user_issue_flag_report_id` | BTREE          | `report_id`    | Enable quick lookup of all flags raised for a specific issue                  |

---
# **CivicTrack API Documentation**

---

## **1. Overview**
This API documentation defines the routes for the **CivicTrack System**, which manages user authentication, issue reporting, moderation, and category assignments.  
All endpoints are prefixed with `/api/` and secured by **JWT authentication** through HTTP-only cookies.

---

## **2. Authentication & Authorization Routes**

| **Endpoint** | **Method** | **Access** | **Description** |
|---------------|-------------|-------------|-----------------|
| `/api/auth/signup` | POST | Public | Allows a **Citizen** to register by providing name, email, and password. |
| `/api/auth/login` | POST | Public | Authenticates a user (Citizen, Authority, or Admin) and sets a secure cookie. |
| `/api/auth/logout` | POST | Authenticated | Clears authentication cookie and ends the session. |
| `/api/auth/me` | GET | Authenticated | Returns logged-in user details and assigned role. |

**Notes:**
- Authentication uses cookies instead of headers.  
- Passwords are hashed using bcrypt.  
- All routes are protected using middleware that verifies user roles.  

---

## **3. User Management (Admin Only)**

| **Endpoint** | **Method** | **Access** | **Description** |
|---------------|-------------|-------------|-----------------|
| `/api/users` | GET | Admin | Lists all users (Citizens, Authorities, Admins). |
| `/api/users/:id` | GET | Admin | Retrieves user details by ID. |
| `/api/users` | POST | Admin | Creates a new Authority or Admin account. |
| `/api/users/:id` | PUT | Admin | Updates user details (name, role, status). |
| `/api/users/:id` | DELETE | Admin | Permanently deletes a user record. |

**Notes:**
- Authorities are created by Admins only.  
- Admins can modify roles and deactivate users if needed.  

---

## **4. Issue Management**

| **Endpoint** | **Method** | **Access** | **Description** |
|---------------|-------------|-------------|-----------------|
| `/api/issues` | POST | Citizen | Allows a citizen to report a civic issue with title, description, location, and photo. |
| `/api/issues` | GET | Citizen, Authority, Admin | Fetches all issues. Supports optional filters via query params: `?status=resolved&category=water`. |
| `/api/issues/:id` | GET | Citizen, Authority, Admin | Retrieves details of a specific issue. |
| `/api/issues/:id/status` | PUT | Authority | Updates issue status (e.g., Reported → In Progress → Resolved). Automatically creates a log entry. |
| `/api/issues/:id/logs` | GET | Authority, Admin | Returns all status change logs for the given issue. |
| `/api/issues/:id` | DELETE | Admin | Removes an issue from the system permanently. |

**Notes:**
- Each issue is automatically linked to the reporter (Citizen).  
- Logs are **automatically generated** whenever status changes.  
- Images are uploaded and stored in **AWS S3**.  
- Status flow: `Reported → In Progress → Resolved`.  

---

## **5. Flagging & Moderation**

| **Endpoint** | **Method** | **Access** | **Description** |
|---------------|-------------|-------------|-----------------|
| `/api/flags` | POST | Citizen | Flags an issue as inappropriate or duplicate. |
| `/api/flags` | GET | Admin | Retrieves a list of flagged issues for review. |
| `/api/flags/:issueId/review` | PUT | Admin | Admin reviews and updates issue visibility (hide/unhide). |

**Notes:**
- A flagged issue becomes hidden after a threshold number of flags.  
- Admin can restore or permanently hide flagged content.  

---

## **6. Category Management**

| **Endpoint** | **Method** | **Access** | **Description** |
|---------------|-------------|-------------|-----------------|
| `/api/categories` | GET | Public | Returns all available issue categories (e.g., Roads, Water, Sanitation). |
| `/api/categories` | POST | Admin | Creates a new issue category. |
| `/api/categories/:id` | PUT | Admin | Updates category name or description. |
| `/api/categories/:id` | DELETE | Admin | Deletes a category permanently. |
| `/api/categories/assign` | PUT | Authority | Allows an authority to choose or change which categories they handle. |

**Notes:**
- Authorities can self-assign categories through `/assign`.  
- Each category can have multiple authorities.  

---

## **7. Issue Logs (Auto-Generated)**

| **Endpoint** | **Method** | **Access** | **Description** |
|---------------|-------------|-------------|-----------------|
| `/api/issues/:id/logs` | GET | Authority, Admin | Fetches the complete log history for a specific issue. |

**Log Behavior:**
- Logs are created automatically **only for user issues**.  
- Each log entry includes:
  - `issue_id`  
  - `old_status → new_status`  
  - `updated_by` (Authority ID)  
  - `timestamp`  
- Logs cannot be edited or deleted manually.  

**Examples of Logged Events:**
- Status changed from *Reported* to *In Progress*  
- Status changed from *In Progress* to *Resolved*  

---

## **8. Common Query Parameters**

| **Parameter** | **Type** | **Used In** | **Description** |
|----------------|-----------|--------------|-----------------|
| `status` | String | `/api/issues` | Filter issues by status (reported/in_progress/resolved). |
| `category` | String | `/api/issues` | Filter issues by category name. |
| `page` | Number | `/api/issues` | Paginate issue results. |
| `limit` | Number | `/api/issues` | Control number of issues returned per page. |

---

## **9. Status Codes Summary**

| **Code** | **Meaning** | **When Used** |
|-----------|-------------|---------------|
| **200 OK** | Request successful | Successful GET/PUT/POST/DELETE actions |
| **201 Created** | New record created | On signup or issue creation |
| **400 Bad Request** | Validation or missing fields | Invalid input from user |
| **401 Unauthorized** | User not logged in | Missing/invalid cookie |
| **403 Forbidden** | Access denied | Role does not have permission |
| **404 Not Found** | Resource missing | Record does not exist |
| **500 Server Error** | Internal system error | Unexpected backend failure |

---

## **10. Summary**

| **Module** | **Purpose** |
|-------------|-------------|
| **Auth** | Handles secure login/logout and role-based access |
| **Users** | Admin-only user management and role assignment |
| **Issues** | Reporting and updating civic issues with geolocation and photo support |
| **Flags** | Allows users to report spam or duplicate issues |
| **Categories** | Defines and manages issue types; authorities choose categories |
| **Issue Logs** | Tracks all issue status changes automatically |

---
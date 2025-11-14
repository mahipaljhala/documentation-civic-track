# **Authentication & Authorization Module - CivicTrack**

---

## **1. Authentication & Authorization Sequence Diagram**
![Sequence Diagram](../images/auth-seq.png)
---

## **2. Authentication Flow**

#### **Step-by-Step Explanation**

1. **User Login Request**
   - The user enters their email and password in the frontend (React).
   - This data is sent to the backend API via a secure HTTPS POST request.

2. **Password Verification**
   - The backend fetches the user record from PostgreSQL.
   - The stored password hash is compared using **bcrypt**.

3. **JWT Creation**
   - If valid, the backend generates a **JWT** containing:
     - `user_id`
     - `role_id` (citizen / authority / admin)
     - Token expiry (e.g., 24 hours)
   - The JWT is signed using a secret key stored in `.env`.

4. **Token Storage (Secure Cookie)**
   - Instead of sending the token in headers, it is stored in a **secure HTTP-only cookie**:
     - `Secure: true` → sent only over HTTPS  
     - `SameSite: strict` → prevents cross-site request forgery  
     - `HttpOnly: true` → cannot be read by JavaScript  

5. **Login Success**
   - The backend sends a success message (`200 OK`) and sets the cookie in the browser.
   - The frontend stores **no sensitive data** — only relies on cookies for authentication.

---

## **3. Authorization Flow**

After login, every API request must include the cookie automatically.

1. **Frontend Request**
   - When a user visits a protected page (e.g., `/api/issues`), the browser sends the cookie.

2. **Backend Middleware (Auth Check)**
   - The backend verifies the JWT signature and expiry.
   - If valid, it extracts the user’s `role` and attaches it to the request object.

3. **Role-Based Access**
   - The backend checks user permissions:
     - **Citizen** → can report/view own issues  
     - **Authority** → can view and update issues of assigned categories  
     - **Admin** → can manage users, categories, and flags  

4. **Access Decision**
   - If permitted, the backend responds with `200 OK`.
   - Otherwise, it returns `403 Forbidden`.

---

## **4. Logout Process**

- When the user clicks **Logout**, the backend clears the cookie:
  ```js
  res.clearCookie("token", {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
  });
  ```
- This invalidates the session immediately.

---


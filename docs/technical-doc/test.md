# **CivicTrack – Test Plan**  
---
## **1. Scope**

#### **Scope**
- User Registration (UC-01)  
- Login & Authentication (UC-02)  
- Report Issue (UC-03)  
- Update Issue Status (UC-04)  
- Flag Issue (UC-05)  
- Review Flagged Issues (UC-06)  
- Manage Users (UC-07)  
- Change Password (UC-08)  
- Image upload to AWS S3  
- API testing (Postman)  
- Jest Unit + Integration testing  

<!-- ### **Out of Scope**
- Load / Performance testing  
- Penetration testing  
- Multi-language support  
- Mobile app testing   -->

---

## **2. Test Approach**

#### **A) Manual Testing**
- All user flows tested through UI  
- Validate redirects, form validations, permissions  
- Verify dashboards for Citizen, Authority, Admin  

#### **B) API Testing (Postman)**
Check the following:
- Status codes  
- Response structure  
- Cookie-based authentication  
- Authorization errors (403, 401)

#### **C) Jest Testing**
- Unit tests for controllers, services  
- Middleware tests (auth, role check)  
- S3 upload mocked  
- API integration tests using Supertest  

#### **D) Database Validation**
- Verify FK relations  
- Check unique constraints  
- Indexes working properly (quick filtering)

---

## **3. Features to Be Tested**

#### **UC-01 Register User**
- Valid/invalid inputs  
- Duplicate email handling  

#### **UC-02 Login**
- Correct credentials  
- Wrong password  
- JWT stored in HTTP-only cookie  

#### **UC-03 Report Issue**
- Submit issue with/without image  
- Upload image to S3  
- Issue stored with correct status  

#### **UC-04 Update Issue Status**
- Authority updates to "in_progress" / "resolved"  
- Unauthorized users blocked  

#### **UC-05 Flag Issue**
- Single user can't flag twice  
- Auto-hide after threshold  

#### **UC-06 Review Flagged Issues**
- Admin hides/restores issue  

#### **UC-07 Manage Users**
- Assign/remove roles  
- Check role-based access  

#### **UC-08 Change Password**
- Valid current password  
- Password hashing  
- Session reset  

---

## **4. Entry Criteria**
- Backend API up and running  
- Database configured  
- S3 bucket working  
- Postman collection ready  

---

## **5. Exit Criteria**
- All test cases executed  
- No critical bugs open  
- Core flows functioning correctly  

---

## **6. Risks & Mitigation**

| Risk | Mitigation |
|------|------------|
| Incorrect AWS permissions | Test IAM policy early |
| Deployment issues | Use staging environment |
| Unclear requirements | Regular communication |
<!-- | Cookie issues behind ALB | Validate HTTPS settings | -->

---

## **7. Deliverables**
- Test Plan  
- Test Cases  
- Test Execution Report  
- Bug Report  
- Final Testing Summary  

---

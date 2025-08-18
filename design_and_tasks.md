# Field Officer Mobile App - Design and Tasks Document

## Project Overview

**Project Name:** Field Officer Mobile Application  
**Technology Stack:**
- Frontend: Flutter with Riverpod state management
- Backend: FastAPI with Supabase for Auth and Database
- Database: Supabase (PostgreSQL)
- Storage: Supabase Storage for file attachments

**Project Duration:** 12-16 weeks (estimated)  
**Team Size:** 3-4 developers (1 Flutter, 1 Backend, 1 Full-stack, 1 QA)

---

## High-Level Architecture

### System Components
1. **Flutter Mobile App** - Field Officer interface
2. **FastAPI Backend** - REST API server
3. **Supabase Database** - User data, audits, clients
4. **Supabase Auth** - Authentication and authorization
5. **Supabase Storage** - File and image storage
6. **Email Service** - Notification system (SendGrid/Postmark)

### Data Flow
```
Mobile App → FastAPI Backend → Supabase Database/Auth/Storage
                ↓
        Email Notifications → Operations Manager
```

---

## Database Schema Design

### Core Tables

#### 1. Profiles Table
```sql
profiles (
    id UUID PRIMARY KEY REFERENCES auth.users,
    email VARCHAR NOT NULL,
    role VARCHAR DEFAULT 'field_officer',
    full_name VARCHAR,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
)
```

#### 2. Clients Table
```sql
clients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR NOT NULL,
    location VARCHAR NOT NULL,
    site_code VARCHAR UNIQUE NOT NULL, -- For QR/RFID scanning
    key_contacts JSONB,
    allocated_staff_count INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
)
```

#### 3. Visits Table
```sql
visits (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_id UUID REFERENCES clients(id),
    field_officer_id UUID REFERENCES profiles(id),
    visit_date DATE NOT NULL,
    frequency VARCHAR CHECK (frequency IN ('daily', 'weekly', 'fortnightly', 'monthly')),
    status VARCHAR DEFAULT 'pending' CHECK (status IN ('pending', 'completed')),
    created_at TIMESTAMP DEFAULT NOW()
)
```

#### 4. Audits Table
```sql
audits (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_id UUID REFERENCES clients(id),
    field_officer_id UUID REFERENCES profiles(id),
    audit_entry_token VARCHAR UNIQUE NOT NULL,
    
    -- Attendance Section
    attendance_verified BOOLEAN,
    attendance_attachment_url VARCHAR,
    staff_allocated_count INTEGER,
    staff_actual_count INTEGER,
    attendance_observations TEXT,
    
    -- Uniform & Grooming
    uniform_grooming_status VARCHAR,
    uniform_grooming_issues TEXT,
    uniform_grooming_attachment_url VARCHAR,
    
    -- Patrol Logs
    patrol_logs_status VARCHAR,
    patrol_logs_issues TEXT,
    patrol_logs_attachment_url VARCHAR,
    
    -- Incident Reports
    incident_reports_reviewed BOOLEAN,
    incident_reports_summary TEXT,
    
    -- Assets (JSONB for flexibility)
    assets_review JSONB,
    
    -- SOP Compliance (JSONB for structured data)
    sop_compliance JSONB,
    
    -- Emergency Readiness
    emergency_readiness_status VARCHAR,
    emergency_readiness_remarks TEXT,
    
    -- Client Meeting
    client_meeting_status VARCHAR,
    client_contact_name VARCHAR,
    client_feedback_summary TEXT,
    
    -- Payments
    payments_status VARCHAR,
    payments_remarks TEXT,
    
    -- Closing
    closing_recommendations TEXT,
    
    -- Metadata
    submitted_at TIMESTAMP DEFAULT NOW(),
    created_at TIMESTAMP DEFAULT NOW()
)
```

---

## Task Breakdown and Dependencies

### Phase 1: Authentication & Security (Weeks 1-3)

#### Task 1.1: Backend Authentication Setup
**Estimated Time:** 5 days  
**Dependencies:** None  
**Assignee:** Backend Developer

**Subtasks:**
- [ ] Set up FastAPI project structure with Supabase integration
- [ ] Implement `/auth/register` endpoint with email domain validation
- [ ] Implement `/auth/login` endpoint with JWT generation
- [ ] Set up JWT middleware for protected routes
- [ ] Create profiles table and registration logic
- [ ] Write unit tests for auth endpoints

**Requirements Coverage:**
- ✅ Story 1.1: User Registration (Backend)
- ✅ Story 1.2: User Login (Backend)
- ✅ Story 1.5: Secure Access and Token Management (Backend)

#### Task 1.2: Flutter Authentication UI
**Estimated Time:** 4 days  
**Dependencies:** Task 1.1  
**Assignee:** Flutter Developer

**Subtasks:**
- [ ] Set up Flutter project with Riverpod
- [ ] Create registration screen with validation
- [ ] Create login screen with error handling
- [ ] Implement JWT storage using flutter_secure_storage
- [ ] Set up authentication state management with Riverpod
- [ ] Implement HTTP interceptor for token management

**Requirements Coverage:**
- ✅ Story 1.1: User Registration (Frontend)
- ✅ Story 1.2: User Login (Frontend)
- ✅ Story 1.5: Secure Access and Token Management (Frontend)

#### Task 1.3: PIN and Biometric Authentication
**Estimated Time:** 4 days  
**Dependencies:** Task 1.2  
**Assignee:** Flutter Developer

**Subtasks:**
- [ ] Implement PIN setup and validation logic
- [ ] Create PIN entry UI with attempt tracking
- [ ] Integrate local_auth package for biometric support
- [ ] Implement fallback mechanisms (PIN → Password)
- [ ] Add biometric toggle in settings
- [ ] Handle security scenarios (max attempts, logout)

**Requirements Coverage:**
- ✅ Story 1.3: Secure PIN Login (Frontend)
- ✅ Story 1.4: Biometric Login (Frontend)

---

### Phase 2: Dashboard & Visit Management (Weeks 3-5)

#### Task 2.1: Backend Visit Management
**Estimated Time:** 4 days  
**Dependencies:** Task 1.1  
**Assignee:** Backend Developer

**Subtasks:**
- [ ] Create clients and visits tables
- [ ] Implement `/visits` endpoint with time-based filtering
- [ ] Add visit status tracking logic
- [ ] Create database seed scripts for test data
- [ ] Write API tests for visit endpoints

**Requirements Coverage:**
- ✅ Story 2.1: View Visit Schedules (Backend)
- ✅ Story 2.2: Visual Cue for Completed Audits (Backend)

#### Task 2.2: Dashboard UI Implementation
**Estimated Time:** 5 days  
**Dependencies:** Task 2.1, Task 1.2  
**Assignee:** Flutter Developer

**Subtasks:**
- [ ] Create dashboard screen with tabbed layout
- [ ] Implement visit lists for different time periods
- [ ] Add visual indicators for completed audits
- [ ] Implement pull-to-refresh functionality
- [ ] Add navigation to audit start screen
- [ ] Handle empty states and error scenarios

**Requirements Coverage:**
- ✅ Story 2.1: View Visit Schedules (Frontend)
- ✅ Story 2.2: Visual Cue for Completed Audits (Frontend)

---

### Phase 3: Audit Workflow - Site Validation (Weeks 5-7)

#### Task 3.1: Site Validation Backend
**Estimated Time:** 3 days  
**Dependencies:** Task 2.1  
**Assignee:** Backend Developer

**Subtasks:**
- [ ] Implement `/audits/validate-site` endpoint
- [ ] Add audit entry token generation logic
- [ ] Create site code validation against clients table
- [ ] Return client data for audit pre-population
- [ ] Add error handling for invalid codes

**Requirements Coverage:**
- ✅ Story 3.1: Site Validation via Scanner (Backend)

#### Task 3.2: QR/Barcode Scanner Implementation
**Estimated Time:** 4 days  
**Dependencies:** Task 3.1, Task 2.2  
**Assignee:** Flutter Developer

**Subtasks:**
- [ ] Integrate mobile_scanner package
- [ ] Create scanning UI with camera overlay
- [ ] Implement scan result validation flow
- [ ] Add navigation logic (success → audit, failure → dashboard)
- [ ] Handle camera permissions and errors
- [ ] Store validated client data in state

**Requirements Coverage:**
- ✅ Story 3.1: Site Validation via Scanner (Frontend)

---

### Phase 4: Audit Checklist Implementation (Weeks 7-11)

#### Task 4.1: Audit Data Model and Backend
**Estimated Time:** 6 days  
**Dependencies:** Task 3.1  
**Assignee:** Backend Developer

**Subtasks:**
- [ ] Create comprehensive audits table schema
- [ ] Implement `/audits` POST endpoint for submission
- [ ] Add file upload handling for Supabase Storage
- [ ] Implement audit entry token validation
- [ ] Set up email notification system
- [ ] Add data validation and error handling
- [ ] Write comprehensive API tests

**Requirements Coverage:**
- ✅ Story 4.1: Complete a Site Audit (Backend)
- ✅ Story 5.1: Submit the Audit and Get Confirmation (Backend)

#### Task 4.2: Audit Form UI - Basic Structure
**Estimated Time:** 4 days  
**Dependencies:** Task 3.2  
**Assignee:** Flutter Developer

**Subtasks:**
- [ ] Create scrollable audit form layout
- [ ] Implement form state management with Riverpod
- [ ] Add client information display header
- [ ] Create reusable form components
- [ ] Implement form validation logic
- [ ] Add image picker functionality

**Requirements Coverage:**
- ✅ Story 4.1: Complete a Site Audit (Frontend - Structure)

#### Task 4.3: Audit Form Sections Implementation
**Estimated Time:** 8 days  
**Dependencies:** Task 4.2  
**Assignee:** Flutter Developer + Full-stack Developer

**Subtasks:**
- [ ] Implement Attendance Verification section (4.1.1)
- [ ] Implement Uniform & Grooming section (4.1.2)
- [ ] Implement Patrol Logs section (4.1.3)
- [ ] Implement Incident Reports Review section (4.1.4)
- [ ] Implement Assets Review section (4.1.5)
- [ ] Implement SOP Compliance section (4.1.6)
- [ ] Implement Emergency Readiness section (4.1.7)
- [ ] Implement Client Meeting Status section (4.1.8)
- [ ] Implement Payments Status section (4.1.9)
- [ ] Implement Closing Recommendations section (4.1.10)

**Requirements Coverage:**
- ✅ All subsections of Story 4.1: Complete a Site Audit

#### Task 4.4: Audit Submission and Completion
**Estimated Time:** 3 days  
**Dependencies:** Task 4.1, Task 4.3  
**Assignee:** Flutter Developer

**Subtasks:**
- [ ] Implement submission logic with loading states
- [ ] Add success/error handling and user feedback
- [ ] Implement navigation back to dashboard
- [ ] Add offline submission capability (stretch goal)
- [ ] Update dashboard state after submission

**Requirements Coverage:**
- ✅ Story 5.1: Submit the Audit and Get Confirmation (Frontend)

---

### Phase 5: Testing and Deployment (Weeks 11-12)

#### Task 5.1: Integration Testing
**Estimated Time:** 4 days  
**Dependencies:** All previous tasks  
**Assignee:** QA Engineer + Team

**Subtasks:**
- [ ] End-to-end testing of complete audit workflow
- [ ] Cross-device testing (Android/iOS)
- [ ] Network failure and offline scenarios testing
- [ ] Performance testing with large data sets
- [ ] Security testing for authentication flows
- [ ] User acceptance testing

#### Task 5.2: Deployment and DevOps
**Estimated Time:** 2 days  
**Dependencies:** Task 5.1  
**Assignee:** Full-stack Developer

**Subtasks:**
- [ ] Set up production Supabase instance
- [ ] Deploy FastAPI backend to cloud (AWS/GCP/Azure)
- [ ] Configure environment variables and secrets
- [ ] Set up monitoring and logging
- [ ] Create deployment documentation
- [ ] App store preparation (if applicable)

---

## Risk Assessment and Mitigation

### High Risk Items
1. **RFID Integration Complexity**
   - **Risk:** RFID scanning may require device-specific implementations
   - **Mitigation:** Start with QR/Barcode in Phase 1, research RFID as separate POC

2. **File Upload Performance**
   - **Risk:** Large image uploads may cause performance issues
   - **Mitigation:** Implement image compression and progress indicators

3. **Offline Functionality**
   - **Risk:** Network connectivity issues in field locations
   - **Mitigation:** Implement local storage for draft audits, sync when online

### Medium Risk Items
1. **Email Notification Reliability**
   - **Mitigation:** Implement retry logic and fallback notification methods

2. **JWT Token Expiration Handling**
   - **Mitigation:** Implement automatic refresh token logic

---

## Success Criteria

### Functional Requirements
- [ ] Field officers can register and login securely
- [ ] Dashboard shows visit schedules organized by time periods
- [ ] Site validation prevents audits at wrong locations
- [ ] Complete audit checklist can be filled and submitted
- [ ] Operations managers receive email notifications

### Non-Functional Requirements
- [ ] App loads within 3 seconds on average mobile devices
- [ ] 99.5% uptime for backend services
- [ ] Support for 100+ concurrent users
- [ ] Secure handling of sensitive data (GDPR/compliance)
- [ ] Cross-platform compatibility (Android/iOS)

---

## Future Enhancements (Post-Phase 1)

### Phase 2 Enhancements
- Master data management for incidents
- Advanced reporting and analytics
- Bulk operations for visit scheduling

### Phase 3 Enhancements
- Asset master data integration
- Advanced offline capabilities
- Real-time notifications

### Operations Manager Web Interface
- Comprehensive dashboard for audit review
- User and client management
- Advanced analytics and reporting
- Master data management tools

---

## Conclusion

This document provides a comprehensive roadmap for developing the Field Officer Mobile Application. The phased approach ensures systematic development with clear dependencies and measurable outcomes. Regular sprint reviews and stakeholder feedback will be essential for successful delivery.

**Next Steps:**
1. Team assignment and sprint planning
2. Environment setup (development, staging, production)
3. Sprint 1 kickoff with authentication implementation
4. Regular stakeholder demos and feedback sessions
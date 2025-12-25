
# Permit-to-Work System: Feature Mapping Document

## Overview
This document maps the SafetyPermitManager features to the Atlas CMMS implementation, following CMMS architecture patterns.

---

## 1. Entity Mapping

### SafetyPermitManager Entities → CMMS Entities

| SafetyPermitManager Entity | CMMS Entity | Notes |
|---------------------------|-------------|-------|
| `permits` | `Permit.java` | Extends `OwnUser`, includes comprehensive permit fields |
| `work_locations` | `PermitLocation.java` | Extends `OwnUser`, includes map integration |
| `moc_change_requests` | `ManagementOfChange.java` | Extends `OwnUser`, main MoC entity |
| `moc_actions` | `ManagementOfChangeAction.java` | Extends `OwnUser`, actions for MoC |
| `moc_approvals` | `ManagementOfChangeApproval.java` | Extends `OwnUser`, approvals for MoC |
| `permit_attachments` | `PermitAttachment.java` | Extends `OwnUser`, file attachments |
| `moc_attachments` | `ManagementOfChangeAttachment.java` | Extends `OwnUser`, file attachments |
| `users` | Use existing `User.java` | Leverage existing user system |
| `sessions` | Use existing CMMS auth | Use JWT-based authentication |
| `notifications` | Use existing `Notification.java` | Leverage existing notification system |
| `map_backgrounds` | Use existing map features | Integrate with CMMS map system |
| `system_settings` | Use existing company settings | Extend company settings |

---

## 2. Database Schema Mapping

### Permit Table (Main Entity)
```sql
CREATE TABLE permit (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    permit_number VARCHAR(100) UNIQUE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL, -- 'confined_space', 'hot_work', 'electrical', 'chemical', 'height'
    status VARCHAR(50) NOT NULL, -- 'DRAFT', 'PENDING', 'APPROVED', 'ACTIVE', 'COMPLETED', 'EXPIRED', 'REJECTED'
    risk_level VARCHAR(50), -- 'LOW', 'MEDIUM', 'HIGH', 'CRITICAL'
    overall_risk VARCHAR(50),
    start_date TIMESTAMP,
    end_date TIMESTAMP,
    location_id BIGINT,
    work_location_id BIGINT,
    map_position_x DOUBLE,
    map_position_y DOUBLE,
    requestor_id BIGINT,
    department VARCHAR(255) NOT NULL,
    contact_number VARCHAR(100) NOT NULL,
    emergency_contact VARCHAR(100) NOT NULL,
    safety_officer VARCHAR(255),
    department_head VARCHAR(255),
    maintenance_approver VARCHAR(255),
    identified_hazards TEXT,
    additional_comments TEXT,
    selected_hazards TEXT, -- JSON array
    hazard_notes TEXT, -- JSON object
    completed_measures TEXT, -- JSON array
    immediate_actions TEXT,
    before_work_starts TEXT,
    compliance_notes TEXT,
    status_history TEXT, -- JSON array
    submitted_at TIMESTAMP,
    approved_at TIMESTAMP,
    activated_at TIMESTAMP,
    completed_at TIMESTAMP,
    submitted_by BIGINT,
    performer_name VARCHAR(255),
    performer_signature TEXT,
    work_started_at TIMESTAMP,
    work_completed_at TIMESTAMP,
    company_id BIGINT NOT NULL,
    created_by BIGINT NOT NULL,
    updated_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (location_id) REFERENCES permit_location(id),
    FOREIGN KEY (work_location_id) REFERENCES location(id),
    FOREIGN KEY (requestor_id) REFERENCES user(id),
    FOREIGN KEY (submitted_by) REFERENCES user(id),
    FOREIGN KEY (company_id) REFERENCES company(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id)
);
```

### Permit Location Table
```sql
CREATE TABLE permit_location (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT,
    building VARCHAR(255),
    area VARCHAR(255),
    map_position_x INTEGER,
    map_position_y INTEGER,
    map_background_id BIGINT,
    is_active BOOLEAN DEFAULT TRUE,
    company_id BIGINT NOT NULL,
    created_by BIGINT NOT NULL,
    updated_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (map_background_id) REFERENCES map_background(id),
    FOREIGN KEY (company_id) REFERENCES company(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id)
);
```

### Management of Change Tables
```sql
CREATE TABLE management_of_change (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    affected_systems TEXT, -- JSON array
    risk_level VARCHAR(50) NOT NULL, -- 'LOW', 'MEDIUM', 'HIGH'
    planned_commissioning_date TIMESTAMP,
    status VARCHAR(50) NOT NULL, -- 'DRAFT', 'SUBMITTED', 'IN_REVIEW', 'APPROVED', 'REJECTED', 'COMPLETED'
    company_id BIGINT NOT NULL,
    created_by BIGINT NOT NULL,
    updated_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    FOREIGN KEY (company_id) REFERENCES company(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id)
);

CREATE TABLE management_of_change_action (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    change_request_id BIGINT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    responsible_user_id BIGINT,
    due_timing VARCHAR(50) NOT NULL, -- 'BEFORE_COMMISSIONING', 'AFTER_COMMISSIONING'
    due_date TIMESTAMP,
    status VARCHAR(50) NOT NULL, -- 'OPEN', 'IN_PROGRESS', 'COMPLETED'
    completion_date TIMESTAMP,
    company_id BIGINT NOT NULL,
    created_by BIGINT NOT NULL,
    updated_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (change_request_id) REFERENCES management_of_change(id),
    FOREIGN KEY (responsible_user_id) REFERENCES user(id),
    FOREIGN KEY (company_id) REFERENCES company(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id)
);

CREATE TABLE management_of_change_approval (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    change_request_id BIGINT NOT NULL,
    department VARCHAR(255) NOT NULL,
    approver_user_id BIGINT,
    status VARCHAR(50) NOT NULL, -- 'PENDING', 'APPROVED', 'REJECTED', 'NOT_REQUIRED'
    approved_at TIMESTAMP,
    comments TEXT,
    company_id BIGINT NOT NULL,
    created_by BIGINT NOT NULL,
    updated_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (change_request_id) REFERENCES management_of_change(id),
    FOREIGN KEY (approver_user_id) REFERENCES user(id),
    FOREIGN KEY (company_id) REFERENCES company(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id)
);
```

---

## 3. API Endpoint Mapping

### Permit Endpoints
| SafetyPermitManager | CMMS | Method | Description |
|--------------------|------|--------|-------------|
| `/api/permits` | `/api/permits` | GET | Get all permits |
| `/api/permits` | `/api/permits` | POST | Create new permit |
| `/api/permits/:id` | `/api/permits/{id}` | GET | Get permit by ID |
| `/api/permits/:id` | `/api/permits/{id}` | PATCH | Update permit |
| `/api/permits/:id` | `/api/permits/{id}` | DELETE | Delete permit |
| `/api/permits/:id/workflow` | `/api/permits/{id}/workflow` | POST | Update permit workflow |
| `/api/permits/:id/approve` | `/api/permits/{id}/approve` | POST | Approve permit |
| `/api/permits/:id/reject` | `/api/permits/{id}/reject` | POST | Reject permit |
| `/api/permits/stats` | `/api/permits/stats` | GET | Get permit statistics |
| `/api/permits/status/:status` | `/api/permits/status/{status}` | GET | Get permits by status |
| `/api/permits/map` | `/api/permits/map` | GET | Get permits for map display |

### Management of Change Endpoints
| SafetyPermitManager | CMMS | Method | Description |
|--------------------|------|--------|-------------|
| `/api/moc/change-requests` | `/api/management-of-change` | GET | Get all MoC requests |
| `/api/moc/change-requests` | `/api/management-of-change` | POST | Create MoC request |
| `/api/moc/change-requests/:id` | `/api/management-of-change/{id}` | GET | Get MoC by ID |
| `/api/moc/change-requests/:id` | `/api/management-of-change/{id}` | PATCH | Update MoC |
| `/api/moc/change-requests/:id` | `/api/management-of-change/{id}` | DELETE | Delete MoC |
| `/api/moc/change-requests/:id/actions` | `/api/management-of-change/{id}/actions` | GET | Get MoC actions |
| `/api/moc/change-requests/:id/actions` | `/api/management-of-change/{id}/actions` | POST | Add MoC action |
| `/api/moc/change-requests/:id/approvals` | `/api/management-of-change/{id}/approvals` | GET | Get MoC approvals |
| `/api/moc/change-requests/:id/approvals` | `/api/management-of-change/{id}/approvals` | POST | Add MoC approval |

### Permit Location Endpoints
| SafetyPermitManager | CMMS | Method | Description |
|--------------------|------|--------|-------------|
| `/api/work-locations` | `/api/permit-locations` | GET | Get all permit locations |
| `/api/work-locations` | `/api/permit-locations` | POST | Create permit location |
| `/api/work-locations/:id` | `/api/permit-locations/{id}` | GET | Get location by ID |
| `/api/work-locations/:id` | `/api/permit-locations/{id}` | PATCH | Update location |
| `/api/work-locations/:id` | `/api/permit-locations/{id}` | DELETE | Delete location |

---

## 4. Frontend Component Mapping

### Main Components
| SafetyPermitManager Component | CMMS Component | Location |
|-----------------------------|----------------|----------|
| Permit List | `PermitList.tsx` | `/frontend/src/content/own/PermitManagement/PermitList.tsx` |
| Permit Details | `PermitDetails.tsx` | `/frontend/src/content/own/PermitManagement/Details/PermitDetails.tsx` |
| Permit Form | `PermitForm.tsx` | `/frontend/src/content/own/PermitManagement/PermitForm.tsx` |
| MoC List | `ManagementOfChangeList.tsx` | `/frontend/src/content/own/PermitManagement/ManagementOfChangeList.tsx` |
| MoC Details | `ManagementOfChangeDetails.tsx` | `/frontend/src/content/own/PermitManagement/Details/ManagementOfChangeDetails.tsx` |
| Location List | `PermitLocationList.tsx` | `/frontend/src/content/own/PermitManagement/PermitLocationList.tsx` |
| Main Module | `index.tsx` | `/frontend/src/content/own/PermitManagement/index.tsx` |

### Supporting Components
| SafetyPermitManager Component | CMMS Component | Location |
|-----------------------------|----------------|----------|
| Permit Status Badge | `PermitStatusChip.tsx` | `/frontend/src/content/own/PermitManagement/components/PermitStatusChip.tsx` |
| Workflow Buttons | `PermitWorkflowButtons.tsx` | `/frontend/src/content/own/PermitManagement/components/PermitWorkflowButtons.tsx` |
| Map Integration | Use existing Map components | `/frontend/src/content/own/components/Map/` |
| File Upload | Use existing FileUpload | `/frontend/src/content/own/components/FileUpload/` |
| Signature Pad | `SignaturePad.tsx` | `/frontend/src/content/own/PermitManagement/components/SignaturePad.tsx` |

---

## 5. Service and API Mapping

### Backend Services
| SafetyPermitManager | CMMS Service | Notes |
|--------------------|--------------|-------|
| Permit storage | `PermitService.java` | Business logic for permits |
| MoC storage | `ManagementOfChangeService.java` | Business logic for MoC |
| Location storage | `PermitLocationService.java` | Business logic for locations |
| Notification storage | Use existing `NotificationService.java` | Leverage existing service |

### Frontend Services
| SafetyPermitManager | CMMS Service | Location |
|--------------------|--------------|----------|
| Permit API calls | `permits.ts` | `/frontend/src/slices/permits.ts` |
| MoC API calls | `managementOfChange.ts` | `/frontend/src/slices/managementOfChange.ts` |
| Location API calls | `permitLocations.ts` | `/frontend/src/slices/permitLocations.ts` |

---

## 6. Permission Mapping

### Permission Entities
Add to `PermissionEntity.java`:
```java
PERMITS("Permits"),
PERMIT_LOCATIONS("Permit Locations"),
MANAGEMENT_OF_CHANGE("Management of Change")
```

### Role Permissions
Extend existing roles to include permit permissions:
- Admin: Full access to all permit features
- Safety Officer: View, create, approve permits
- Department Head: View, create, approve permits
- Maintenance: View, create, approve permits
- Employee: View, create permits

---

## 7. Implementation Strategy

### Phase 1: Database & Backend
1. Create Liquibase changelog for permit tables
2. Create Java entities following CMMS patterns
3. Create repositories, services, controllers, DTOs, mappers
4. Update permissions and roles
5. Test backend endpoints

### Phase 2: Frontend
1. Create API services (Redux slices)
2. Create TypeScript models
3. Create React components following CMMS patterns
4. Integrate with existing CMMS features
5. Test frontend functionality

### Phase 3: Integration
1. Integrate with existing CMMS modules (users, companies, locations)
2. Add permit-related notifications
3. Add permit-related analytics
4. Add permit-related reports

---

## 8. Key Technical Decisions

1. **Database**: Use Liquibase for schema management (CMMS standard)
2. **Authentication**: Use existing JWT-based authentication
3. **Authorization**: Use @PreAuthorize with PermissionEntity enum
4. **Pagination**: Use Pageable interface for pagination
5. **Error Handling**: Use existing GlobalExceptionHandlerController
6. **Auditing**: Use Hibernate Envers for entity auditing
7. **Validation**: Use Bean validation annotations
8. **Frontend**: Use Redux for state management (CMMS standard)
9. **UI**: Use Material-UI components (CMMS standard)
10. **Internationalization**: Use existing i18n system

---

## 9. Risk Assessment

### High Risk Items
- Complex permit workflow with multiple approval steps
- TRBS hazard assessment integration
- Map integration with existing CMMS maps
- Data migration from SafetyPermitManager (if needed)

### Mitigation Strategies
- Implement workflow step-by-step with thorough testing
- Create comprehensive test cases for hazard assessment
- Reuse existing CMMS map components
- Plan data migration carefully if needed

---

## 10. Testing Strategy

### Backend Testing
- Unit tests for services
- Integration tests for controllers
- API endpoint testing with Postman/curl
- Performance testing for large datasets

### Frontend Testing
- Component tests for React components
- Integration tests for API calls
- End-to-end testing for user flows
- Cross-browser testing
- Mobile responsiveness testing

### User Acceptance Testing
- Permit creation and approval workflow
- Management of Change workflow
- Map integration functionality
- Hazard assessment functionality
- Reporting and analytics

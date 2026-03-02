# Use Case and Entity Relationship Diagrams

Comprehensive visual representations of system functionality and data structure.

---

## 1. Use Case Diagram - Complete System

```mermaid
graph TB
    Admin["👤 Admin User"]
    RegularUser["👤 Regular User"]
    
    subgraph AuthSystem["🔐 Authentication System"]
        Login["Login"]
        Logout["Logout"]
        Register["Register Account"]
        ResetPassword["Reset Password"]
    end
    
    subgraph ProjectMgmt["📋 Project Management"]
        ViewProjects["View All Projects"]
        CreateProject["Create Project"]
        EditProject["Edit Project"]
        DeleteProject["Delete Project"]
        AssignProject["Assign to User"]
        FilterProjects["Filter Projects"]
    end
    
    subgraph TaskMgmt["✅ Task Management"]
        ViewTasks["View Tasks"]
        CreateTask["Create Task"]
        UpdateTask["Update Task Status"]
        AssignTask["Assign Task"]
        SetPriority["Set Priority"]
        DeleteTask["Delete Task"]
        ViewTaskDetail["View Task Details"]
    end
    
    subgraph FileMgmt["📁 File Management"]
        UploadFile["Upload File"]
        PreviewFile["Preview File"]
        DownloadFile["Download File"]
        DeleteFile["Delete File"]
        ListFiles["List All Files"]
    end
    
    subgraph Dashboard["📊 Dashboard & Analytics"]
        ViewStats["View Statistics"]
        ViewCharts["View Charts"]
        ExportData["Export Data"]
        ViewActivityLog["View Activity Log"]
    end
    
    subgraph Notifications["🔔 Notifications"]
        SendReminder["Send Deadline Reminder"]
        NotifyAssignment["Notify Task Assignment"]
        NotifyStatusChange["Notify Status Change"]
    end
    
    subgraph UserMgmt["👥 User Management"]
        ManageUsers["Manage Users"]
        AssignRoles["Assign Roles"]
        SetPermissions["Set Permissions"]
    end
    
    Admin --> AuthSystem
    Admin --> ProjectMgmt
    Admin --> TaskMgmt
    Admin --> FileMgmt
    Admin --> Dashboard
    Admin --> Notifications
    Admin --> UserMgmt
    
    RegularUser --> AuthSystem
    RegularUser --> ViewProjects
    RegularUser --> ViewTasks
    RegularUser --> UpdateTask
    RegularUser --> ViewTaskDetail
    RegularUser --> UploadFile
    RegularUser --> PreviewFile
    RegularUser --> DownloadFile
    RegularUser --> ViewStats
    RegularUser --> ViewActivityLog
    
    CreateProject -.-> AssignProject
    CreateTask -.-> AssignTask
    AssignTask -.-> SendReminder
    UpdateTask -.-> NotifyStatusChange
    
    style AuthSystem fill:#e1f5ff
    style ProjectMgmt fill:#f3e5f5
    style TaskMgmt fill:#fff9c4
    style FileMgmt fill:#f1f8e9
    style Dashboard fill:#fce4ec
    style Notifications fill:#ffe0b2
    style UserMgmt fill:#e0f2f1
```

---

## 2. Actor and Use Case Relationships

```mermaid
graph LR
    subgraph Actors
        Admin["<b>Admin</b><br/>Full Access"]
        User["<b>User</b><br/>Limited Access"]
        System["<b>System</b><br/>Automated Tasks"]
    end
    
    subgraph Primary["Primary Use Cases"]
        UC1["Manage Projects<br/>(CRUD)"]
        UC2["Manage Tasks<br/>(CRUD)"]
        UC3["Manage Files<br/>(Upload/Download)"]
    end
    
    subgraph Secondary["Secondary Use Cases"]
        UC4["View Dashboard"]
        UC5["Track Activity"]
        UC6["Generate Reports"]
    end
    
    subgraph Automated["Automated Use Cases"]
        UC7["Send Reminders"]
        UC8["Log Activities"]
        UC9["Cleanup Files"]
    end
    
    Admin --> UC1
    Admin --> UC2
    Admin --> UC3
    User --> UC2
    User --> UC3
    User --> UC4
    
    Admin --> UC4
    Admin --> UC5
    Admin --> UC6
    User --> UC5
    
    System --> UC7
    System --> UC8
    System --> UC9
    
    style Actors fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style Primary fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Secondary fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style Automated fill:#ffe0b2,stroke:#e65100,stroke-width:2px
```

---

## 3. Entity Relationship Diagram (Detailed)

```mermaid
erDiagram
    USERS ||--|| ROLES : "has"
    USERS ||--o{ PROJECTS : "creates_as_owner"
    USERS ||--o{ PROJECTS : "assigned_to"
    USERS ||--o{ TASKS : "assigned_to"
    USERS ||--o{ FILES : "uploads"
    USERS ||--o{ ACTIVITY_LOGS : "performs"
    
    ROLES ||--o{ PERMISSIONS : "has"
    
    PROJECTS ||--o{ TASKS : "contains"
    PROJECTS ||--o{ FILES : "has"
    PROJECTS ||--o{ ACTIVITY_LOGS : "generates"
    
    TASKS ||--o{ FILES : "may_have"
    TASKS ||--o{ ACTIVITY_LOGS : "generates"
    
    FILES ||--o{ ACTIVITY_LOGS : "generates"
    
    USERS {
        bigint id PK
        string name
        string email UK
        string password
        bigint role_id FK
        string telegram_chat_id
        enum status
        timestamp last_login_at
        timestamp created_at
        timestamp updated_at
    }
    
    ROLES {
        bigint id PK
        string name UK
        text description
        timestamp created_at
        timestamp updated_at
    }
    
    PERMISSIONS {
        bigint id PK
        string name UK
        text description
        string resource
        string action
        timestamp created_at
        timestamp updated_at
    }
    
    PROJECTS {
        bigint id PK
        string name
        string client_name
        text description
        date deadline
        enum status
        bigint assigned_to FK
        bigint created_by FK
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at ST
    }
    
    TASKS {
        bigint id PK
        bigint project_id FK
        string name
        text description
        enum priority
        date deadline
        enum status
        bigint assigned_to FK
        int progress_percentage
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at ST
    }
    
    FILES {
        bigint id PK
        bigint task_id FK
        bigint project_id FK
        bigint uploaded_by FK
        string filename
        string original_filename
        string file_type
        bigint file_size
        string file_path
        string mime_type
        int download_count
        timestamp last_downloaded_at
        timestamp created_at
        timestamp deleted_at ST
    }
    
    ACTIVITY_LOGS {
        bigint id PK
        bigint user_id FK
        string action
        string model_type
        bigint model_id
        text description
        json changes
        string ip_address
        text user_agent
        timestamp created_at
    }
```

---

## 4. Relationship Cardinality Details

### One-to-Many Relationships

```
USER (1) ──────→ (Many) PROJECTS
└─ One user can create multiple projects
└─ One user can be assigned multiple projects

USER (1) ──────→ (Many) TASKS
└─ One user can be assigned multiple tasks

PROJECT (1) ────→ (Many) TASKS
└─ One project can have many tasks

PROJECT (1) ────→ (Many) FILES
└─ One project can have multiple files

TASK (1) ──────→ (Many) FILES
└─ One task can have multiple attachments

ROLE (1) ──────→ (Many) PERMISSIONS
└─ One role can have many permissions
```

### Many-to-Many Relationships

```
USERS (Many) ←──→ (Many) ROLES
└─ Future: Single user could have multiple roles
└─ Currently: Single role per user (via role_id)

ROLES (Many) ←──→ (Many) PERMISSIONS
└─ Implemented via pivot table: role_permissions
└─ One role can have multiple permissions
└─ One permission can be assigned to multiple roles
```

---

## 5. Project Lifecycle Diagram

```mermaid
stateDiagram-v2
    [*] --> Pending: Project Created
    
    Pending --> Active: Approve & Start
    Pending --> Cancelled: Cancel Project
    
    Active --> OnHold: Pause Project
    Active --> Completed: Finish All Tasks
    Active --> Cancelled: Cancel Project
    
    OnHold --> Active: Resume Project
    OnHold --> Cancelled: Cancel Project
    OnHold --> Completed: Mark Complete
    
    Completed --> [*]: Project Archived
    Cancelled --> [*]: Project Removed
    
    note right of Pending
        Project created
        Awaiting approval
        No tasks assigned
    end note
    
    note right of Active
        Project in progress
        Tasks being executed
        Team actively working
    end note
    
    note right of OnHold
        Project paused
        Resources available
        Can resume later
    end note
    
    note right of Completed
        All tasks done
        Project successful
        Ready for archive
    end note
    
    note right of Cancelled
        Project stopped
        May be incomplete
        Reason documented
    end note
```

---

## 6. Task Lifecycle Diagram

```mermaid
stateDiagram-v2
    [*] --> Pending: Task Created
    
    Pending --> InProgress: Start Work
    Pending --> Blocked: Cannot Start
    Pending --> Cancelled: Cancel Task
    
    InProgress --> Completed: Mark Done
    InProgress --> Blocked: Hit Obstacle
    InProgress --> Overdue: Deadline Passed
    InProgress --> Cancelled: Cancel Task
    
    Blocked --> InProgress: Resolve Issue
    Blocked --> Cancelled: Cannot Continue
    
    Overdue --> Completed: Complete Late
    Overdue --> Cancelled: Give Up
    
    Cancelled --> [*]: Task Abandoned
    Completed --> [*]: Task Finished
    
    note right of Pending
        Task created
        Not started
        Awaiting assignment
    end note
    
    note right of InProgress
        Work started
        Making progress
        Not yet complete
    end note
    
    note right of Blocked
        Cannot proceed
        Waiting on something
        Needs resolution
    end note
    
    note right of Overdue
        Deadline passed
        Still working on it
        Priority increases
    end note
    
    note right of Completed
        Work finished
        Ready for review
        Can be archived
    end note
```

---

## 7. File Attachment Relationships

```mermaid
graph TD
    File["📄 File Record<br/>(id, filename, path)"]
    
    File -->|Can be attached to| Task["✅ Task<br/>(task_id)"]
    File -->|Can be attached to| Project["📋 Project<br/>(project_id)"]
    
    Task -->|Part of| Project
    
    File -->|Uploaded by| User["👤 User<br/>(uploaded_by)"]
    
    File -->|Supports types| Types["<b>File Types:</b><br/>PDF, DOCX, XLSX, TXT<br/>JPG, JPEG, PNG, GIF"]
    
    File -->|Validates| Constraints["<b>Constraints:</b><br/>Max Size: 10 MB<br/>Unique Filename<br/>Scannable"]
    
    File -->|Has metadata| Meta["<b>Metadata:</b><br/>file_type<br/>file_size<br/>mime_type<br/>download_count"]
    
    style File fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Task fill:#fff9c4
    style Project fill:#fff9c4
    style User fill:#e1f5ff
    style Types fill:#f1f8e9
    style Constraints fill:#ffe0b2
    style Meta fill:#f3e5f5
```

---

## 8. Permission Hierarchy

```mermaid
graph TD
    Root["Root Access"]
    
    Root --> Admin["Admin Role"]
    Root --> Manager["Manager Role"]
    Root --> User["User Role"]
    
    Admin --> AdminPerms["<b>Admin Permissions:</b><br/>✓ Manage All Projects<br/>✓ Create Projects<br/>✓ Delete Projects<br/>✓ Assign Tasks<br/>✓ Manage Users<br/>✓ View All Files<br/>✓ View Activity Logs<br/>✓ Export Reports"]
    
    Manager --> ManagerPerms["<b>Manager Permissions:</b><br/>✓ Create Projects<br/>✓ Edit Own Projects<br/>✓ Assign Tasks<br/>✓ View Team Tasks<br/>✓ Upload Files<br/>✓ View Dashboard<br/>✗ Manage Users<br/>✗ Delete Projects"]
    
    User --> UserPerms["<b>User Permissions:</b><br/>✓ View Assigned Tasks<br/>✓ Update Task Status<br/>✓ Upload Files<br/>✓ Download Files<br/>✓ Preview Files<br/>✓ View Dashboard<br/>✗ Create Projects<br/>✗ Delete Tasks<br/>✗ Manage Users"]
    
    style Root fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Admin fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    style Manager fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style User fill:#c8e6c9,stroke:#1b5e20,stroke-width:2px
```

---

## 9. Data Flow Between Entities

```mermaid
graph LR
    subgraph Input["Input Layer"]
        Form1["Create Project Form"]
        Form2["Create Task Form"]
        Form3["Upload File Form"]
    end
    
    subgraph Processing["Processing Layer"]
        Validate1["Validate Input"]
        Process1["Process Data"]
        Generate1["Generate IDs"]
    end
    
    subgraph Storage["Storage Layer"]
        DB1["projects table"]
        DB2["tasks table"]
        DB3["files table"]
        FS["File Storage"]
    end
    
    subgraph Retrieval["Retrieval Layer"]
        Query1["Query Projects"]
        Query2["Query Tasks"]
        Query3["Query Files"]
    end
    
    subgraph Output["Output Layer"]
        View1["Project List View"]
        View2["Task List View"]
        View3["File Gallery View"]
    end
    
    Form1 --> Validate1
    Form2 --> Validate1
    Form3 --> Validate1
    
    Validate1 --> Process1
    Process1 --> Generate1
    
    Generate1 --> DB1
    Generate1 --> DB2
    Generate1 --> DB3
    Generate1 --> FS
    
    DB1 --> Query1
    DB2 --> Query2
    DB3 --> Query3
    
    Query1 --> View1
    Query2 --> View2
    Query3 --> View3
    
    style Input fill:#e1f5ff
    style Processing fill:#fff9c4
    style Storage fill:#f3e5f5
    style Retrieval fill:#f1f8e9
    style Output fill:#c8e6c9
```

---

## 10. System Integration Points

```mermaid
graph TB
    App["Project Management Dashboard<br/>(Laravel Application)"]
    
    App -->|User Management| Auth["Authentication System<br/>(Laravel Auth)"]
    App -->|Store Data| DB["MySQL Database<br/>(User, Project, Task, File records)"]
    App -->|Store Files| Storage["File Storage<br/>(Local Filesystem)"]
    App -->|Send Notifications| Telegram["Telegram Bot API<br/>(Deadline Reminders)"]
    App -->|Log Activities| Logs["Activity Logs<br/>(Database & Files)"]
    App -->|Schedule Tasks| Scheduler["Task Scheduler<br/>(Laravel Scheduler)"]
    
    Auth -->|Verify| Credentials["User Credentials<br/>(Hashed Passwords)"]
    
    DB -->|Contains| Tables["<b>Core Tables:</b><br/>users, roles, permissions<br/>projects, tasks, files<br/>activity_logs"]
    
    Storage -->|Stores| Files["<b>File Types:</b><br/>PDF, DOCX, XLSX, TXT<br/>Images (JPG, PNG, GIF)"]
    
    Telegram -->|Sends to| Users["Telegram Users<br/>(With Bot Token)"]
    
    Scheduler -->|Triggers| Commands["<b>Cron Jobs:</b><br/>SendTaskReminders<br/>CleanupOldFiles<br/>GenerateReports"]
    
    style App fill:#fff9c4,stroke:#f57f17,stroke-width:3px
    style Auth fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style DB fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style Storage fill:#f1f8e9,stroke:#1b5e20,stroke-width:2px
    style Telegram fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    style Logs fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    style Scheduler fill:#e0f2f1,stroke:#004d40,stroke-width:2px
```

---

## Diagram Symbols Reference

```
┌──────────────────────────────────────┐
│         Entity/Table                 │  = Database table or entity
├──────────────────────────────────────┤
│ PK  = Primary Key                    │
│ FK  = Foreign Key                    │
│ UK  = Unique Key                     │
│ ST  = Soft Delete (timestamp)        │
└──────────────────────────────────────┘

    ──────────→
       1:N     = One-to-Many relationship

    ←─────────→
       N:N     = Many-to-Many relationship

    ──────────
       1:1     = One-to-One relationship

    ─ · ─ · ─→
    (include)  = Includes or extends
```

---

## Quick Reference Tables

### Status Enumerations

| Entity | Status Values | Meaning |
|--------|---------------|---------|
| Project | active, on_hold, completed, cancelled | Project lifecycle states |
| Task | pending, in_progress, completed, overdue, blocked | Task progress states |
| User | active, inactive, suspended | User account states |
| File | (none) | Files are soft deleted only |

### Priority Levels

| Priority | Level | Response Time |
|----------|-------|----------------|
| Low | 3 | Can wait weeks |
| Medium | 2 | Should be done soon |
| High | 1 | Urgent |
| Critical | 0 | Immediate attention |

### File Type Categories

| Category | Types | Max Size |
|----------|-------|----------|
| Documents | PDF, DOCX, XLSX, TXT | 10 MB |
| Images | JPG, JPEG, PNG, GIF | 10 MB |
| Archives | ZIP, RAR | 10 MB |
| Total File Size | All types | 10 MB per file |

---

**[← Back to System Flow](./SYSTEM-FLOW.md)**

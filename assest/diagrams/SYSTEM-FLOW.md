# System Flow Diagrams - Project Management Dashboard

Visual representations of system processes and user interactions.

---

## 1. User Authentication Flow

```mermaid
flowchart TD
    Start([User Visits App]) --> CheckAuth{Authenticated?}
    CheckAuth -->|No| LoginPage[Display Login Page]
    LoginPage --> Submit[User Submits Credentials]
    Submit --> Validate{Valid Credentials?}
    Validate -->|No| Error[Show Error Message]
    Error --> LoginPage
    Validate -->|Yes| CreateSession[Create Session]
    CreateSession --> SetCookie[Set Session Cookie]
    SetCookie --> LogActivity[Log Login Activity]
    LogActivity --> Redirect[Redirect to Dashboard]
    Redirect --> Dashboard[Dashboard Page]
    
    CheckAuth -->|Yes| Dashboard
    Dashboard --> End([User Session Active])
    
    style Start fill:#e1f5ff
    style End fill:#c8e6c9
    style Error fill:#ffcdd2
    style CreateSession fill:#fff9c4
```

---

## 2. Project Management Flow

```mermaid
flowchart TD
    Admin([Admin User]) --> Dashboard[Dashboard]
    Dashboard --> Menu{Select Action}
    
    Menu -->|View Projects| List[List All Projects]
    List --> Filter{Filter By Status?}
    Filter -->|Yes| Filtered[Show Filtered Projects]
    Filter -->|No| Display[Display All Projects]
    Filtered --> Display
    Display --> SelectProject[Select Project]
    SelectProject --> ViewDetails[View Project Details]
    ViewDetails --> Options{Choose Action}
    
    Options -->|Edit| EditForm[Show Edit Form]
    EditForm --> Validate1{Valid Input?}
    Validate1 -->|No| Error1[Show Errors]
    Error1 --> EditForm
    Validate1 -->|Yes| UpdateDB1[Update Database]
    UpdateDB1 --> Log1[Log Activity]
    Log1 --> Success1[Show Success Message]
    Success1 --> Display
    
    Options -->|View Tasks| TaskList[List Project Tasks]
    TaskList --> Display
    
    Menu -->|Create Project| CreateForm[Show Create Form]
    CreateForm --> Validate2{Valid Input?}
    Validate2 -->|No| Error2[Show Validation Errors]
    Error2 --> CreateForm
    Validate2 -->|Yes| SaveDB[Save to Database]
    SaveDB --> LogCreate[Log Project Creation]
    LogCreate --> SuccessMsg[Show Success Message]
    SuccessMsg --> Display
    
    Menu -->|Delete Project| Confirm{Confirm Delete?}
    Confirm -->|Cancel| Display
    Confirm -->|Yes| Delete[Delete Project]
    Delete --> LogDelete[Log Deletion]
    LogDelete --> RefreshList[Refresh Project List]
    RefreshList --> Display
    
    style Admin fill:#e1f5ff
    style Dashboard fill:#fff9c4
    style Menu fill:#f3e5f5
    style Success1 fill:#c8e6c9
    style SuccessMsg fill:#c8e6c9
    style Error1 fill:#ffcdd2
    style Error2 fill:#ffcdd2
```

---

## 3. Task Assignment and Notification Flow

```mermaid
flowchart TD
    Admin[Admin Creates Task] --> CreateTask[Create Task Form]
    CreateTask --> AssignUser[Assign to User]
    AssignUser --> SetDeadline[Set Deadline]
    SetDeadline --> ValidateTask{All Fields Valid?}
    
    ValidateTask -->|No| ShowError[Show Error]
    ShowError --> CreateTask
    
    ValidateTask -->|Yes| SaveTask[Save Task to Database]
    SaveTask --> LogActivity1[Log Task Creation]
    LogActivity1 --> GetTelegram[Get User Telegram ID]
    GetTelegram --> SendNotif[Send Telegram Notification]
    SendNotif --> Success[Task Created Successfully]
    
    Success --> UpdateStatus[Task Status: Pending]
    UpdateStatus --> Dashboard[Return to Dashboard]
    
    Dashboard --> Scheduler[Scheduler Checks Tasks Daily]
    Scheduler --> CheckDeadline{Task Deadline = Today + 1?}
    
    CheckDeadline -->|No| Skip[Skip Task]
    Skip --> Scheduler
    
    CheckDeadline -->|Yes| BuildMessage[Build Reminder Message]
    BuildMessage --> SendReminder[Send Reminder via Telegram]
    SendReminder --> LogReminder[Log Reminder Sent]
    LogReminder --> Scheduler
    
    style Admin fill:#e1f5ff
    style SendNotif fill:#fff9c4
    style SendReminder fill:#fff9c4
    style Success fill:#c8e6c9
    style ShowError fill:#ffcdd2
```

---

## 4. File Upload and Management Flow

```mermaid
flowchart TD
    User[User Selects File] --> ChooseFile[Click Upload Button]
    ChooseFile --> FileDialog[File Dialog Opens]
    FileDialog --> SelectFile[Select File from Computer]
    SelectFile --> Submit[Submit Form]
    
    Submit --> Receive[Server Receives File]
    Receive --> ValidateType{Check File Type}
    
    ValidateType -->|Invalid| TypeError[Reject: Invalid Type]
    TypeError --> ShowError1[Show Error Message]
    ShowError1 --> ChooseFile
    
    ValidateType -->|Valid| ValidateSize{Check File Size}
    
    ValidateSize -->|Too Large| SizeError[Reject: File Too Large]
    SizeError --> ShowError2[Show Error Message]
    ShowError2 --> ChooseFile
    
    ValidateSize -->|OK| GenerateName[Generate Unique Filename]
    GenerateName --> StoreFile[Store File to Storage]
    StoreFile --> SaveMetadata[Save File Metadata to DB]
    SaveMetadata --> CreateRecord[Create File Record]
    CreateRecord --> LogActivity2[Log File Upload]
    LogActivity2 --> Success2[Upload Successful]
    
    Success2 --> Options{User Action}
    
    Options -->|View| Viewer[Display File Preview]
    Viewer --> Options
    
    Options -->|Download| Incr[Increment Download Count]
    Incr --> UpdateTime[Update Last Downloaded]
    UpdateTime --> ServeFile[Serve File]
    ServeFile --> Options
    
    Options -->|Delete| ConfirmDel{Confirm Delete?}
    ConfirmDel -->|No| Options
    ConfirmDel -->|Yes| SoftDelete[Soft Delete Record]
    SoftDelete --> LogDelete2[Log Deletion]
    LogDelete2 --> Success3[File Deleted]
    Success3 --> FileList[Return to File List]
    
    style User fill:#e1f5ff
    style Success2 fill:#c8e6c9
    style TypeError fill:#ffcdd2
    style SizeError fill:#ffcdd2
    style ShowError1 fill:#ffcdd2
    style ShowError2 fill:#ffcdd2
```

---

## 5. Dashboard Analytics Flow

```mermaid
flowchart TD
    User[User Accesses Dashboard] --> LoadPage[Load Dashboard Page]
    LoadPage --> FetchStats[Fetch Statistics]
    
    FetchStats --> Query1[Query: Total Projects]
    FetchStats --> Query2[Query: Active Projects]
    FetchStats --> Query3[Query: Completed Tasks]
    FetchStats --> Query4[Query: Overdue Tasks]
    
    Query1 --> CountProj[Count All Projects]
    Query2 --> ActiveProj[Count Status = Active]
    Query3 --> CompTasks[Count Status = Completed]
    Query4 --> OverdueTasks[Count Deadline &lt; Today]
    
    CountProj --> Cache{Check Cache}
    ActiveProj --> Cache
    CompTasks --> Cache
    OverdueTasks --> Cache
    
    Cache -->|Cache Hit| UseCached[Use Cached Data]
    Cache -->|Cache Miss| FetchDB[Fetch from Database]
    
    FetchDB --> StoreCache[Store in Cache]
    StoreCache --> DisplayData[Display Statistics]
    UseCached --> DisplayData
    
    DisplayData --> Charts[Display Charts & Graphs]
    Charts --> UserView[User Views Dashboard]
    
    UserView --> Actions{User Clicks}
    Actions -->|View Details| Details[Show Detailed View]
    Actions -->|Export| Export[Generate Export]
    Actions -->|Refresh| FetchStats
    
    Details --> Dashboard([Return to Dashboard])
    Export --> File[Download File]
    File --> Dashboard
    
    style User fill:#e1f5ff
    style DisplayData fill:#fff9c4
    style Charts fill:#fff9c4
    style UserView fill:#c8e6c9
```

---

## 6. Role-Based Access Control Flow

```mermaid
flowchart TD
    User[User Makes Request] --> CheckSession{Session Exists?}
    
    CheckSession -->|No| Redirect1[Redirect to Login]
    Redirect1 --> LoginPage([Login Page])
    
    CheckSession -->|Yes| FetchRole[Fetch User Role]
    FetchRole --> FetchPerms[Fetch Role Permissions]
    FetchPerms --> RequestResource[User Requests Resource]
    
    RequestResource --> CheckPermission{Has Required Permission?}
    
    CheckPermission -->|No| Forbidden[Return 403 Forbidden]
    Forbidden --> Error([Access Denied])
    
    CheckPermission -->|Yes| CheckResourceAccess{Can Access Resource?}
    
    CheckResourceAccess -->|Owner/Admin| AllowAccess[Allow Access]
    CheckResourceAccess -->|Not Owner| CheckShared{Is Shared?}
    
    CheckShared -->|Yes| AllowAccess
    CheckShared -->|No| Forbidden
    
    AllowAccess --> LoadResource[Load Resource Data]
    LoadResource --> RenderPage[Render View/Response]
    RenderPage --> Success([Request Complete])
    
    style User fill:#e1f5ff
    style AllowAccess fill:#c8e6c9
    style Success fill:#c8e6c9
    style Forbidden fill:#ffcdd2
    style Error fill:#ffcdd2
```

---

## 7. Activity Logging Flow

```mermaid
flowchart TD
    Action[User Performs Action] --> Controller[Controller Handles Request]
    Controller --> Middleware[Logging Middleware]
    
    Middleware --> CaptureRequest[Capture Request Data]
    CaptureRequest --> GetUser[Get Current User]
    GetUser --> GetIP[Get User IP Address]
    GetIP --> GetUA[Get User Agent]
    GetUA --> PerformAction[Execute Business Logic]
    
    PerformAction --> Success{Action Successful?}
    
    Success -->|No| LogError[Log Error Details]
    LogError --> SaveLog1[Save Error Log]
    SaveLog1 --> ReturnError([Return Error])
    
    Success -->|Yes| DetectChanges[Detect Data Changes]
    DetectChanges --> CompareValues[Compare Before/After Values]
    CompareValues --> BuildChanges[Build Changes JSON]
    BuildChanges --> LogSuccess[Log Action Details]
    
    LogSuccess --> SaveLog2[Save to activity_logs Table]
    SaveLog2 --> CheckNotify{Notify Admin?}
    
    CheckNotify -->|Yes| SendEmail[Send Admin Email]
    CheckNotify -->|No| Skip[Continue]
    
    SendEmail --> Skip
    Skip --> Return([Return Response])
    
    style Action fill:#e1f5ff
    style PerformAction fill:#fff9c4
    style LogSuccess fill:#fff9c4
    style SaveLog2 fill:#f3e5f5
    style Return fill:#c8e6c9
    style ReturnError fill:#ffcdd2
```

---

## 8. Telegram Bot Notification System

```mermaid
flowchart TD
    Event[System Event Triggered] --> Identify{Event Type}
    
    Identify -->|Task Assignment| TaskEvent[Get Task Details]
    Identify -->|Deadline Reminder| DeadlineEvent[Get Task Info]
    Identify -->|Project Status| ProjectEvent[Get Project Info]
    
    TaskEvent --> GetUser1[Get Assigned User]
    DeadlineEvent --> GetUser2[Get Assigned User]
    ProjectEvent --> GetUser3[Get Project Owner]
    
    GetUser1 --> CheckTelegram{Telegram ID Set?}
    GetUser2 --> CheckTelegram
    GetUser3 --> CheckTelegram
    
    CheckTelegram -->|No| LogMissing[Log Missing Telegram ID]
    LogMissing --> Skip1[Skip Notification]
    
    CheckTelegram -->|Yes| BuildMessage[Build Message Content]
    BuildMessage --> Format[Format with Markdown]
    Format --> CallAPI[Call Telegram Bot API]
    
    CallAPI --> SendRequest[Send HTTP POST Request]
    SendRequest --> APIResponse{API Response OK?}
    
    APIResponse -->|Success| LogSuccess[Log Success]
    LogSuccess --> RecordNotif[Record Notification Sent]
    RecordNotif --> Complete([Notification Sent])
    
    APIResponse -->|Failed| Retry{Retry Count &lt; 3?}
    Retry -->|Yes| Wait[Wait 5 Seconds]
    Wait --> SendRequest
    Retry -->|No| LogFailed[Log Failed]
    LogFailed --> Alert[Alert Administrator]
    Alert --> Failed([Notification Failed])
    
    style Event fill:#e1f5ff
    style BuildMessage fill:#fff9c4
    style CallAPI fill:#fff9c4
    style Complete fill:#c8e6c9
    style Failed fill:#ffcdd2
    style Alert fill:#ffcdd2
```

---

## 9. Task Status Update Flow

```mermaid
flowchart TD
    User[User Opens Task] --> ViewTask[View Task Details]
    ViewTask --> CurrentStatus[Display Current Status]
    CurrentStatus --> UpdateForm[Show Update Form]
    
    UpdateForm --> SelectStatus[Select New Status]
    SelectStatus --> AddComment[Add Optional Comment]
    AddComment --> UpdateProgress[Update Progress %]
    UpdateProgress --> Submit[Submit Changes]
    
    Submit --> Validate{All Valid?}
    Validate -->|No| ShowError[Show Validation Errors]
    ShowError --> UpdateForm
    
    Validate -->|Yes| CheckPermission{Can Update?}
    CheckPermission -->|No| DenyError[Permission Denied]
    DenyError --> ViewTask
    
    CheckPermission -->|Yes| UpdateDB[Update Task Status]
    UpdateDB --> RecordChange[Record Changes in JSON]
    RecordChange --> LogActivity[Log Activity]
    LogActivity --> CheckNotification{Notify Others?}
    
    CheckNotification -->|Yes| NotifyAssignee[Notify Assignee]
    CheckNotification -->|Yes| NotifyProjectOwner[Notify Project Owner]
    
    NotifyAssignee --> SendTelegram1[Send via Telegram]
    NotifyProjectOwner --> SendTelegram2[Send via Telegram]
    
    SendTelegram1 --> Success[Update Complete]
    SendTelegram2 --> Success
    
    Success --> Refresh[Refresh Task View]
    Refresh --> Display[Show Updated Status]
    Display --> ViewTask
    
    style User fill:#e1f5ff
    style Success fill:#c8e6c9
    style Display fill:#c8e6c9
    style DenyError fill:#ffcdd2
    style ShowError fill:#ffcdd2
```

---

## 10. Scheduled Task Reminder Process

```mermaid
flowchart TD
    Scheduler[Scheduler Runs Daily at 8 AM] --> Start[Initialize Reminder Process]
    Start --> Query[Query Database for Reminders]
    
    Query --> GetTasks[Get Tasks with Deadline = Tomorrow]
    GetTasks --> Filter{Tasks Found?}
    
    Filter -->|No| LogEmpty[Log No Reminders Needed]
    LogEmpty --> End1([Process Complete])
    
    Filter -->|Yes| Loop[Loop Through Tasks]
    Loop --> GetTask[Get Task Details]
    GetTask --> GetUser[Get Assigned User]
    GetUser --> CheckTelegram[Check Telegram ID]
    
    CheckTelegram -->|Missing| LogSkip[Log Skipped]
    LogSkip --> NextTask[Move to Next Task]
    
    CheckTelegram -->|Present| BuildMsg[Build Reminder Message]
    BuildMsg --> Include1["Include: Task Name"]
    Include1 --> Include2["Include: Project Name"]
    Include2 --> Include3["Include: Deadline Date"]
    Include3 --> Include4["Include: Priority Level"]
    Include4 --> SendTelegram[Send via Telegram API]
    
    SendTelegram --> RecordEvent[Record Reminder Sent]
    RecordEvent --> LogSuccess[Log Success]
    LogSuccess --> NextTask
    
    NextTask --> MoreTasks{More Tasks?}
    MoreTasks -->|Yes| GetTask
    MoreTasks -->|No| Summary[Generate Summary Report]
    Summary --> LogSummary[Log Total Reminders Sent]
    LogSummary --> End2([Process Complete])
    
    style Scheduler fill:#e1f5ff
    style SendTelegram fill:#fff9c4
    style LogSuccess fill:#fff9c4
    style End2 fill:#c8e6c9
    style LogEmpty fill:#c8e6c9
```

---

## Legend

```
┌─────────────────────┐
│   Start/End Point   │ = Start or end of a process
└─────────────────────┘

    ┌─────────┐
    │ Process │ = Action or operation
    └─────────┘

    ╱─────────╲
   │ Decision │ = Conditional branch
    ╲─────────╱

    ─────────→ = Flow direction
```

Color Legend:
- **Blue**: User action or input
- **Yellow**: Processing or API call
- **Green**: Success completion
- **Red**: Error or denial
- **Purple**: Data storage

---

**[← Back to Database Design](../DATABASE.md)**

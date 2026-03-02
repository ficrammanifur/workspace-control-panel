# Database Design - Project Management Dashboard

Complete database schema, relationships, and design decisions.

**Navigation:** [ARCHITECTURE.md ←](./ARCHITECTURE.md) | [README.md →](./README.md)

---

## Table of Contents

1. [Database Overview](#database-overview)
2. [ERD (Entity Relationship Diagram)](#erd-entity-relationship-diagram)
3. [Table Specifications](#table-specifications)
4. [Relationships](#relationships)
5. [Indexing Strategy](#indexing-strategy)
6. [Migration Examples](#migration-examples)
7. [Data Integrity](#data-integrity)

---

## Database Overview

### Technology Stack

- **Primary**: MySQL 8.0+ or PostgreSQL 14+
- **ORM**: Laravel Eloquent
- **Migrations**: Laravel Migration System
- **Seeders**: Database Seeders for sample data

### Database Statistics

- **Total Tables**: 7 core tables
- **Total Relationships**: 15+ relationships
- **Indexes**: 25+ strategic indexes
- **Sample Data**: ~2,000 records when seeded

### Key Design Decisions

1. **Soft Deletes**: Uses `deleted_at` timestamp for data recovery
2. **Timestamps**: All tables have `created_at` and `updated_at`
3. **UUIDs**: Optional for high-scale applications
4. **Status Enums**: Predefined status values for consistency
5. **Normalization**: 3NF (Third Normal Form) applied

---

## ERD (Entity Relationship Diagram)

### Visual Representation

```
┌─────────────────────┐
│       USERS         │
├─────────────────────┤
│ id (PK)             │
│ name                │
│ email (UNIQUE)      │
│ password            │
│ role_id (FK)        │
│ telegram_chat_id    │
│ status              │
│ created_at          │
│ updated_at          │
└──────────┬──────────┘
           │
           │ 1:N
           ├─────────────────────┐
           │                     │
           │                     ▼
           │        ┌────────────────────┐
           │        │     PROJECTS       │
           │        ├────────────────────┤
           │        │ id (PK)            │
           │        │ name               │
           │        │ client_name        │
           │        │ description        │
           │        │ deadline           │
           │        │ status             │
           │        │ assigned_to (FK)   │
           │        │ created_by (FK)    │
           │        │ created_at         │
           │        │ updated_at         │
           │        │ deleted_at (ST)    │
           │        └─────┬──────────────┘
           │              │
           │              │ 1:N
           │              │
           │              ▼
           │    ┌──────────────────┐
           │    │      TASKS       │
           │    ├──────────────────┤
           │    │ id (PK)          │
           │    │ project_id (FK)  │
           │    │ name             │
           │    │ priority         │
           │    │ deadline         │
           │    │ status           │
           │    │ assigned_to (FK) │
           │    │ created_at       │
           │    │ updated_at       │
           │    │ deleted_at (ST)  │
           │    └──────┬───────────┘
           │           │
           │           │ 1:N
           │           │
           │           ▼
           │    ┌──────────────────┐
           │    │      FILES       │
           │    ├──────────────────┤
           │    │ id (PK)          │
           │    │ task_id (FK)     │
           │    │ project_id (FK)  │
           │    │ uploaded_by (FK) │
           │    │ filename         │
           │    │ file_type        │
           │    │ file_size        │
           │    │ file_path        │
           │    │ created_at       │
           │    │ deleted_at (ST)  │
           │    └──────────────────┘
           │
           │
           └─────────────────────┐
                                 ▼
                    ┌──────────────────────┐
                    │   ACTIVITY_LOGS      │
                    ├──────────────────────┤
                    │ id (PK)              │
                    │ user_id (FK)         │
                    │ action               │
                    │ model_type           │
                    │ model_id             │
                    │ ip_address           │
                    │ user_agent           │
                    │ changes (JSON)       │
                    │ created_at           │
                    └──────────────────────┘

┌─────────────────────┐
│       ROLES         │
├─────────────────────┤
│ id (PK)             │
│ name                │
│ description         │
│ created_at          │
│ updated_at          │
└──────────┬──────────┘
           │
           │ 1:N
           │
           ▼
┌─────────────────────────────────┐
│     ROLE_PERMISSIONS            │
├─────────────────────────────────┤
│ role_id (FK) + permission_id    │
│ (FK) - Composite Primary Key    │
└─────────────────────────────────┘
           ▲
           │ N:N
           │
           │
┌─────────────────────┐
│   PERMISSIONS       │
├─────────────────────┤
│ id (PK)             │
│ name                │
│ description         │
│ resource            │
│ action              │
│ created_at          │
│ updated_at          │
└─────────────────────┘

Legend:
PK = Primary Key
FK = Foreign Key
ST = Soft Delete (timestamps)
1:N = One-to-Many
N:N = Many-to-Many
```

### Mermaid ERD

```mermaid
erDiagram
    USERS ||--o{ PROJECTS : creates
    USERS ||--o{ TASKS : assigned
    USERS ||--o{ FILES : uploads
    USERS ||--o{ ACTIVITY_LOGS : performs
    USERS ||--|| ROLES : has
    ROLES ||--o{ PERMISSIONS : has
    PROJECTS ||--o{ TASKS : contains
    PROJECTS ||--o{ FILES : has
    TASKS ||--o{ FILES : has
    TASKS ||--o{ ACTIVITY_LOGS : triggers

    USERS {
        int id PK
        string name
        string email UK
        string password
        int role_id FK
        string telegram_chat_id
        string status
        timestamp created_at
        timestamp updated_at
    }

    PROJECTS {
        int id PK
        string name
        string client_name
        text description
        date deadline
        string status
        int assigned_to FK
        int created_by FK
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at ST
    }

    TASKS {
        int id PK
        int project_id FK
        string name
        string priority
        date deadline
        string status
        int assigned_to FK
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at ST
    }

    FILES {
        int id PK
        int task_id FK
        int project_id FK
        int uploaded_by FK
        string filename
        string file_type
        bigint file_size
        string file_path
        timestamp created_at
        timestamp deleted_at ST
    }

    ACTIVITY_LOGS {
        int id PK
        int user_id FK
        string action
        string model_type
        int model_id
        string ip_address
        text user_agent
        json changes
        timestamp created_at
    }

    ROLES {
        int id PK
        string name UK
        text description
        timestamp created_at
        timestamp updated_at
    }

    PERMISSIONS {
        int id PK
        string name UK
        text description
        string resource
        string action
        timestamp created_at
        timestamp updated_at
    }
```

---

## Table Specifications

### 1. USERS Table

**Purpose:** Store user accounts and authentication data

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    email_verified_at TIMESTAMP NULL,
    password VARCHAR(255) NOT NULL,
    role_id BIGINT UNSIGNED NOT NULL,
    telegram_chat_id VARCHAR(255) NULL,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    last_login_at TIMESTAMP NULL,
    remember_token VARCHAR(100) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    FOREIGN KEY (role_id) REFERENCES roles(id),
    INDEX idx_email (email),
    INDEX idx_role_id (role_id),
    INDEX idx_status (status)
);
```

**Fields:**
- `id`: Unique identifier (auto-incrementing)
- `name`: User's full name
- `email`: Unique email address (login credential)
- `email_verified_at`: Email verification timestamp
- `password`: Hashed password (bcrypt)
- `role_id`: Reference to user's role
- `telegram_chat_id`: User's Telegram ID for notifications
- `status`: Account status (active/inactive/suspended)
- `last_login_at`: Last successful login timestamp
- `remember_token`: Session remember token

**Constraints:**
- Email must be unique
- Role ID must reference valid role
- Status must be one of predefined values

---

### 2. ROLES Table

**Purpose:** Define user roles

```sql
CREATE TABLE roles (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    INDEX idx_name (name)
);
```

**Sample Data:**
```sql
INSERT INTO roles (name, description) VALUES
('Admin', 'Full system access'),
('Manager', 'Project and task management'),
('User', 'Limited task access');
```

---

### 3. PERMISSIONS Table

**Purpose:** Define granular permissions

```sql
CREATE TABLE permissions (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE,
    description TEXT NULL,
    resource VARCHAR(100) NOT NULL,
    action VARCHAR(100) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    UNIQUE KEY unique_permission (resource, action),
    INDEX idx_resource (resource)
);
```

**Sample Permissions:**
```sql
INSERT INTO permissions (name, description, resource, action) VALUES
('view_projects', 'Can view projects', 'projects', 'view'),
('create_projects', 'Can create projects', 'projects', 'create'),
('edit_projects', 'Can edit projects', 'projects', 'edit'),
('delete_projects', 'Can delete projects', 'projects', 'delete'),
('view_tasks', 'Can view tasks', 'tasks', 'view'),
('create_tasks', 'Can create tasks', 'tasks', 'create'),
('manage_users', 'Can manage users', 'users', 'manage'),
('manage_files', 'Can manage files', 'files', 'manage');
```

---

### 4. ROLE_PERMISSIONS Table

**Purpose:** Pivot table for role-permission relationships

```sql
CREATE TABLE role_permissions (
    role_id BIGINT UNSIGNED NOT NULL,
    permission_id BIGINT UNSIGNED NOT NULL,
    created_at TIMESTAMP NULL,
    
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE,
    INDEX idx_permission_id (permission_id)
);
```

**Admin Role Permissions:**
```sql
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.name = 'Admin'; -- Admin has all permissions
```

---

### 5. PROJECTS Table

**Purpose:** Store project information

```sql
CREATE TABLE projects (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    client_name VARCHAR(255) NOT NULL,
    description TEXT NULL,
    deadline DATE NOT NULL,
    status ENUM('active', 'on_hold', 'completed', 'cancelled') DEFAULT 'active',
    assigned_to BIGINT UNSIGNED NOT NULL,
    created_by BIGINT UNSIGNED NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    deleted_at TIMESTAMP NULL,
    
    FOREIGN KEY (assigned_to) REFERENCES users(id),
    FOREIGN KEY (created_by) REFERENCES users(id),
    INDEX idx_status (status),
    INDEX idx_deadline (deadline),
    INDEX idx_assigned_to (assigned_to),
    INDEX idx_created_by (created_by),
    INDEX idx_deleted_at (deleted_at)
);
```

**Fields:**
- `id`: Unique project identifier
- `name`: Project name
- `client_name`: Client name
- `description`: Project description
- `deadline`: Project deadline date
- `status`: Project status
- `assigned_to`: User responsible for project
- `created_by`: User who created the project
- `deleted_at`: Soft delete timestamp

**Status Values:**
- `active`: Project in progress
- `on_hold`: Project paused
- `completed`: Project finished
- `cancelled`: Project cancelled

---

### 6. TASKS Table

**Purpose:** Store task information

```sql
CREATE TABLE tasks (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    project_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT NULL,
    priority ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
    deadline DATE NOT NULL,
    status ENUM('pending', 'in_progress', 'completed', 'overdue', 'blocked') DEFAULT 'pending',
    assigned_to BIGINT UNSIGNED NOT NULL,
    progress_percentage INT DEFAULT 0,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    deleted_at TIMESTAMP NULL,
    
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_to) REFERENCES users(id),
    INDEX idx_project_id (project_id),
    INDEX idx_status (status),
    INDEX idx_priority (priority),
    INDEX idx_deadline (deadline),
    INDEX idx_assigned_to (assigned_to),
    INDEX idx_deleted_at (deleted_at),
    INDEX idx_project_status (project_id, status)
);
```

**Fields:**
- `id`: Unique task identifier
- `project_id`: Reference to parent project
- `name`: Task name
- `description`: Detailed task description
- `priority`: Task priority level
- `deadline`: Task deadline
- `status`: Current task status
- `assigned_to`: User assigned to task
- `progress_percentage`: Task completion percentage (0-100)

**Priority Levels:**
- `low`: Can be scheduled later
- `medium`: Regular priority
- `high`: Should be done soon
- `critical`: Must be done ASAP

**Status Values:**
- `pending`: Not started
- `in_progress`: Currently being worked on
- `completed`: Task finished
- `overdue`: Deadline passed
- `blocked`: Cannot proceed

---

### 7. FILES Table

**Purpose:** Store file metadata and references

```sql
CREATE TABLE files (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    task_id BIGINT UNSIGNED NULL,
    project_id BIGINT UNSIGNED NULL,
    uploaded_by BIGINT UNSIGNED NOT NULL,
    filename VARCHAR(255) NOT NULL,
    original_filename VARCHAR(255) NOT NULL,
    file_type VARCHAR(50) NOT NULL,
    file_size BIGINT NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    download_count INT DEFAULT 0,
    last_downloaded_at TIMESTAMP NULL,
    created_at TIMESTAMP NULL,
    deleted_at TIMESTAMP NULL,
    
    FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE SET NULL,
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (uploaded_by) REFERENCES users(id),
    INDEX idx_task_id (task_id),
    INDEX idx_project_id (project_id),
    INDEX idx_uploaded_by (uploaded_by),
    INDEX idx_file_type (file_type),
    INDEX idx_deleted_at (deleted_at),
    INDEX idx_created_at (created_at)
);
```

**Fields:**
- `id`: Unique file identifier
- `task_id`: Reference to associated task (nullable)
- `project_id`: Reference to associated project
- `uploaded_by`: User who uploaded the file
- `filename`: Stored filename (hashed/unique)
- `original_filename`: Original uploaded filename
- `file_type`: File extension (pdf, docx, xlsx, etc.)
- `file_size`: File size in bytes
- `file_path`: Path to file on disk
- `mime_type`: MIME type of file
- `download_count`: Number of downloads
- `last_downloaded_at`: Last download timestamp

**Allowed File Types:**
- Documents: PDF, DOCX, XLSX, TXT
- Images: JPG, JPEG, PNG, GIF
- Others: ZIP, RAR (configurable)

---

### 8. ACTIVITY_LOGS Table

**Purpose:** Audit trail for all user actions

```sql
CREATE TABLE activity_logs (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT UNSIGNED NOT NULL,
    action VARCHAR(100) NOT NULL,
    model_type VARCHAR(100) NULL,
    model_id BIGINT UNSIGNED NULL,
    description TEXT NULL,
    changes JSON NULL,
    ip_address VARCHAR(45) NULL,
    user_agent TEXT NULL,
    created_at TIMESTAMP NULL,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_action (action),
    INDEX idx_model_type_id (model_type, model_id),
    INDEX idx_created_at (created_at)
);
```

**Fields:**
- `id`: Log entry identifier
- `user_id`: User who performed action
- `action`: Type of action (created, updated, deleted, etc.)
- `model_type`: Model class (Project, Task, File)
- `model_id`: ID of affected model
- `description`: Human-readable description
- `changes`: JSON object with before/after values
- `ip_address`: User's IP address
- `user_agent`: Browser user agent
- `created_at`: Timestamp of action

**Example Activity Log Entry:**
```json
{
    "user_id": 5,
    "action": "task_updated",
    "model_type": "Task",
    "model_id": 42,
    "description": "Updated task status from pending to in_progress",
    "changes": {
        "status": {
            "from": "pending",
            "to": "in_progress"
        },
        "progress_percentage": {
            "from": 0,
            "to": 25
        }
    },
    "ip_address": "192.168.1.100",
    "user_agent": "Mozilla/5.0...",
    "created_at": "2024-01-15 10:30:45"
}
```

---

## Relationships

### User Relationships

```
User
├─ hasMany(Project) - as project creator
├─ hasMany(Task) - assigned tasks
├─ hasMany(File) - uploaded files
├─ hasMany(ActivityLog)
├─ belongsTo(Role)
└─ hasManyThrough(Permission) - via Role
```

### Project Relationships

```
Project
├─ belongsTo(User, 'assigned_to')
├─ belongsTo(User, 'created_by')
├─ hasMany(Task)
├─ hasMany(File)
└─ hasMany(ActivityLog)
```

### Task Relationships

```
Task
├─ belongsTo(Project)
├─ belongsTo(User, 'assigned_to')
├─ hasMany(File)
└─ hasMany(ActivityLog)
```

### File Relationships

```
File
├─ belongsTo(Project)
├─ belongsTo(Task)
└─ belongsTo(User, 'uploaded_by')
```

---

## Indexing Strategy

### Primary Indexes (Query Performance)

| Table | Column | Type | Reason |
|-------|--------|------|--------|
| users | email | UNIQUE | Login queries |
| roles | name | UNIQUE | Role lookups |
| permissions | name | UNIQUE | Permission checks |
| permissions | resource, action | COMPOSITE | Permission matching |
| projects | status | INDEX | Filter active projects |
| projects | deadline | INDEX | Overdue checks |
| projects | assigned_to | INDEX | User projects |
| tasks | project_id | INDEX | Project tasks |
| tasks | status | INDEX | Filter task status |
| tasks | deadline | INDEX | Deadline reminders |
| tasks | assigned_to | INDEX | User tasks |
| files | file_type | INDEX | File filtering |
| files | uploaded_by | INDEX | User uploads |
| activity_logs | user_id | INDEX | User activity |
| activity_logs | action | INDEX | Action filtering |

### Query Optimization Examples

```sql
-- Fast: Uses index on projects.status
SELECT * FROM projects WHERE status = 'active';

-- Fast: Uses composite index on permissions(resource, action)
SELECT * FROM permissions WHERE resource = 'projects' AND action = 'create';

-- Fast: Uses index on tasks(project_id, status)
SELECT * FROM tasks WHERE project_id = 5 AND status = 'pending';

-- Slow: No index (avoid)
SELECT * FROM tasks WHERE description LIKE '%something%';

-- Better: Add FULLTEXT index if needed
ALTER TABLE tasks ADD FULLTEXT INDEX ft_description (description);
```

---

## Migration Examples

### Sample Migration File

```php
// database/migrations/2024_01_01_000000_create_projects_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('projects', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('client_name');
            $table->text('description')->nullable();
            $table->date('deadline');
            $table->enum('status', ['active', 'on_hold', 'completed', 'cancelled'])
                  ->default('active');
            $table->foreignId('assigned_to')
                  ->constrained('users');
            $table->foreignId('created_by')
                  ->constrained('users');
            $table->timestamps();
            $table->softDeletes();
            
            // Indexes
            $table->index('status');
            $table->index('deadline');
            $table->index(['assigned_to', 'status']);
        });
    }
    
    public function down(): void
    {
        Schema::dropIfExists('projects');
    }
};
```

### Running Migrations

```bash
# Run all pending migrations
php artisan migrate

# Run migrations with seeders
php artisan migrate --seed

# Rollback last batch
php artisan migrate:rollback

# Reset database
php artisan migrate:reset

# Refresh (rollback + migrate)
php artisan migrate:refresh

# Check migration status
php artisan migrate:status
```

---

## Data Integrity

### Referential Integrity

**Cascade Delete:**
- When project is deleted, all tasks are deleted
- When task is deleted, all files attached to it are deleted
- When role is deleted, all related permissions are removed

**Set Null on Delete:**
- When task is deleted, files' task_id becomes NULL
- Preserves file records while removing task reference

### Constraints

```sql
-- Foreign Key Constraints
ALTER TABLE projects 
ADD CONSTRAINT fk_projects_assigned_to 
FOREIGN KEY (assigned_to) REFERENCES users(id);

-- Check Constraints (MySQL 8.0+)
ALTER TABLE tasks 
ADD CONSTRAINT check_progress_percentage 
CHECK (progress_percentage >= 0 AND progress_percentage <= 100);

-- Unique Constraints
ALTER TABLE projects 
ADD CONSTRAINT unique_project_name_per_year 
UNIQUE (name, YEAR(created_at));
```

### Data Validation

**At Database Level:**
```sql
-- File size validation
ALTER TABLE files 
ADD CONSTRAINT check_file_size 
CHECK (file_size > 0 AND file_size <= 10485760); -- 10MB

-- Deadline validation
ALTER TABLE tasks 
ADD CONSTRAINT check_deadline 
CHECK (deadline >= DATE(NOW()));
```

**At Application Level (Laravel Validation):**
```php
$validated = $request->validate([
    'name' => 'required|string|max:255',
    'email' => 'required|email|unique:users',
    'deadline' => 'required|date|after:today',
    'priority' => 'required|in:low,medium,high,critical',
    'file' => 'required|file|mimes:pdf,docx|max:10240',
]);
```

### Transaction Example

```php
DB::transaction(function () {
    $project = Project::create($projectData);
    
    foreach ($tasksData as $taskData) {
        $project->tasks()->create($taskData);
    }
    
    ActivityLog::create([
        'action' => 'project_created',
        'model_type' => 'Project',
        'model_id' => $project->id,
    ]);
});
```

If any operation fails, all changes are rolled back.

---

## Performance Considerations

### Query Count Optimization

**Problem (N+1 Queries):**
```php
// This executes 101 queries (1 for projects + 100 for tasks)
$projects = Project::all(); // 1 query
foreach ($projects as $project) {
    $tasks = $project->tasks; // 100 queries
}
```

**Solution (Eager Loading):**
```php
// This executes only 2 queries
$projects = Project::with('tasks')->get();
foreach ($projects as $project) {
    $tasks = $project->tasks; // No additional queries
}
```

### Large Table Handling

For tables with millions of records:

```php
// Use chunking for processing
Task::chunk(100, function ($tasks) {
    foreach ($tasks as $task) {
        // Process task
    }
});

// Use pagination for display
$tasks = Task::paginate(50);
```

### Index Maintenance

```bash
# Check index usage
ANALYZE TABLE projects;

# Optimize table
OPTIMIZE TABLE projects;

# Check for unused indexes
SELECT * FROM mysql.innodb_trx;
```

---

## Backup Strategy

### Regular Backups

```bash
# MySQL backup
mysqldump -u root -p project_dashboard > backup_$(date +%Y%m%d_%H%M%S).sql

# PostgreSQL backup
pg_dump project_dashboard > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore from backup
mysql -u root -p project_dashboard < backup.sql
```

### Backup Scheduling

```bash
# Cron job for daily backup
0 2 * * * /usr/bin/mysqldump -u root -p project_dashboard > /backups/project_dashboard_$(date +\%Y\%m\%d).sql
```

---

**[← Back to ARCHITECTURE.md](./ARCHITECTURE.md) | [→ Next: README.md](./README.md)**

# Quick Reference Guide

Fast lookup for developers working with the Project Management Dashboard.

---

## 🚀 Start Here

```bash
# Installation (5 minutes)
git clone <repository>
cd project-management-dashboard
composer install
cp .env.example .env
php artisan key:generate
# Configure .env (database, telegram)
php artisan migrate --seed
php artisan serve
```

Visit: `http://localhost:8000`

**Demo Login:**
- Email: `admin@example.com`
- Password: `password`

---

## 📂 File Locations

| Component | Location |
|-----------|----------|
| Models | `app/Models/` |
| Controllers | `app/Http/Controllers/` |
| Services | `app/Services/` |
| Repositories | `app/Repositories/` |
| Migrations | `database/migrations/` |
| Seeders | `database/seeders/` |
| Views | `resources/views/` |
| Routes | `routes/web.php` |
| Config | `config/` |

---

## 🗄️ Database Commands

```bash
# Run all migrations
php artisan migrate

# Run with seeders
php artisan migrate --seed

# Rollback last batch
php artisan migrate:rollback

# Reset database
php artisan migrate:reset

# Check status
php artisan migrate:status

# Fresh migration
php artisan migrate:fresh --seed
```

---

## 💾 Models & Relationships

### User Model
```php
$user->role;              // User's role
$user->projects();        // Projects created
$user->tasks();           // Assigned tasks
$user->uploadedFiles();   // Uploaded files
$user->hasPermission('create_projects');
```

### Project Model
```php
$project->assignee;       // Assigned user
$project->creator;        // Created by user
$project->tasks();        // All tasks
$project->files();        // All files
$project->progress;       // Progress percentage
$project->isOverdue();    // Check if overdue
```

### Task Model
```php
$task->project;           // Parent project
$task->assignee;          // Assigned user
$task->files();           // Attachments
$task->isOverdue();       // Check if overdue
$task->updateProgress(50); // Update progress
```

---

## 🛣️ API Endpoints

### Projects
```
GET    /projects                 List all projects
POST   /projects                 Create project
GET    /projects/{id}            Show project
PUT    /projects/{id}            Update project
DELETE /projects/{id}            Delete project
```

### Tasks
```
GET    /tasks                    List tasks
POST   /tasks                    Create task
GET    /tasks/{id}               Show task
PUT    /tasks/{id}               Update task
PATCH  /tasks/{id}/status        Update status
DELETE /tasks/{id}               Delete task
```

### Files
```
POST   /files/upload             Upload file
GET    /files/{id}/download      Download file
DELETE /files/{id}               Delete file
```

### Dashboard
```
GET    /dashboard                Dashboard page
```

---

## 🔍 Query Examples

### Get Active Projects
```php
$projects = Project::active()->get();
```

### Get Overdue Tasks
```php
$tasks = Task::overdue()->with('assignee')->get();
```

### Get User's Projects
```php
$projects = Project::where('assigned_to', auth()->id())->get();
```

### Get Completed Tasks
```php
$tasks = Task::where('status', 'completed')->count();
```

### Get User with Permissions
```php
$user = User::with('role.permissions')->find($id);
```

---

## 🔐 Authorization

### Check Permission
```php
if (auth()->user()->hasPermission('create_projects')) {
    // Allow action
}
```

### Policy
```php
@can('create', App\Models\Project::class)
    <a href="{{ route('projects.create') }}">Create</a>
@endcan
```

### Middleware
```php
Route::post('/projects', [ProjectController::class, 'store'])
    ->middleware('auth');
```

---

## 📝 Migrations

### Create New Table
```bash
php artisan make:migration create_projects_table
```

### Add Column
```bash
php artisan make:migration add_status_to_projects_table
```

### Modify Column
```bash
php artisan make:migration modify_projects_table
```

---

## 🧪 Testing

### Run All Tests
```bash
php artisan test
```

### Run Specific Test
```bash
php artisan test tests/Feature/ProjectTest.php
```

### With Coverage
```bash
php artisan test --coverage
```

---

## 🔧 Artisan Commands

```bash
# Generate model with factory and migration
php artisan make:model Project -mf

# Generate controller with CRUD
php artisan make:controller ProjectController -r

# Generate request validation
php artisan make:request StoreProjectRequest

# Generate migration
php artisan make:migration create_projects_table

# Generate seeder
php artisan make:seeder ProjectSeeder

# Clear caches
php artisan cache:clear
php artisan config:clear
php artisan view:clear

# Interactive shell
php artisan tinker
```

---

## 📊 Table Structures

### Projects Table
```
id              - Primary Key
name            - Project name
client_name     - Client name
description     - Description
deadline        - Due date
status          - Status (active, on_hold, completed, cancelled)
assigned_to     - Foreign Key to users
created_by      - Foreign Key to users
created_at      - Created timestamp
updated_at      - Updated timestamp
deleted_at      - Soft delete timestamp
```

### Tasks Table
```
id              - Primary Key
project_id      - Foreign Key to projects
name            - Task name
description     - Description
priority        - Priority (low, medium, high, critical)
deadline        - Due date
status          - Status (pending, in_progress, completed, overdue, blocked)
assigned_to     - Foreign Key to users
progress_percentage - 0-100
created_at      - Created timestamp
updated_at      - Updated timestamp
deleted_at      - Soft delete timestamp
```

### Files Table
```
id              - Primary Key
task_id         - Foreign Key to tasks (nullable)
project_id      - Foreign Key to projects
uploaded_by     - Foreign Key to users
filename        - Stored filename
original_filename - Original name
file_type       - Extension (pdf, docx, etc)
file_size       - Size in bytes
file_path       - Storage path
mime_type       - MIME type
download_count  - Download counter
last_downloaded_at - Last download timestamp
created_at      - Created timestamp
deleted_at      - Soft delete timestamp
```

---

## 🔔 Telegram Notifications

### Setup
```bash
1. Find @BotFather on Telegram
2. Create new bot (get token)
3. Add to .env: TELEGRAM_BOT_TOKEN=xxx
4. Get your Chat ID from getUpdates
5. Add to .env: TELEGRAM_CHAT_ID=xxx
```

### Send Message
```php
use App\Services\TelegramNotificationService;

$service = app(TelegramNotificationService::class);
$service->notify($chatId, "Message text");
```

---

## 🛡️ Security Checklist

- [ ] Set `APP_DEBUG=false` in production
- [ ] Set `APP_ENV=production`
- [ ] Run `php artisan key:generate`
- [ ] Configure HTTPS
- [ ] Set proper file permissions
- [ ] Enable database backups
- [ ] Configure firewall rules
- [ ] Use environment variables
- [ ] Enable CSRF protection
- [ ] Implement rate limiting

---

## ⚡ Performance Tips

### Query Optimization
```php
// Bad - N+1 queries
$projects = Project::all();
foreach ($projects as $p) $p->tasks->count();

// Good - Eager loading
$projects = Project::with('tasks')->get();
```

### Caching
```php
$data = cache()->remember('key', 3600, function () {
    return expensive_operation();
});
```

### Pagination
```php
$items = Model::paginate(15);
```

### Indexes
```php
// In migration
$table->index('status');
$table->index(['project_id', 'status']);
```

---

## 🐛 Debugging

### Enable Debug Mode
```env
APP_DEBUG=true
```

### Log Messages
```php
Log::info('Message', ['context' => $data]);
Log::error('Error', ['exception' => $e]);
```

### Tinker Shell
```bash
php artisan tinker
> User::find(1)
> Project::active()->count()
```

### Check Logs
```bash
tail -f storage/logs/laravel.log
```

---

## 📚 Useful Links

- [Laravel Docs](https://laravel.com/docs)
- [PHP Docs](https://www.php.net/docs.php)
- [MySQL Docs](https://dev.mysql.com/doc/)
- [Telegram Bot API](https://core.telegram.org/bots/api)

---

## 🚨 Common Issues

### "Class not found"
```bash
composer dump-autoload
```

### "Database connection failed"
```bash
# Check .env credentials
# Verify database is running
mysql -u root -p -e "SELECT 1"
```

### "SQLSTATE error"
```bash
php artisan config:clear
php artisan migrate
```

### "Permissions denied"
```bash
chmod -R 775 storage
chmod -R 775 bootstrap/cache
```

### "Token mismatch"
```blade
<form method="POST">
    @csrf
    <!-- form content -->
</form>
```

---

## 🎯 Common Tasks

### Create New Project
```bash
php artisan make:model NewModel -mfc
# Creates Model, Migration, Factory, Controller
```

### Add New Permission
```php
// In migration
Permission::create([
    'name' => 'delete_projects',
    'resource' => 'projects',
    'action' => 'delete',
]);
```

### Assign Permission to Role
```php
$role->permissions()->attach($permission);
```

### Test Feature
```php
public function test_admin_can_create_project()
{
    $admin = User::factory()->admin()->create();
    $response = $this->actingAs($admin)
        ->post('/projects', ['name' => 'Test']);
    $response->assertRedirect();
}
```

---

## 📋 Environment Variables

```env
APP_NAME=Project Management Dashboard
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=project_dashboard
DB_USERNAME=root
DB_PASSWORD=

TELEGRAM_BOT_TOKEN=xxx
TELEGRAM_CHAT_ID=xxx

FILE_UPLOAD_MAX_SIZE=10240
ALLOWED_FILE_TYPES=pdf,docx,xlsx,txt,jpg,jpeg,png,gif
```

---

## 🔄 Git Workflow

```bash
# Create feature branch
git checkout -b feature/add-notifications

# Make changes and commit
git add .
git commit -m "feat: add telegram notifications"

# Push to remote
git push origin feature/add-notifications

# Create pull request
# (via GitHub interface)

# Merge and delete branch
git checkout main
git pull
git branch -d feature/add-notifications
```

---

## 💡 Code Style

### Model
```php
class Project extends Model {
    use HasFactory, SoftDeletes;
    protected $fillable = [];
    public function tasks() { return $this->hasMany(Task::class); }
    public function isOverdue(): bool { return ...; }
}
```

### Controller
```php
class ProjectController extends Controller {
    public function __construct(private ProjectService $service) {}
    public function store(Request $request) {
        $validated = $request->validate([...]);
        $project = $this->service->create($validated);
        return redirect()->route('projects.show', $project);
    }
}
```

### Service
```php
class ProjectService {
    public function create(array $data): Project {
        return DB::transaction(function () use ($data) {
            $project = Project::create($data);
            $this->log('created', $project);
            return $project;
        });
    }
}
```

---

## 📞 Quick Help

**Need documentation?** → See [README.md](./README.md)
**Setup help?** → See [SETUP.md](./SETUP.md)
**Architecture?** → See [ARCHITECTURE.md](./ARCHITECTURE.md)
**Database?** → See [DATABASE.md](./DATABASE.md)
**Code examples?** → See [docs/CODE-EXAMPLES.md](./docs/CODE-EXAMPLES.md)
**Diagrams?** → See [assets/diagrams/](./assets/diagrams/)

---

**Last Updated:** 2024
**Version:** 1.0

For complete documentation, visit [DOCUMENTATION.md](./DOCUMENTATION.md)

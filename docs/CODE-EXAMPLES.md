# Laravel Code Examples - Project Management Dashboard

Complete code examples following clean architecture and best practices.

---

## Table of Contents

1. [Models (Domain Layer)](#models-domain-layer)
2. [Controllers (Presentation Layer)](#controllers-presentation-layer)
3. [Services (Business Logic Layer)](#services-business-logic-layer)
4. [Repositories (Data Access Layer)](#repositories-data-access-layer)
5. [Migrations (Database)](#migrations-database)
6. [Routes](#routes)
7. [Views (Blade Templates)](#views-blade-templates)
8. [Validation & Requests](#validation--requests)
9. [Events & Listeners](#events--listeners)

---

## Models (Domain Layer)

### User Model

```php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasFactory, Notifiable;
    
    protected $fillable = [
        'name',
        'email',
        'password',
        'role_id',
        'telegram_chat_id',
        'status',
        'last_login_at',
    ];
    
    protected $hidden = [
        'password',
        'remember_token',
    ];
    
    protected $casts = [
        'email_verified_at' => 'datetime',
        'last_login_at' => 'datetime',
        'password' => 'hashed',
    ];
    
    // Relationships
    public function role()
    {
        return $this->belongsTo(Role::class);
    }
    
    public function projects()
    {
        return $this->hasMany(Project::class, 'assigned_to');
    }
    
    public function createdProjects()
    {
        return $this->hasMany(Project::class, 'created_by');
    }
    
    public function tasks()
    {
        return $this->hasMany(Task::class, 'assigned_to');
    }
    
    public function uploadedFiles()
    {
        return $this->hasMany(File::class, 'uploaded_by');
    }
    
    public function activityLogs()
    {
        return $this->hasMany(ActivityLog::class);
    }
    
    // Accessors
    public function hasPermission(string $permission): bool
    {
        return $this->role?->permissions()
            ->where('name', $permission)
            ->exists() ?? false;
    }
    
    public function hasAnyPermission(array $permissions): bool
    {
        return $this->role?->permissions()
            ->whereIn('name', $permissions)
            ->exists() ?? false;
    }
    
    public function isAdmin(): bool
    {
        return $this->role?->name === 'Admin';
    }
    
    public function getFullNameAttribute(): string
    {
        return ucwords($this->name);
    }
    
    // Scopes
    public function scopeActive($query)
    {
        return $query->where('status', 'active');
    }
    
    public function scopeWithRole($query, string $role)
    {
        return $query->whereHas('role', fn($q) => $q->where('name', $role));
    }
    
    // Methods
    public function updateLastLogin(): void
    {
        $this->update(['last_login_at' => now()]);
    }
}
```

### Project Model

```php
// app/Models/Project.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Project extends Model
{
    use HasFactory, SoftDeletes;
    
    protected $fillable = [
        'name',
        'client_name',
        'description',
        'deadline',
        'status',
        'assigned_to',
        'created_by',
    ];
    
    protected $casts = [
        'deadline' => 'date',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
        'deleted_at' => 'datetime',
    ];
    
    // Relationships
    public function assignee()
    {
        return $this->belongsTo(User::class, 'assigned_to');
    }
    
    public function creator()
    {
        return $this->belongsTo(User::class, 'created_by');
    }
    
    public function tasks()
    {
        return $this->hasMany(Task::class);
    }
    
    public function files()
    {
        return $this->hasMany(File::class);
    }
    
    // Accessors
    public function isOverdue(): bool
    {
        return $this->deadline < now() && $this->status !== 'completed';
    }
    
    public function isActive(): bool
    {
        return $this->status === 'active';
    }
    
    public function getDaysUntilDeadlineAttribute(): int
    {
        return $this->deadline->diffInDays(now(), false);
    }
    
    public function getProgressAttribute(): float
    {
        $totalTasks = $this->tasks()->count();
        
        if ($totalTasks === 0) {
            return 0;
        }
        
        $completedTasks = $this->tasks()
            ->where('status', 'completed')
            ->count();
        
        return round(($completedTasks / $totalTasks) * 100, 2);
    }
    
    // Scopes
    public function scopeActive($query)
    {
        return $query->where('status', 'active');
    }
    
    public function scopeOverdue($query)
    {
        return $query->where('deadline', '<', now())
                    ->whereNot('status', 'completed');
    }
    
    public function scopeByStatus($query, string $status)
    {
        return $query->where('status', $status);
    }
    
    public function scopeAssignedTo($query, int $userId)
    {
        return $query->where('assigned_to', $userId);
    }
    
    // Methods
    public function getCompletedTasksCount(): int
    {
        return $this->tasks()
            ->where('status', 'completed')
            ->count();
    }
    
    public function getPendingTasksCount(): int
    {
        return $this->tasks()
            ->whereIn('status', ['pending', 'in_progress'])
            ->count();
    }
    
    public function getOverdueTasksCount(): int
    {
        return $this->tasks()
            ->where('deadline', '<', now())
            ->whereNot('status', 'completed')
            ->count();
    }
}
```

### Task Model

```php
// app/Models/Task.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Task extends Model
{
    use HasFactory, SoftDeletes;
    
    protected $fillable = [
        'project_id',
        'name',
        'description',
        'priority',
        'deadline',
        'status',
        'assigned_to',
        'progress_percentage',
    ];
    
    protected $casts = [
        'deadline' => 'date',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
        'deleted_at' => 'datetime',
    ];
    
    // Relationships
    public function project()
    {
        return $this->belongsTo(Project::class);
    }
    
    public function assignee()
    {
        return $this->belongsTo(User::class, 'assigned_to');
    }
    
    public function files()
    {
        return $this->hasMany(File::class);
    }
    
    // Accessors
    public function isOverdue(): bool
    {
        return $this->deadline < now() && $this->status !== 'completed';
    }
    
    public function isDueToday(): bool
    {
        return $this->deadline->isToday();
    }
    
    public function getDaysUntilDeadlineAttribute(): int
    {
        return $this->deadline->diffInDays(now(), false);
    }
    
    public function getPriorityBadgeAttribute(): string
    {
        return match($this->priority) {
            'critical' => 'danger',
            'high' => 'warning',
            'medium' => 'info',
            'low' => 'secondary',
            default => 'light',
        };
    }
    
    // Scopes
    public function scopeOverdue($query)
    {
        return $query->where('deadline', '<', now())
                    ->whereNot('status', 'completed');
    }
    
    public function scopeByPriority($query, string $priority)
    {
        return $query->where('priority', $priority);
    }
    
    public function scopeByStatus($query, string $status)
    {
        return $query->where('status', $status);
    }
    
    public function scopeAssignedTo($query, int $userId)
    {
        return $query->where('assigned_to', $userId);
    }
    
    public function scopeDueToday($query)
    {
        return $query->where('deadline', now()->format('Y-m-d'));
    }
    
    // Methods
    public function getFileCount(): int
    {
        return $this->files()->count();
    }
    
    public function markAsCompleted(): void
    {
        $this->update([
            'status' => 'completed',
            'progress_percentage' => 100,
        ]);
    }
    
    public function updateProgress(int $percentage): void
    {
        if ($percentage > 100) {
            $percentage = 100;
        }
        
        $status = $percentage === 100 ? 'completed' : 'in_progress';
        
        $this->update([
            'progress_percentage' => $percentage,
            'status' => $status,
        ]);
    }
}
```

---

## Controllers (Presentation Layer)

### ProjectController

```php
// app/Http/Controllers/ProjectController.php

namespace App\Http\Controllers;

use App\Models\Project;
use App\Services\ProjectService;
use Illuminate\Http\Request;

class ProjectController extends Controller
{
    public function __construct(
        private ProjectService $projectService
    ) {}
    
    public function index()
    {
        $projects = Project::with('assignee')
            ->where('created_by', auth()->id())
            ->orWhere('assigned_to', auth()->id())
            ->paginate(15);
        
        return view('projects.index', compact('projects'));
    }
    
    public function show(Project $project)
    {
        $this->authorize('view', $project);
        
        $tasks = $project->tasks()
            ->with('assignee')
            ->paginate(10);
        
        $stats = [
            'total_tasks' => $project->tasks()->count(),
            'completed_tasks' => $project->getCompletedTasksCount(),
            'pending_tasks' => $project->getPendingTasksCount(),
            'overdue_tasks' => $project->getOverdueTasksCount(),
            'progress' => $project->progress,
        ];
        
        return view('projects.show', compact('project', 'tasks', 'stats'));
    }
    
    public function create()
    {
        $this->authorize('create', Project::class);
        
        return view('projects.create');
    }
    
    public function store(Request $request)
    {
        $this->authorize('create', Project::class);
        
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'client_name' => 'required|string|max:255',
            'description' => 'nullable|string|max:1000',
            'deadline' => 'required|date|after:today',
            'assigned_to' => 'required|exists:users,id',
        ]);
        
        $validated['created_by'] = auth()->id();
        
        $project = $this->projectService->createProject($validated);
        
        return redirect()
            ->route('projects.show', $project)
            ->with('success', 'Project created successfully!');
    }
    
    public function edit(Project $project)
    {
        $this->authorize('update', $project);
        
        return view('projects.edit', compact('project'));
    }
    
    public function update(Request $request, Project $project)
    {
        $this->authorize('update', $project);
        
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'client_name' => 'required|string|max:255',
            'description' => 'nullable|string|max:1000',
            'deadline' => 'required|date',
            'status' => 'required|in:active,on_hold,completed,cancelled',
            'assigned_to' => 'required|exists:users,id',
        ]);
        
        $this->projectService->updateProject($project, $validated);
        
        return redirect()
            ->route('projects.show', $project)
            ->with('success', 'Project updated successfully!');
    }
    
    public function destroy(Project $project)
    {
        $this->authorize('delete', $project);
        
        $this->projectService->deleteProject($project);
        
        return redirect()
            ->route('projects.index')
            ->with('success', 'Project deleted successfully!');
    }
}
```

### TaskController

```php
// app/Http/Controllers/TaskController.php

namespace App\Http\Controllers;

use App\Models\Task;
use App\Models\Project;
use App\Services\TaskService;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function __construct(
        private TaskService $taskService
    ) {}
    
    public function index()
    {
        $tasks = Task::with(['project', 'assignee'])
            ->where('assigned_to', auth()->id())
            ->paginate(15);
        
        return view('tasks.index', compact('tasks'));
    }
    
    public function show(Task $task)
    {
        $this->authorize('view', $task);
        
        $files = $task->files()->paginate(10);
        
        return view('tasks.show', compact('task', 'files'));
    }
    
    public function create(Project $project)
    {
        $this->authorize('create', Task::class);
        
        return view('tasks.create', compact('project'));
    }
    
    public function store(Request $request, Project $project)
    {
        $this->authorize('create', Task::class);
        
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'description' => 'nullable|string|max:1000',
            'priority' => 'required|in:low,medium,high,critical',
            'deadline' => 'required|date|after:today',
            'assigned_to' => 'required|exists:users,id',
        ]);
        
        $validated['project_id'] = $project->id;
        $validated['status'] = 'pending';
        $validated['progress_percentage'] = 0;
        
        $task = $this->taskService->createTask($validated);
        
        return redirect()
            ->route('tasks.show', $task)
            ->with('success', 'Task created successfully!');
    }
    
    public function edit(Task $task)
    {
        $this->authorize('update', $task);
        
        return view('tasks.edit', compact('task'));
    }
    
    public function update(Request $request, Task $task)
    {
        $this->authorize('update', $task);
        
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'description' => 'nullable|string|max:1000',
            'priority' => 'required|in:low,medium,high,critical',
            'deadline' => 'required|date',
            'status' => 'required|in:pending,in_progress,completed,overdue,blocked',
            'progress_percentage' => 'required|integer|min:0|max:100',
            'assigned_to' => 'required|exists:users,id',
        ]);
        
        $this->taskService->updateTask($task, $validated);
        
        return redirect()
            ->route('tasks.show', $task)
            ->with('success', 'Task updated successfully!');
    }
    
    public function updateStatus(Request $request, Task $task)
    {
        $this->authorize('update', $task);
        
        $validated = $request->validate([
            'status' => 'required|in:pending,in_progress,completed,overdue,blocked',
            'progress_percentage' => 'nullable|integer|min:0|max:100',
        ]);
        
        $this->taskService->updateTaskStatus($task, $validated);
        
        return response()->json([
            'success' => true,
            'message' => 'Task status updated successfully',
            'task' => $task,
        ]);
    }
}
```

---

## Services (Business Logic Layer)

### ProjectService

```php
// app/Services/ProjectService.php

namespace App\Services;

use App\Models\Project;
use App\Models\ActivityLog;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

class ProjectService
{
    public function __construct(
        private TelegramNotificationService $telegramService,
        private ActivityLogService $activityLog
    ) {}
    
    public function createProject(array $data): Project
    {
        try {
            return DB::transaction(function () use ($data) {
                $project = Project::create($data);
                
                $this->activityLog->log('project_created', [
                    'project_id' => $project->id,
                    'user_id' => auth()->id(),
                    'data' => $project->toArray(),
                ]);
                
                // Notify assignee
                $assignee = $project->assignee;
                if ($assignee && $assignee->telegram_chat_id) {
                    $this->telegramService->notify(
                        $assignee->telegram_chat_id,
                        "📋 New Project Assigned: {$project->name}"
                    );
                }
                
                return $project;
            });
        } catch (\Exception $e) {
            Log::error('Project creation failed', ['error' => $e->getMessage()]);
            throw new \Exception('Failed to create project');
        }
    }
    
    public function updateProject(Project $project, array $data): Project
    {
        try {
            return DB::transaction(function () use ($project, $data) {
                $originalData = $project->toArray();
                
                $project->update($data);
                
                $this->activityLog->log('project_updated', [
                    'project_id' => $project->id,
                    'user_id' => auth()->id(),
                    'changes' => array_diff_assoc($data, $originalData),
                ]);
                
                // Notify on status change
                if (isset($data['status']) && $originalData['status'] !== $data['status']) {
                    $this->notifyStatusChange($project, $originalData['status'], $data['status']);
                }
                
                return $project;
            });
        } catch (\Exception $e) {
            Log::error('Project update failed', ['error' => $e->getMessage()]);
            throw new \Exception('Failed to update project');
        }
    }
    
    public function deleteProject(Project $project): bool
    {
        try {
            return DB::transaction(function () use ($project) {
                $projectName = $project->name;
                $projectId = $project->id;
                
                // Delete related tasks
                $project->tasks()->each(function ($task) {
                    $task->files()->delete();
                    $task->delete();
                });
                
                // Delete project files
                $project->files()->each(function ($file) {
                    $file->delete();
                });
                
                $project->delete();
                
                $this->activityLog->log('project_deleted', [
                    'project_id' => $projectId,
                    'user_id' => auth()->id(),
                    'project_name' => $projectName,
                ]);
                
                return true;
            });
        } catch (\Exception $e) {
            Log::error('Project deletion failed', ['error' => $e->getMessage()]);
            throw new \Exception('Failed to delete project');
        }
    }
    
    public function getProjectStatistics(): array
    {
        return cache()->remember('project:stats', 3600, function () {
            return [
                'total' => Project::count(),
                'active' => Project::active()->count(),
                'completed' => Project::where('status', 'completed')->count(),
                'overdue' => Project::overdue()->count(),
            ];
        });
    }
    
    private function notifyStatusChange(Project $project, string $oldStatus, string $newStatus): void
    {
        $assignee = $project->assignee;
        if (!$assignee || !$assignee->telegram_chat_id) {
            return;
        }
        
        $message = "📋 Project '{$project->name}' status changed from {$oldStatus} to {$newStatus}";
        
        $this->telegramService->notify($assignee->telegram_chat_id, $message);
    }
}
```

### FileValidationService

```php
// app/Services/FileValidationService.php

namespace App\Services;

use Illuminate\Http\UploadedFile;
use App\Exceptions\InvalidFileException;

class FileValidationService
{
    private const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
    
    private const ALLOWED_EXTENSIONS = [
        'pdf', 'docx', 'xlsx', 'txt', 
        'jpg', 'jpeg', 'png', 'gif'
    ];
    
    private const MIME_TYPES = [
        'application/pdf' => 'pdf',
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document' => 'docx',
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' => 'xlsx',
        'text/plain' => 'txt',
        'image/jpeg' => 'jpg',
        'image/png' => 'png',
        'image/gif' => 'gif',
    ];
    
    public function validate(UploadedFile $file): bool
    {
        $this->validateSize($file);
        $this->validateExtension($file);
        $this->validateMimeType($file);
        
        return true;
    }
    
    private function validateSize(UploadedFile $file): void
    {
        if ($file->getSize() > self::MAX_FILE_SIZE) {
            throw new InvalidFileException(
                "File size exceeds maximum of " . 
                ($this::MAX_FILE_SIZE / (1024 * 1024)) . "MB"
            );
        }
        
        if ($file->getSize() === 0) {
            throw new InvalidFileException("File is empty");
        }
    }
    
    private function validateExtension(UploadedFile $file): void
    {
        $extension = strtolower($file->getClientOriginalExtension());
        
        if (!in_array($extension, self::ALLOWED_EXTENSIONS)) {
            throw new InvalidFileException(
                "File type '{$extension}' is not allowed. " .
                "Allowed types: " . implode(', ', self::ALLOWED_EXTENSIONS)
            );
        }
    }
    
    private function validateMimeType(UploadedFile $file): void
    {
        $mimeType = $file->getMimeType();
        
        if (!array_key_exists($mimeType, self::MIME_TYPES)) {
            throw new InvalidFileException(
                "File MIME type '{$mimeType}' is not allowed"
            );
        }
    }
    
    public function generateStoragePath(string $originalName): string
    {
        $extension = pathinfo($originalName, PATHINFO_EXTENSION);
        $filename = uniqid('file_') . '_' . time() . '.' . $extension;
        
        return 'uploads/' . date('Y/m/d') . '/' . $filename;
    }
}
```

### TelegramNotificationService

```php
// app/Services/TelegramNotificationService.php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class TelegramNotificationService
{
    private string $botToken;
    private string $apiUrl = 'https://api.telegram.org/bot';
    
    public function __construct()
    {
        $this->botToken = config('services.telegram.token');
    }
    
    public function notify(string $chatId, string $message): bool
    {
        if (!$chatId || !$this->botToken) {
            Log::warning('Telegram notification skipped', [
                'chat_id' => $chatId ?? 'missing',
                'reason' => 'Missing configuration'
            ]);
            return false;
        }
        
        try {
            $response = Http::timeout(10)
                ->post($this->getApiUrl('sendMessage'), [
                    'chat_id' => $chatId,
                    'text' => $message,
                    'parse_mode' => 'Markdown',
                ]);
            
            if (!$response->successful()) {
                Log::error('Telegram notification failed', [
                    'response' => $response->body(),
                    'status' => $response->status(),
                ]);
                return false;
            }
            
            return true;
        } catch (\Exception $e) {
            Log::error('Telegram notification error', [
                'error' => $e->getMessage(),
                'chat_id' => $chatId,
            ]);
            return false;
        }
    }
    
    public function sendTaskReminder(Task $task, User $user): bool
    {
        $message = $this->formatReminderMessage($task);
        
        return $this->notify($user->telegram_chat_id, $message);
    }
    
    private function formatReminderMessage(Task $task): string
    {
        return sprintf(
            "⏰ *Task Reminder*\n\n" .
            "*Task:* %s\n" .
            "*Project:* %s\n" .
            "*Due:* %s\n" .
            "*Priority:* %s\n\n" .
            "_Deadline is tomorrow. Please complete this task._",
            $task->name,
            $task->project->name,
            $task->deadline->format('M d, Y'),
            strtoupper($task->priority)
        );
    }
    
    private function getApiUrl(string $method): string
    {
        return "{$this->apiUrl}{$this->botToken}/{$method}";
    }
}
```

---

## Repositories (Data Access Layer)

### ProjectRepository

```php
// app/Repositories/ProjectRepository.php

namespace App\Repositories;

use App\Models\Project;
use Illuminate\Pagination\Paginate;

class ProjectRepository
{
    public function getAll($paginate = true)
    {
        $query = Project::with('assignee', 'creator', 'tasks');
        
        return $paginate ? $query->paginate(15) : $query->get();
    }
    
    public function findById(int $id)
    {
        return Project::with('assignee', 'creator', 'tasks', 'files')
            ->find($id);
    }
    
    public function create(array $data): Project
    {
        return Project::create($data);
    }
    
    public function update(Project $project, array $data): Project
    {
        $project->update($data);
        return $project;
    }
    
    public function delete(Project $project): bool
    {
        return $project->delete();
    }
    
    public function getByUser(int $userId, $paginate = true)
    {
        $query = Project::where('assigned_to', $userId)
            ->orWhere('created_by', $userId)
            ->with('assignee', 'creator');
        
        return $paginate ? $query->paginate(15) : $query->get();
    }
    
    public function getActive()
    {
        return Project::active()->with('assignee')->get();
    }
    
    public function getOverdue()
    {
        return Project::overdue()->with('assignee')->get();
    }
    
    public function getByStatus(string $status, $paginate = true)
    {
        $query = Project::where('status', $status)
            ->with('assignee', 'creator');
        
        return $paginate ? $query->paginate(15) : $query->get();
    }
}
```

---

## Migrations (Database)

### Create Projects Table

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
                  ->constrained('users')
                  ->onDelete('cascade');
            $table->foreignId('created_by')
                  ->constrained('users')
                  ->onDelete('cascade');
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

---

## Routes

### Web Routes

```php
// routes/web.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProjectController;
use App\Http\Controllers\TaskController;
use App\Http\Controllers\FileController;
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\AuthController;

Route::middleware('guest')->group(function () {
    Route::get('login', [AuthController::class, 'showLogin'])->name('login');
    Route::post('login', [AuthController::class, 'login'])->name('login.post');
    Route::get('register', [AuthController::class, 'showRegister'])->name('register');
    Route::post('register', [AuthController::class, 'register'])->name('register.post');
});

Route::middleware('auth')->group(function () {
    Route::post('logout', [AuthController::class, 'logout'])->name('logout');
    
    Route::get('dashboard', [DashboardController::class, 'index'])->name('dashboard');
    
    // Projects
    Route::resource('projects', ProjectController::class);
    
    // Tasks
    Route::resource('tasks', TaskController::class);
    Route::patch('tasks/{task}/status', [TaskController::class, 'updateStatus'])
        ->name('tasks.update-status');
    
    // Files
    Route::post('files/upload', [FileController::class, 'store'])->name('files.store');
    Route::get('files/{file}/download', [FileController::class, 'download'])
        ->name('files.download');
    Route::delete('files/{file}', [FileController::class, 'destroy'])
        ->name('files.destroy');
});

Route::get('/', function () {
    return redirect()->route('dashboard');
});
```

---

## Views (Blade Templates)

### Projects Index

```blade
<!-- resources/views/projects/index.blade.php -->

@extends('layouts.app')

@section('content')
<div class="container mx-auto py-8">
    <div class="flex justify-between items-center mb-6">
        <h1 class="text-3xl font-bold">Projects</h1>
        @if(auth()->user()->isAdmin())
            <a href="{{ route('projects.create') }}" class="btn btn-primary">
                + Create Project
            </a>
        @endif
    </div>
    
    @if($projects->isEmpty())
        <div class="alert alert-info">
            No projects found. 
            @if(auth()->user()->isAdmin())
                <a href="{{ route('projects.create') }}">Create one now</a>
            @endif
        </div>
    @else
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            @foreach($projects as $project)
                <div class="card shadow-lg">
                    <div class="card-body">
                        <h2 class="card-title">{{ $project->name }}</h2>
                        <p class="text-gray-600">{{ $project->client_name }}</p>
                        
                        <div class="my-4">
                            <span class="badge" :class="{
                                'badge-success': $project->isActive(),
                                'badge-warning': $project->status === 'on_hold',
                                'badge-info': $project->status === 'completed',
                            }">
                                {{ ucfirst($project->status) }}
                            </span>
                        </div>
                        
                        <div class="flex justify-between items-center text-sm text-gray-500">
                            <span>📅 {{ $project->deadline->format('M d, Y') }}</span>
                            <span>📊 {{ $project->progress }}%</span>
                        </div>
                        
                        <div class="card-actions justify-end mt-4">
                            <a href="{{ route('projects.show', $project) }}" class="btn btn-sm btn-ghost">
                                View
                            </a>
                            @can('update', $project)
                                <a href="{{ route('projects.edit', $project) }}" class="btn btn-sm btn-ghost">
                                    Edit
                                </a>
                            @endcan
                        </div>
                    </div>
                </div>
            @endforeach
        </div>
        
        <div class="mt-6">
            {{ $projects->links() }}
        </div>
    @endif
</div>
@endsection
```

---

**[← Back to Database Design](../DATABASE.md) | [→ Continue with more examples in repository](../docs/)**

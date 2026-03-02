# Project Management Dashboard

A comprehensive, production-ready project management system built with Laravel. Designed for teams to efficiently manage projects, tasks, and collaborate seamlessly with real-time Telegram notifications.

## 🎯 Features

### Authentication & Authorization
- Secure user authentication with session management
- Role-based access control (Admin & User roles)
- Permission-based authorization
- Session timeout protection

### Project Management
- Create, read, update, delete projects
- Assign projects to team members
- Track project status (Active, On Hold, Completed)
- Client information management
- Deadline tracking

### Task Management
- CRUD operations for tasks
- Priority levels (Low, Medium, High, Critical)
- Task assignment to users
- Progress tracking and status updates
- Deadline management
- Task activity logging

### File Management
- Support for multiple file types (PDF, DOCX, XLSX, TXT, Images)
- File preview functionality
- Secure file downloads
- File size validation and constraints
- Attachment organization per project/task

### Dashboard Analytics
- Real-time statistics dashboard
- Total projects counter
- Active projects tracking
- Completed tasks metrics
- Overdue tasks alerts
- Visual data representation

### Notifications
- Telegram Bot integration
- Automatic deadline reminders (1 day before)
- Task assignment notifications
- Project status change alerts
- Real-time notification delivery

## 🛠 Tech Stack

![PHP](https://img.shields.io/badge/PHP-8-777BB4?style=flat-square&logo=php)
![Laravel](https://img.shields.io/badge/Laravel-11-FF2D20?style=flat-square&logo=laravel)
![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?style=flat-square&logo=mysql)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript)
![HTML5](https://img.shields.io/badge/HTML5-Latest-E34C26?style=flat-square&logo=html5)
![CSS3](https://img.shields.io/badge/CSS3-Latest-1572B6?style=flat-square&logo=css3)

**Backend:** Laravel 11 | PHP 8.2+
**Frontend:** HTML5, CSS3, Vanilla JavaScript
**Database:** MySQL 8 / PostgreSQL 14+
**External APIs:** Telegram Bot API
**Storage:** Local filesystem

## 📋 Requirements

- PHP >= 8.2
- Composer
- MySQL 8.0+ or PostgreSQL 14+
- Node.js & npm (for frontend build tools, optional)
- Telegram Bot Token (for notifications)

## 🚀 Quick Start

### Installation

Detailed setup instructions are available in [SETUP.md](./SETUP.md).

```bash
# Clone the repository
git clone https://github.com/yourusername/project-management-dashboard.git
cd project-management-dashboard

# Install PHP dependencies
composer install

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Run database migrations
php artisan migrate

# Seed sample data (optional)
php artisan db:seed

# Start development server
php artisan serve
```

Access the application at `http://localhost:8000`

**Default Credentials (Demo):**
- Admin: `admin@example.com` / `password`
- User: `user@example.com` / `password`

## 📂 Project Structure

```
project-management-dashboard/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── AuthController.php
│   │   │   ├── ProjectController.php
│   │   │   ├── TaskController.php
│   │   │   ├── FileController.php
│   │   │   └── DashboardController.php
│   │   └── Middleware/
│   ├── Models/
│   │   ├── User.php
│   │   ├── Project.php
│   │   ├── Task.php
│   │   ├── File.php
│   │   ├── Role.php
│   │   └── ActivityLog.php
│   └── Services/
│       ├── TelegramNotificationService.php
│       ├── FileValidationService.php
│       └── TaskReminderService.php
├── database/
│   ├── migrations/
│   └── seeders/
├── resources/
│   ├── views/
│   │   ├── layouts/
│   │   ├── auth/
│   │   ├── projects/
│   │   ├── tasks/
│   │   ├── files/
│   │   └── dashboard/
│   └── css/
├── routes/
│   ├── web.php
│   └── api.php
├── assets/
│   ├── screenshots/
│   ├── diagrams/
│   └── docs/
└── storage/
    └── app/
        └── uploads/
```

## 🏗 Architecture

The application follows **Clean Architecture** principles with:

- **MVC Pattern**: Models, Views, Controllers separation
- **Service Layer**: Business logic encapsulation
- **Repository Pattern**: Data access abstraction
- **Middleware**: Request/response pipeline
- **Events**: Decoupled event handling for notifications

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed design documentation.

## 📊 Database Schema

The database consists of 7 main tables with relationships:

- **users**: User accounts and authentication
- **roles**: Role definitions (Admin, User)
- **permissions**: Permission definitions
- **projects**: Project information
- **tasks**: Task details and assignments
- **files**: File metadata and storage
- **activity_logs**: Audit trail for all actions

See [DATABASE.md](./DATABASE.md) for complete ERD and table specifications.

## 🔄 System Flow

```
User Login → Dashboard → [Admin/User Routes]
             ├─ Admin: Create Project → Assign Tasks → View Analytics
             └─ User: View Tasks → Update Progress → Upload Files
                      
Telegram Notification System:
Task Deadline (1 day before) → Service → Telegram API → User Notification
```

See [diagrams/system-flow.md](./assets/diagrams/system-flow.md) for visual flowchart.

## 🔐 Security Features

- **Authentication**: Session-based with secure password hashing
- **Authorization**: Role and permission-based access control
- **Input Validation**: Server-side validation on all inputs
- **CSRF Protection**: Token-based protection on forms
- **File Security**: Whitelist-based file type validation
- **SQL Injection Prevention**: Parameterized queries via Eloquent ORM
- **Activity Logging**: All critical actions are logged

## 🌐 API Endpoints (RESTful)

### Authentication
- `POST /api/login` - User login
- `POST /api/logout` - User logout
- `POST /api/register` - New user registration

### Projects
- `GET /api/projects` - List all projects
- `POST /api/projects` - Create new project
- `GET /api/projects/{id}` - Get project details
- `PUT /api/projects/{id}` - Update project
- `DELETE /api/projects/{id}` - Delete project

### Tasks
- `GET /api/tasks` - List all tasks
- `POST /api/tasks` - Create new task
- `PUT /api/tasks/{id}` - Update task
- `DELETE /api/tasks/{id}` - Delete task

### Files
- `POST /api/files/upload` - Upload file
- `GET /api/files/{id}` - Download file
- `DELETE /api/files/{id}` - Delete file

### Dashboard
- `GET /api/dashboard/statistics` - Get dashboard stats

## 📸 Screenshots

### Dashboard
![Dashboard](./assets/screenshots/dashboard.png)

### Projects Management
![Projects](./assets/screenshots/projects.png)

### Tasks Management
![Tasks](./assets/screenshots/tasks.png)

### File Management
![Files](./assets/screenshots/files.png)

## 🧪 Testing

Run tests with:
```bash
php artisan test
```

Test coverage includes:
- Authentication tests
- Authorization tests
- CRUD operations
- File validation
- Notification delivery

## 📚 Documentation

- [SETUP.md](./SETUP.md) - Installation and configuration guide
- [ARCHITECTURE.md](./ARCHITECTURE.md) - System architecture and design patterns
- [DATABASE.md](./DATABASE.md) - Database schema and relationships
- [API.md](./docs/API.md) - Complete API documentation

## 🔧 Configuration

All configuration is managed through `.env` file:

```
APP_NAME="Project Management Dashboard"
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=project_dashboard
DB_USERNAME=root
DB_PASSWORD=

TELEGRAM_BOT_TOKEN=your_bot_token_here
FILE_UPLOAD_MAX_SIZE=10240 # in KB
```

## 🚦 Development Workflow

1. Create a feature branch: `git checkout -b feature/feature-name`
2. Make changes following coding standards
3. Commit with meaningful messages: `git commit -m "feat: add new feature"`
4. Push to branch: `git push origin feature/feature-name`
5. Create Pull Request with detailed description

## 📝 Coding Standards

- **PSR-12**: PHP coding standard
- **Clean Code**: Self-documenting code with clear naming
- **SOLID Principles**: Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion
- **DRY**: Don't repeat yourself

## 🤝 Contributing

Contributions are welcome! Please follow the development workflow and ensure code quality.

## 📄 License

This project is open source and available under the MIT License.

## 👨‍💻 Author

Your Name - [GitHub Profile](https://github.com/yourusername)

## 🆘 Support

For issues and questions:
1. Check existing [GitHub Issues](https://github.com/yourusername/project-management-dashboard/issues)
2. Create detailed bug reports with reproduction steps
3. Ask questions in Discussions

## 🎓 Learning Resources

- [Laravel Documentation](https://laravel.com/docs)
- [PHP Documentation](https://www.php.net/docs.php)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Telegram Bot API](https://core.telegram.org/bots/api)

---

**[📖 View Full Documentation](./SETUP.md)**

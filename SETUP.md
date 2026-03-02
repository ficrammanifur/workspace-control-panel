# Setup Guide - Project Management Dashboard

Complete step-by-step installation and configuration guide for the Project Management Dashboard.

**Navigation:** [← README.md](./README.md) | [ARCHITECTURE.md →](./ARCHITECTURE.md)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Installation Steps](#installation-steps)
4. [Database Configuration](#database-configuration)
5. [Telegram Bot Setup](#telegram-bot-setup)
6. [Verification Checklist](#verification-checklist)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

- **PHP**: 8.2 or higher
- **Composer**: Latest version
- **Database**: MySQL 8.0+ or PostgreSQL 14+
- **Node.js**: 16+ (optional, for build tools)
- **Git**: For version control
- **Terminal/CMD**: Command line access

### Verify Installations

```bash
# Check PHP version
php --version
# Output: PHP 8.2.x or higher

# Check Composer
composer --version
# Output: Composer version 2.x.x

# Check MySQL
mysql --version
# Output: mysql Ver 8.0.x or higher
```

---

## Environment Setup

### Step 1: Clone Repository

```bash
# Clone the repository
git clone https://github.com/yourusername/project-management-dashboard.git

# Navigate to project directory
cd project-management-dashboard
```

### Step 2: Install Dependencies

```bash
# Install PHP dependencies with Composer
composer install

# This will install:
# - Laravel framework and dependencies
# - Database libraries
# - Validation libraries
# - And other required packages
```

**Expected output:** "Generating optimized autoload files"

### Step 3: Create Environment File

```bash
# Copy example environment file
cp .env.example .env
```

### Step 4: Generate Application Key

```bash
php artisan key:generate
```

This generates a unique encryption key for your application.

---

## Installation Steps

### Step 5: Configure Database Connection

Edit `.env` file with your database credentials:

```env
APP_NAME="Project Management Dashboard"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=project_dashboard
DB_USERNAME=root
DB_PASSWORD=your_password_here

# For PostgreSQL, use:
# DB_CONNECTION=pgsql
# DB_HOST=127.0.0.1
# DB_PORT=5432
# DB_DATABASE=project_dashboard
```

**Database Creation:**

```bash
# MySQL
mysql -u root -p
CREATE DATABASE project_dashboard;
EXIT;

# Or PostgreSQL
createdb project_dashboard
```

### Step 6: Run Database Migrations

```bash
# Create all tables
php artisan migrate

# Output should show: "Migrated [timestamp]_create_users_table"
```

**Tables created:**
- users
- roles
- permissions
- projects
- tasks
- files
- activity_logs

### Step 7: Seed Sample Data (Optional)

```bash
# Populate database with sample data
php artisan db:seed

# This creates:
# - 2 admin users
# - 5 regular users
# - 10 sample projects
# - 30 sample tasks
# - 20 sample files
```

**Sample Login Credentials:**

| Role  | Email              | Password | Purpose        |
|-------|------------------|----------|-----------------|
| Admin | admin@example.com  | password | Full access     |
| User  | user@example.com   | password | Limited access  |

---

## Database Configuration

### MySQL Setup

```bash
# Create user and grant privileges
mysql -u root -p
```

```sql
CREATE DATABASE project_dashboard;
CREATE USER 'dashboard_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON project_dashboard.* TO 'dashboard_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Update `.env`:
```env
DB_USERNAME=dashboard_user
DB_PASSWORD=secure_password
```

### PostgreSQL Setup

```bash
# Create database and user
createdb project_dashboard
createuser dashboard_user -P
# Enter password when prompted
```

```bash
# Grant privileges
psql -U postgres -d project_dashboard
```

```sql
GRANT ALL PRIVILEGES ON DATABASE project_dashboard TO dashboard_user;
\q
```

### File Storage Configuration

Create upload directories:

```bash
# Create storage directories
mkdir -p storage/app/uploads/projects
mkdir -p storage/app/uploads/tasks
mkdir -p storage/app/uploads/temp
mkdir -p public/uploads

# Set proper permissions
chmod -R 775 storage
chmod -R 775 bootstrap/cache
chmod -R 755 public
```

Configure `.env`:

```env
APP_STORAGE_PATH=storage/app/uploads
FILE_UPLOAD_MAX_SIZE=10240
ALLOWED_FILE_TYPES=pdf,docx,xlsx,txt,jpg,jpeg,png,gif
```

---

## Telegram Bot Setup

### Step 1: Create Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/start` command
3. Send `/newbot` command
4. Follow the prompts:
   - Bot name: "Project Dashboard Bot"
   - Bot username: "project_dashboard_bot" (must be unique)
5. Copy the **API Token** (you'll need this)

### Step 2: Configure Telegram in .env

```env
TELEGRAM_BOT_TOKEN=your_bot_token_here
TELEGRAM_CHAT_ID=your_chat_id_here
TELEGRAM_API_URL=https://api.telegram.org/bot
```

### Step 3: Get Your Telegram Chat ID

```bash
# Send a message to your bot in Telegram
# Visit this URL in browser (replace token)
https://api.telegram.org/botYOUR_TOKEN/getUpdates

# Look for the "id" field in the response
# This is your TELEGRAM_CHAT_ID
```

### Step 4: Test Bot Connection

```bash
# Run test command
php artisan tinker

# Inside tinker shell
> $service = app('App\Services\TelegramNotificationService');
> $service->sendMessage('Test notification from Project Dashboard');
> exit
```

**Expected output:** "True" (message sent successfully)

---

## Verification Checklist

After installation, verify everything is working:

### Application Health Check

```bash
# 1. Start development server
php artisan serve

# 2. In browser, visit:
http://localhost:8000

# 3. You should see the login page
```

### Database Verification

```bash
# Check migrations
php artisan migrate:status

# All migrations should show "Migrated"
```

### Configuration Verification

```bash
# Check environment setup
php artisan tinker

# Verify database connection
> DB::connection()->getPdo()
# Should return a PDO object without error

# Check if tables exist
> DB::select('SHOW TABLES')
# Should list all 7 tables

exit
```

### File Permissions

```bash
# Verify storage is writable
touch storage/app/test.txt
rm storage/app/test.txt

# Verify bootstrap/cache is writable
touch bootstrap/cache/test.txt
rm bootstrap/cache/test.txt
```

### Checklist

- [ ] PHP version is 8.2+
- [ ] Composer dependencies installed
- [ ] `.env` file created and configured
- [ ] Application key generated
- [ ] Database created
- [ ] Migrations ran successfully
- [ ] Sample data seeded
- [ ] Application accessible at http://localhost:8000
- [ ] Login works with demo credentials
- [ ] Telegram bot token configured
- [ ] File upload directories created
- [ ] Storage is writable

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Class not found" Errors

```bash
# Solution: Regenerate autoload files
composer dump-autoload

# Clear all caches
php artisan config:clear
php artisan cache:clear
php artisan view:clear
```

#### Issue: "Database connection failed"

```bash
# Verify connection details in .env
# Check database is running:

# MySQL
mysql -u root -p -e "SELECT 1"

# PostgreSQL
psql -U postgres -c "SELECT 1"

# If connection fails, check:
# 1. Database server is running
# 2. Credentials are correct
# 3. Database exists
```

#### Issue: "SQLSTATE[HY000]: General error"

```bash
# Clear cached configuration
php artisan config:clear

# Retry migrations
php artisan migrate
```

#### Issue: "Storage directory not writable"

```bash
# Fix permissions
sudo chown -R www-data:www-data storage
sudo chown -R www-data:www-data bootstrap/cache
chmod -R 775 storage bootstrap/cache

# Or for development
chmod -R 777 storage bootstrap/cache
```

#### Issue: "Telegram bot not sending messages"

```bash
# 1. Verify token in .env is correct
# 2. Test API connection
curl https://api.telegram.org/botYOUR_TOKEN/getMe

# 3. Check Telegram chat ID is correct
# 4. Check bot is not rate limited
```

#### Issue: "File upload not working"

```bash
# Check directory exists
ls -la storage/app/uploads/

# Check permissions
chmod -R 755 storage/app/uploads

# Check .env file settings
FILE_UPLOAD_MAX_SIZE=10240
ALLOWED_FILE_TYPES=pdf,docx,xlsx,txt,jpg,jpeg,png,gif
```

#### Issue: "Class App\Http\Controllers\ProjectController not found"

```bash
# Regenerate autoloader
composer dump-autoload -o

# Verify file exists at:
# app/Http/Controllers/ProjectController.php
```

#### Issue: "Port 8000 already in use"

```bash
# Use different port
php artisan serve --port=8001

# Or kill process using port 8000
# On Linux/Mac
lsof -ti:8000 | xargs kill -9

# On Windows
netstat -ano | findstr :8000
taskkill /PID PID_NUMBER /F
```

### Getting Help

1. **Check Laravel Logs:**
   ```bash
   tail -f storage/logs/laravel.log
   ```

2. **Enable Debug Mode:**
   - Set `APP_DEBUG=true` in `.env` (only for development)
   - Check error messages in browser

3. **Database Debugging:**
   ```bash
   # Enable query logging
   DB::listen(function ($query) {
       \Log::info($query->sql, $query->bindings);
   });
   ```

4. **GitHub Issues:**
   - Search existing issues
   - Provide error log and `.env` details (without passwords)
   - Include PHP and database versions

---

## Advanced Configuration

### Caching Setup

```env
CACHE_DRIVER=file
SESSION_DRIVER=file
```

For production:
```env
CACHE_DRIVER=redis
SESSION_DRIVER=redis
```

### Email Configuration

```env
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=465
MAIL_USERNAME=your_username
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@dashboard.com
```

### Rate Limiting

Configure in `config/rate-limit.php`:

```php
'throttle' => 'api:60,1', // 60 requests per minute
```

### CORS Configuration

For API calls from different domains, update `config/cors.php`:

```php
'allowed_origins' => ['*'], // Or specify domains
'allowed_methods' => ['GET', 'POST', 'PUT', 'DELETE'],
'allowed_headers' => ['*'],
```

---

## Production Deployment

Before deploying to production:

1. **Security Checks:**
   ```bash
   # Set secure app key
   php artisan key:generate
   
   # Verify .env has APP_DEBUG=false
   # Change APP_ENV=production
   ```

2. **Database:**
   ```bash
   # Run migrations on production
   php artisan migrate --force
   ```

3. **Optimize:**
   ```bash
   # Clear and cache configuration
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   
   # Optimize autoloader
   composer dump-autoload --optimize
   ```

4. **File Permissions:**
   ```bash
   # Set to web server user
   sudo chown -R www-data:www-data .
   chmod -R 755 storage bootstrap/cache
   ```

---

**Next Steps:**
- Read [ARCHITECTURE.md](./ARCHITECTURE.md) to understand system design
- Check [DATABASE.md](./DATABASE.md) for database structure
- Review code examples in the repository
- Start building features!

---

**[← Back to README](./README.md)**

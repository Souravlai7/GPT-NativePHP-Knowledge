# Deployment And Operations Knowledge

Use this file as deployment, server, Linux, Windows, environment, queue worker, and operations training/reference data.

## Laravel Deployment Checklist

```bash
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan queue:restart
```

Production `.env`:

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://example.com
LOG_LEVEL=warning
```

Writable folders:

```text
storage/
bootstrap/cache/
```

## Zero-Downtime Ideas

- Put app in maintenance mode only if necessary.
- Build assets before switching release.
- Run migrations carefully.
- Avoid destructive migrations during high traffic.
- Restart queue workers after code deploy.
- Keep backups before risky database changes.

## Queue Workers

Run locally:

```bash
php artisan queue:work
```

Restart after deploy:

```bash
php artisan queue:restart
```

Supervisor concept:

```text
Supervisor/systemd keeps queue workers running in production.
Workers should restart automatically after failure.
```

Worker flags:

```bash
php artisan queue:work --tries=3
php artisan queue:work --queue=high,default
php artisan queue:work --timeout=120
```

## Cron Scheduler

Laravel scheduler:

```bash
php artisan schedule:run
```

Crontab:

```cron
* * * * * cd /path/to/app && php artisan schedule:run >> /dev/null 2>&1
```

Scheduled command:

```php
Schedule::command('reports:send')->daily();
Schedule::job(new CleanupTempFiles)->hourly();
```

## Linux Commands

Files:

```bash
ls -la
pwd
cp source dest
mv old new
rm file
mkdir folder
```

Logs:

```bash
tail -f storage/logs/laravel.log
journalctl -u service-name -f
```

Permissions:

```bash
chmod -R ug+rw storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

Processes:

```bash
ps aux
top
htop
kill PID
```

Network:

```bash
curl -I https://example.com
ping example.com
netstat -tulpn
ss -tulpn
```

## Windows Commands

PowerShell:

```powershell
Get-ChildItem
Get-Content .\storage\logs\laravel.log -Tail 100 -Wait
Get-Process
Get-NetTCPConnection -LocalPort 8000
ipconfig
```

Find process using port:

```powershell
Get-NetTCPConnection -LocalPort 8000 | Select-Object OwningProcess
Get-Process -Id <pid>
```

## Environment Variables

Rules:

- `.env` is environment-specific.
- Do not commit `.env`.
- Run `php artisan config:clear` after changing `.env` locally.
- Run `php artisan config:cache` in production.

Common production variables:

```env
APP_ENV=production
APP_DEBUG=false
CACHE_STORE=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
LOG_CHANNEL=stack
```

## Backups

Backup before:

- Large migrations.
- Deleting records.
- Production data imports.
- Changing payment, payroll, or accounting data.

MySQL dump:

```bash
mysqldump -u user -p database_name > backup.sql
```

Restore:

```bash
mysql -u user -p database_name < backup.sql
```

## Monitoring

Watch:

- Error rate.
- Slow requests.
- Queue length.
- Failed jobs.
- Database CPU/load.
- Disk space.
- Memory usage.
- SSL certificate expiry.

Laravel places to inspect:

```text
storage/logs/laravel.log
failed_jobs table
queue worker logs
web server logs
database slow query log
```

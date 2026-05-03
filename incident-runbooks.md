# Incident Runbooks

Use this file to help GPT respond quickly to production and real-user problems.

## Production Site Down

Immediate steps:

1. Confirm from outside the server.
2. Check recent deploys.
3. Check web server/PHP logs.
4. Check Laravel logs.
5. Check database connectivity.
6. Check disk space.
7. Roll back if recent deploy caused outage.

Commands:

```bash
curl -I https://example.com
tail -f storage/logs/laravel.log
df -h
free -m
php artisan about
```

Common causes:

- Syntax/runtime error after deploy.
- `.env` missing or wrong.
- Config cache stale.
- Database down.
- Disk full.
- File permissions wrong.
- PHP extension missing.

## Database Slow Or Down

Immediate checks:

- Is database reachable?
- Is CPU high?
- Are there long-running queries?
- Did a migration or report start?
- Did traffic spike?

MySQL:

```sql
SHOW FULL PROCESSLIST;
SHOW ENGINE INNODB STATUS;
```

Laravel:

```bash
php artisan queue:failed
```

Fix options:

- Kill clearly bad long-running query.
- Disable expensive report route/job.
- Add missing index after testing.
- Move heavy report to queue.
- Restore from backup only when data is damaged.

## Queue Backlog

Symptoms:

- Emails delayed.
- Imports stuck.
- Notifications not sent.
- `jobs` table growing.

Checks:

```bash
php artisan queue:failed
php artisan queue:work --tries=3
```

Fix:

- Start/restart workers.
- Check failed job exception.
- Increase worker count if safe.
- Fix bad job and retry.
- Do not blindly retry all jobs if the bug still exists.

Retry:

```bash
php artisan queue:retry all
```

Flush failed jobs only after deciding they are not needed:

```bash
php artisan queue:flush
```

## Payment Failure

Rules:

- Never charge twice without verifying provider state.
- Use idempotency keys where provider supports them.
- Log provider request ID, transaction ID, order ID.
- Keep payment state machine explicit.
- Reconcile webhook and direct response.

Statuses:

```text
pending
authorized
paid
failed
cancelled
refunded
partially_refunded
```

Debug:

- Check application logs.
- Check payment provider dashboard.
- Check webhook delivery logs.
- Check duplicate requests.
- Check timeout behavior.

## Email Delivery Incident

Checks:

- Is mail queued?
- Is worker running?
- Are SMTP/API credentials valid?
- Is provider rejecting messages?
- Are messages going to spam?
- Is DNS configured: SPF, DKIM, DMARC?

Temporary mitigation:

- Switch to backup provider if configured.
- Retry failed jobs after fixing cause.
- Notify users through in-app notification if email is delayed.

## File Storage Incident

Symptoms:

- Upload fails.
- Images broken.
- Downloads return 404/403.
- Disk full.

Checks:

```bash
php artisan storage:link
df -h
```

Laravel checks:

- Disk configured in `config/filesystems.php`.
- `storage` folder writable.
- Public symlink exists.
- Cloud credentials valid.
- File visibility public/private matches route.

## Security Incident

If secrets are leaked:

1. Revoke/rotate leaked secrets.
2. Remove from Git history only if required, but rotation matters most.
3. Check logs for misuse.
4. Deploy new secrets.
5. Document impact.

If account compromise suspected:

- Force password reset.
- Revoke tokens/sessions.
- Review recent actions.
- Add rate limiting or MFA if missing.

## Bad Deploy

Mitigation:

```bash
php artisan down
git switch previous-release
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan optimize:clear
php artisan up
```

Better production systems use release folders and symlink switching.

After incident:

- Write root cause.
- Add regression test.
- Add monitor/alert if possible.
- Improve deploy checklist.

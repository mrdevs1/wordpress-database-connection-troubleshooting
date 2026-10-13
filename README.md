# WordPress Database Connection Troubleshooting Guide

A practical guide to diagnosing and resolving WordPress database connection failures, MySQL/MariaDB service issues, incorrect credentials, database permission problems, connection limits, and database resource exhaustion.

## Table of Contents

* [Overview](#overview)
* [Common Errors and Symptoms](#common-errors-and-symptoms)
* [1. Diagnose the Error](#1-diagnose-the-error)
* [2. Verify Database Configuration](#2-verify-database-configuration)
* [3. Check MySQL or MariaDB Availability](#3-check-mysql-or-mariadb-availability)
* [4. Verify Database User Permissions](#4-verify-database-user-permissions)
* [5. Investigate Connection Limits](#5-investigate-connection-limits)
* [6. Check Disk Space and Resources](#6-check-disk-space-and-resources)
* [7. Review Error Logs](#7-review-error-logs)
* [8. Check Database Tables](#8-check-database-tables)
* [9. Troubleshoot cPanel Hosting](#9-troubleshoot-cpanel-hosting)
* [10. PHP Database Extension Diagnostics](#10-php-database-extension-diagnostics)
* [11. Safe Recovery Procedure](#11-safe-recovery-procedure)
* [Troubleshooting Checklist](#troubleshooting-checklist)
* [Security Best Practices](#security-best-practices)
* [Official References](#official-references)

## Overview

WordPress relies on a database to store posts, pages, settings, user accounts, and other site information.

A database connection failure can result from incorrect credentials, unavailable database services, insufficient permissions, exhausted resources, or network and socket configuration issues.

The message **"Error Establishing a Database Connection"** does not necessarily mean that the database is corrupted. Identify the actual cause before attempting repairs.

## Common Errors and Symptoms

| Error or symptom                         | Possible cause                                         |
| ---------------------------------------- | ------------------------------------------------------ |
| Error Establishing a Database Connection | Incorrect credentials or unavailable database service  |
| Access denied for user                   | Incorrect password, username, host, or permissions     |
| Unknown database                         | Incorrect database name or missing database            |
| Too many connections                     | Database connection limit reached                      |
| Connection timed out                     | Network, service, or resource issue                    |
| Connection refused                       | Database service not listening at the expected address |
| MySQL server has gone away               | Connection interruption, timeout, or server-side issue |
| SQLSTATE connection error                | PHP database driver or connection configuration issue  |
| Database server unavailable              | Service failure, maintenance, or resource exhaustion   |

These are common possibilities, not definitive diagnoses. Exact causes depend on the error message and hosting environment.

## 1. Diagnose the Error

Before changing any configuration, collect basic information.

1. Record the exact error message.
2. Determine whether the entire website is affected or only the WordPress admin area.
3. Check whether the error began after a password change, migration, plugin update, or server maintenance.
4. Check the hosting provider's service status, if available.
5. Review relevant PHP and database logs.
6. Determine whether other websites using the same database server are affected.

If several unrelated websites fail simultaneously, investigate the shared database service or server resources before changing individual WordPress configurations.

## 2. Verify Database Configuration

WordPress stores its database connection settings in `wp-config.php`, normally located in the WordPress installation directory.

The relevant configuration entries look like this:

```php
define( 'DB_NAME', 'database_name_here' );
define( 'DB_USER', 'database_user_here' );
define( 'DB_PASSWORD', 'replace_with_actual_password' );
define( 'DB_HOST', 'localhost' );
```

These are placeholders. Use the actual values supplied by your hosting environment.

### Verification steps

1. Back up `wp-config.php`.
2. Confirm the database name.
3. Confirm the database username.
4. Verify the password through a trusted configuration source.
5. Confirm the database host and, where applicable, port or socket configuration.
6. Check for accidental whitespace, quoting errors, or duplicate configuration definitions.
7. Test the site again after correcting any verified mismatch.

On cPanel, database names and usernames may include an account-specific prefix. Use the full names displayed in the control panel.

**Important:** Do not publish `wp-config.php`, database credentials, or screenshots containing passwords in a public repository or support ticket.

Official documentation: https://developer.wordpress.org/advanced-administration/wordpress/wp-config/

## 3. Check MySQL or MariaDB Availability

If the credentials appear correct, determine whether the database service is available.

### For authorized server administrators

Check the service status using the command appropriate for the server:

```bash
systemctl status mariadb
```

On systems using the MySQL service name:

```bash
systemctl status mysqld
```

Service names vary by operating system and installation.

If you do not have administrative access, ask your hosting provider to verify database service availability.

### Check the database connection

If you have authorized database credentials, test the connection using the MySQL client:

```bash
mysql -h localhost -u DATABASE_USER -p DATABASE_NAME
```

Replace the placeholders with the correct values. The `-p` option prompts for the password, rather than placing it directly in the command.

If the connection fails, record the exact error without exposing credentials.

A successful command-line connection does not guarantee that the website's PHP environment uses the same host, socket, credentials, or connection settings.

## 4. Verify Database User Permissions

A valid database user must have the appropriate permissions for the database used by WordPress.

### In cPanel

1. Open **MySQL Databases** or the equivalent database management interface.
2. Locate the intended database.
3. Verify that the correct user is assigned to the database.
4. Review the assigned privileges.
5. Confirm that the account has the permissions needed by the site's normal operations.

WordPress generally requires read and write permissions for normal operation, including appropriate permissions for schema changes during legitimate updates.

Do not grant global database privileges or administrative access simply to resolve a connection error. Apply the minimum appropriate permissions to the correct database.

## 5. Investigate Connection Limits

A database may reject new connections when it reaches its configured connection limit or when a hosting account exhausts its resource allowance.

### Symptoms

* Intermittent database connection failures
* "Too many connections" errors
* Errors during traffic spikes
* Multiple sites failing at the same time
* Database connections recovering after traffic decreases

### Diagnostic steps

1. Review the database error log.
2. Check the current connection count and configured limit, if authorized.
3. Inspect PHP-FPM worker usage and application traffic.
4. Look for slow queries or plugins generating excessive database requests.
5. Check whether connections are being closed properly by custom applications.
6. Ask the database administrator or hosting provider to investigate if limits are managed centrally.

Do not repeatedly restart the database service or increase connection limits without identifying the underlying resource constraints.

## 6. Check Disk Space and Resources

Database services can fail when the server runs out of storage or other required resources.

### Check filesystem space

On an authorized Linux server:

```bash
df -h
```

### Check inode usage

```bash
df -i
```

### Check memory availability

```bash
free -h
```

Review the output alongside database and system logs.

Investigate:

* Full database or root filesystem
* Exhausted inodes
* Insufficient memory
* Database temporary-file failures
* Hosting account resource limits
* Filesystem or storage errors

Do not delete database files, binary logs, temporary files, or database directories manually to free space. Use the database administrator's documented maintenance procedures.

## 7. Review Error Logs

Logs often provide the clearest indication of the actual failure.

### WordPress debug log

In a controlled environment, you can enable WordPress debugging in `wp-config.php`:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

If these constants already exist, edit their existing definitions instead of adding duplicates.

The default debug log is usually located at:

```text
wp-content/debug.log
```

Use temporary debugging only when appropriate. Restrict access to logs and disable unnecessary debugging after the investigation.

### Database and server logs

Depending on your environment, inspect:

* MySQL or MariaDB error logs
* PHP or PHP-FPM logs
* Apache or LiteSpeed error logs
* Nginx error logs
* Operating system logs
* cPanel service logs

Correlate entries by timestamp. Avoid sharing logs publicly without removing credentials, tokens, private data, and other sensitive information.

Official debugging documentation: https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/

## 8. Check Database Tables

A connection error is not the same as table corruption. Confirm that WordPress can connect before investigating table integrity.

### Verify the database

Using phpMyAdmin or an authorized database client:

1. Open the correct database.
2. Confirm that the expected WordPress tables exist.
3. Check whether the database contains the expected site data.
4. Review database logs for table or storage-engine errors.
5. Back up the database before performing any repair or maintenance operation.

### WordPress database repair

WordPress has a database repair feature, but it should only be considered after verifying the cause and creating a reliable backup.

Do not assume that enabling database repair will fix incorrect credentials, an unavailable service, or exhausted server resources.

Avoid running repair operations against production tables without understanding the affected storage engine and potential impact.

## 9. Troubleshoot cPanel Hosting

For websites hosted on cPanel, follow this sequence:

1. Open **cPanel → MySQL Databases** and verify the database and assigned user.
2. Check the account's database configuration and quotas.
3. Review **MultiPHP Manager** to identify the PHP version assigned to the domain.
4. Check the relevant PHP error log.
5. Ask the hosting provider to verify MySQL/MariaDB service health if you do not have root access.
6. Check whether other websites on the same server are affected.
7. Re-test the website after the confirmed issue is resolved.

For managed or reseller hosting, service restarts and global database configuration changes should be handled by the provider or authorized administrator.

Do not modify another customer's database or server configuration.

## 10. PHP Database Extension Diagnostics

WordPress needs a supported PHP database driver to communicate with MySQL or MariaDB.

### Check installed PHP extensions

From a terminal, run:

```bash
php -m | grep -Ei '^(mysqli|pdo_mysql)$'
```

The output can help identify whether the command-line PHP environment has the expected extension.

However, command-line PHP may use a different version from the PHP handler assigned to the website.

### Check the website's PHP environment

Use the hosting control panel or an authorized diagnostic method to confirm the active PHP version and its database extensions.

Do not leave a public `phpinfo()` page on a production website. It may expose configuration details useful to attackers.

If the required extension is missing, enable or install the compatible extension through the hosting control panel or ask the server administrator.

PHP reference: https://www.php.net/manual/en/ref.mysqli.php

## 11. Safe Recovery Procedure

Use this sequence when restoring a website affected by a database connection error.

1. **Preserve evidence:** Record the error and relevant log entries.
2. **Back up:** Preserve the database and relevant configuration files before making changes, if possible.
3. **Verify credentials:** Check database name, username, password, and host.
4. **Verify permissions:** Confirm that the database user is assigned correctly.
5. **Check availability:** Confirm the database service is healthy.
6. **Check resources:** Inspect disk space, memory, connection limits, and account restrictions.
7. **Correct the confirmed cause:** Make the smallest appropriate change.
8. **Verify recovery:** Test the homepage, WordPress admin area, and database-dependent features.
9. **Monitor:** Review logs to determine whether the error returns.
10. **Document:** Record the cause, resolution, and preventive action.

If the database is unavailable or corruption is suspected, involve the authorized server administrator before attempting invasive repairs.

## Troubleshooting Checklist

* [ ] Record the exact database error.
* [ ] Determine whether one or multiple websites are affected.
* [ ] Verify database name, username, password, and host.
* [ ] Confirm that the database user has appropriate permissions.
* [ ] Check MySQL/MariaDB availability.
* [ ] Review database and PHP logs.
* [ ] Check disk space, inodes, and memory.
* [ ] Investigate connection limits and resource restrictions.
* [ ] Confirm the PHP database extension is available.
* [ ] Back up before database repair or configuration changes.
* [ ] Test the website after applying the fix.
* [ ] Document the verified root cause.

## Security Best Practices

* Never publish database credentials or `wp-config.php`.
* Use strong, unique database passwords.
* Grant only the privileges required by the application.
* Restrict database access to trusted users and hosts.
* Keep WordPress, PHP, MySQL/MariaDB, and plugins updated.
* Protect diagnostic logs from public access.
* Back up databases before maintenance or repair operations.
* Avoid destructive commands and manual deletion of database files.
* Do not expose detailed database errors to website visitors.
* Make production changes only with appropriate authorization.

## Official References

* [WordPress Common Errors](https://developer.wordpress.org/advanced-administration/wordpress/common-errors/)
* [Editing wp-config.php](https://developer.wordpress.org/advanced-administration/wordpress/wp-config/)
* [WordPress Debugging](https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/)
* [WordPress Installation Guide](https://developer.wordpress.org/advanced-administration/before-install/howto-install/)
* [PHP MySQLi Documentation](https://www.php.net/manual/en/book.mysqli.php)
* [PHP PDO MySQL Driver](https://www.php.net/manual/en/ref.pdo-mysql.php)

## Contributing

Contributions are welcome.

When submitting a fix or improvement:

1. Describe the error and affected environment.
2. Include relevant WordPress, PHP, and database versions.
3. Provide sanitized logs or error messages.
4. Explain how the root cause was verified.
5. Document the solution and its limitations.
6. Never include credentials, customer information, or private production data.

## License

This repository can be distributed under the MIT License. Add a `LICENSE` file containing the MIT License text if you choose this license.

---

**Disclaimer:** This is an independent troubleshooting resource and is not an official WordPress project. Always verify the root cause and back up production data before making potentially disruptive changes.


## Important Log Files
```
| File                  | Purpose                             |
| --------------------- | ----------------------------------- |
| `/var/log/messages`   | Generic system logs                 |
| `/var/log/syslog`     | Generic system activity             |
| `/var/log/auth.log`   | Authentication logs (Debian)        |
| `/var/log/secure`     | Authentication logs (RedHat/CentOS) |
| `/var/log/boot.log`   | Boot information                    |
| `/var/log/dmesg`      | Hardware logs                       |
| `/var/log/kern.log`   | Kernel logs                         |
| `/var/log/faillog`    | Failed logins                       |
| `/var/log/cron`       | Cron jobs                           |
| `/var/log/mail.log`   | Mail server                         |
| `/var/log/httpd`      | Apache                              |
| `/var/log/mysqld.log` | MySQL                               |
```

## NSE Script Categories

| Category      | Purpose                                                                                              |
| ------------- | ---------------------------------------------------------------------------------------------------- |
| **auth**      | Finds or tests authentication credentials.                                                           |
| **broadcast** | Discovers hosts using broadcast traffic and can automatically add discovered hosts to later scans.   |
| **brute**     | Performs brute-force login attempts using credentials.                                               |
| **default**   | Default scripts executed with `-sC`.                                                                 |
| **discovery** | Collects information about accessible services.                                                      |
| **dos**       | Checks for Denial-of-Service (DoS) vulnerabilities. Used less often because it can harm services.    |
| **exploit**   | Attempts to exploit known vulnerabilities.                                                           |
| **external**  | Uses external services for additional processing.                                                    |
| **fuzzer**    | Sends different inputs to discover vulnerabilities or unusual packet handling. Can take a long time. |
| **intrusive** | Runs intrusive scripts that may negatively affect the target.                                        |
| **malware**   | Checks whether the target is infected with malware.                                                  |
| **safe**      | Runs safe, non-intrusive, and non-destructive scripts.                                               |
| **version**   | Extends service version detection.                                                                   |
| **vuln**      | Identifies known vulnerabilities.                                                                    |

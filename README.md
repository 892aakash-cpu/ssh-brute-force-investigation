# SSH Brute Force Investigation (Linux logs)

## 1. Log sample

```text
Nov 24 09:00:12 srv sshd[700]: Accepted publickey for amit from 10.0.0.15 port 50100 ssh2
Nov 24 09:14:01 srv sshd[810]: Failed password for root from 103.77.5.20 port 6001 ssh2
Nov 24 09:14:03 srv sshd[810]: Failed password for root from 103.77.5.20 port 6003 ssh2
Nov 24 09:14:05 srv sshd[810]: Failed password for invalid user dev from 103.77.5.20 port 6005 ssh2
Nov 24 09:14:07 srv sshd[810]: Failed password for admin from 103.77.5.20 port 6007 ssh2
Nov 24 09:14:09 srv sshd[810]: Failed password for root from 103.77.5.20 port 6009 ssh2
Nov 24 09:14:11 srv sshd[810]: Failed password for admin from 103.77.5.20 port 6011 ssh2
Nov 24 09:14:40 srv sshd[815]: Accepted password for admin from 103.77.5.20 port 6100 ssh2
Nov 24 09:15:10 srv sudo: admin : COMMAND=/bin/bash
Nov 24 09:15:50 srv crontab[820]: (root) REPLACE (root)
```

## 2. Commands

The output is from the full log.

### 1. Top IPs (failed logins)

```text
grep "Failed password" log3.log | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -rn
```

Output:

```text
      6 103.77.5.20
      3 62.4.9.8
      1 10.0.0.21
```

This command filters failed password lines from log3.log, extracts the IP addresses, counts each one, and ranks them from highest to lowest.

### 2. Top usernames (failed logins)

```text
grep "Failed password" log3.log | grep -oE 'for (invalid user )?[a-z]+' | awk '{print $NF}' | sort | uniq -c | sort -rn
```

Output:

```text
      3 root
      2 ftp
      2 admin
      1 test
      1 riya
      1 dev
```

This command filters failed password lines, extracts the "for <user>" part, prints the last word of it (the username), then counts and ranks the usernames.

### 3. First failed time of 103.77.5.20

```text
grep "Failed password" log3.log | grep "103.77.5.20" | awk '{print $3}' | head -1
```

Output:

```text
09:14:01
```

This command filters the failed lines for 103.77.5.20, prints the time column, and shows the first line.

### 4. Last failed time of 103.77.5.20

```text
grep "Failed password" log3.log | grep "103.77.5.20" | awk '{print $3}' | tail -1
```

Output:

```text
09:14:11
```

This command filters the failed lines for 103.77.5.20, prints the time column, and shows the last line.

## 3. Finding

103.77.5.20 made 6 failed logins on root, admin and invalid user dev between 09:14:01 and 09:14:11.

It logged in as admin at 09:14:40.

It then ran sudo at 09:15:10 and replaced root's crontab at 09:15:50.

This is a compromise.

Action: recommend blocking the IP, reviewing and removing the crontab entry, resetting the admin and root credentials, and escalating to L2.

Passwords are not visible in the log, so spray versus brute force cannot be confirmed.

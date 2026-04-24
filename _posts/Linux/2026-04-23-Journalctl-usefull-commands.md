---
title: Useful Journalctl Commands
date: 2026-04-23 14:04:00 +0200
categories: [Linux]
tags: [linux, journalctl, systemd, logs]
description: A collection of useful Journalctl commands for managing and analyzing system logs.
---

## Document Information

- **Ubuntu Version:** 24.04 LTS

## Modify configuration

### Open the configuration file

```bash
sudo nano /etc/systemd/journald.conf
```

### Restart the journald service to apply changes

```bash
sudo systemctl restart systemd-journald
```

## View logs

### View logs for a specific service

```bash
journalctl -u webserver
```

### View logs for a specific time range

```bash
journalctl --since "2024-01-01" --until "2024-01-31"
```

### View logs in real-time

```bash
journalctl -f
```

### Search for specific keywords in logs

```bash
journalctl | grep "error"
```

## Analyze logs

### See current log size

```bash
journalctl --disk-usage
```

## Export logs

### Export logs to a file

```bash
sudo touch /home/log.txt
journalctl -u webserver --no-tail | grep "User subscribed" | sudo tee /home/log.txt > /dev/null
```

### Download logs from a remote server

```bash
scp user@server:/home/log.txt C:\Users\YourUsername\Downloads\log.txt
```

## Clear logs

### Clear all logs

```bash
sudo journalctl --vacuum-time=1s
```

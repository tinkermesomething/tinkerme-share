---
id: integrations
---

# Integrations

## ClamAV (Malware Scanning)

ClamAV automatically scans uploaded files for malware and removes infected shares. This is **highly recommended** for public-facing instances.

### How it works

1. When a share is completed, all files are scanned
2. If malware is detected, the entire share is deleted
3. The share creator sees a message explaining why their share was removed

### Requirements

- **RAM**: 1-4GB (ClamAV loads virus definitions into memory)
- **First startup**: 2-5 minutes (downloads latest virus definitions)
- **Disk**: ~500MB for virus definitions

### Docker Setup

The easiest way is to uncomment the ClamAV section in your `docker-compose.yml`:

```yaml
services:
  tinkerme-share:
    image: ghcr.io/tinkermesomething/tinkerme-share
    # ... other config ...
    depends_on:
      clamav:
        condition: service_healthy

  clamav:
    image: clamav/clamav
    restart: unless-stopped
    # Optional: limit memory usage
    # deploy:
    #   resources:
    #     limits:
    #       memory: 2G
```

After starting, check the logs for "ClamAV is active":

```bash
docker compose logs tinkerme-share | grep -i clamav
```

### Using an existing ClamAV instance

If you already have ClamAV running elsewhere (e.g., on your host or another container), point to it with environment variables:

```yaml
services:
  tinkerme-share:
    environment:
      - CLAMAV_HOST=192.168.1.100  # IP or hostname of your ClamAV server
      - CLAMAV_PORT=3310           # Default ClamAV port
```

### Stand-Alone Setup (without Docker)

1. Install ClamAV daemon on your system:
   ```bash
   # Debian/Ubuntu
   apt install clamav-daemon

   # Start the daemon
   systemctl start clamav-daemon
   ```

2. Set environment variables before starting the backend:
   ```bash
   export CLAMAV_HOST=127.0.0.1
   export CLAMAV_PORT=3310
   ```

### Verification

To verify ClamAV is working:

1. Check Tinkerme Share logs for "ClamAV is active"
2. Upload a test file - it should complete without issues
3. (Optional) Test with [EICAR test file](https://www.eicar.org/download-anti-malware-testfile/) - the share should be rejected

### Troubleshooting

| Issue | Solution |
|-------|----------|
| "ClamAV is not active" in logs | ClamAV not reachable - check host/port and that ClamAV is running |
| Container won't start | ClamAV still downloading definitions - wait 2-5 minutes |
| Out of memory errors | Increase container memory limit or host RAM |

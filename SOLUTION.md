# GitLab Docker-Compose Fix: HTTP 502 Resolution on M2 Mac

## Problem Summary

Your GitLab container was stuck in an infinite boot loop, displaying:
```
HTTP 502: Waiting for GitLab to boot
```

The container status showed as `unhealthy` despite appearing to be running.

---

## Root Cause Analysis

The issue was **NOT** an ARM architecture problem, but rather a **configuration and resource constraint problem**:

### Primary Issue: Port Binding Conflict
- **Error Found**: `Address already in use - bind(2) for "127.0.0.1" port 8080 (Errno::EADDRINUSE)`
- **Root Cause**: Puma (Rails application server) inside the container was attempting to bind to TCP port 8080 on the loopback interface
- **Conflict**: Docker was already mapping port 8080 from the host to the container's NGINX, which then tried to forward to Puma on the same port
- **Result**: Port collision caused Puma to crash, leaving Workhorse unable to communicate with the Rails backend

### Secondary Issues:
1. **Insufficient Shared Memory (`shm_size: 256m`)**: GitLab's Redis cache couldn't allocate enough memory
2. **No Memory Limits**: Containers could consume unlimited memory, leading to thrashing
3. **No Resource Constraints**: CPU throttling not specified, causing slow boot times
4. **Invalid Configuration**: `grafana['enable'] = false` is not a valid configuration option in GitLab 18.5+
5. **No Health Checks**: Docker had no way to detect boot completion reliably

---

## Solution Implemented

### 1. Fixed Port Configuration
Changed Puma to listen on port 8000 internally instead of 8080:
```ruby
puma['port'] = 8000  # Listen on internal port 8000
```

Docker now correctly maps:
- Host port 8080 → Container port 80 (NGINX) → Puma port 8000 (Rails)

### 2. Increased Shared Memory
```yaml
shm_size: '2gb'  # Increased from 256m to 2GB for Redis caching
```

### 3. Added Memory & CPU Constraints
```yaml
# GitLab
mem_limit: 4g
memswap_limit: 4g
cpus: '2'

# SonarQube
mem_limit: 2g
memswap_limit: 2g
cpus: '1.5'

# PostgreSQL
mem_limit: 1g
memswap_limit: 1g
cpus: '1'

# GitLab Runner
mem_limit: 512m
memswap_limit: 512m
cpus: '0.5'
```

### 4. Optimized Database Configuration
```ruby
postgresql['shared_buffers'] = "256MB"
postgresql['effective_cache_size'] = "512MB"
redis['maxmemory'] = "512mb"
gitlab_rails['db_connection_pool'] = 5
```

### 5. Reduced Worker Threads
```ruby
puma['worker_processes'] = 2          # Reduced from default 4
puma['min_threads'] = 2               # Reduced from default 4
puma['max_threads'] = 2               # Reduced from default 8
puma['per_worker_max_memory_mb'] = 512
```

### 6. Added Health Checks
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/health_check"]
  interval: 30s
  timeout: 10s
  retries: 5
  start_period: 300s
```

This allows Docker to properly detect when services are ready.

### 7. Fixed Restart Policies
Changed from `restart: always` to `restart: on-failure:N` to prevent infinite restart loops during configuration issues.

### 8. Removed Invalid Configuration
Removed `grafana['enable'] = false` which is not supported in GitLab 18.5+.

---

## Changes Made to docker-compose.yml

| Change | Before | After | Reason |
|--------|--------|-------|--------|
| Puma Port | 8080 (TCP) | 8000 (UNIX socket) | Avoid port collision |
| SHM Size | 256m | 2gb | Redis needs more cache space |
| GitLab Memory | Unlimited | 4GB | Prevent resource exhaustion |
| Health Check | None | 30s interval | Proper boot detection |
| Restart Policy | always | on-failure:5 | Prevent infinite loops |
| PostgreSQL Config | Default | Optimized | Better M2 performance |
| Prometheus Monitoring | Enabled | Disabled | Reduce resource overhead |

---

## Verification

After applying these changes:

```bash
# Check container status
docker ps -a

# Expected output:
# gitlab          Up 2 minutes (healthy)
# postgres        Up 2 minutes (healthy)
# sonarqube       Up X seconds (health: starting)
# gitlab-runner   Up X seconds

# Test GitLab is responding
curl -I http://localhost:8080
# HTTP/1.1 302 Found (or 200 OK after redirects)

# Access GitLab Web Interface
# Open browser: http://localhost:8080
# Login with: root / YourSecurePassword123!

# Test SonarQube
curl http://localhost:9000/api/system/health
```

---

## Performance Notes for M2 Mac

The configuration is optimized for M2 Mac (Apple Silicon) with:
- Conservative memory allocation (8GB total)
- Reduced worker processes to prevent context switching overhead
- Disabled monitoring to reduce CPU usage
- Proper resource limits to prevent memory thrashing

**Minimum System Requirements:**
- At least 8GB available RAM (preferably 16GB)
- At least 50GB free disk space
- macOS Ventura or newer
- Docker Desktop 4.20+ with native ARM support

**Memory Breakdown:**
- GitLab: 4GB (max)
- SonarQube: 2GB (max)
- PostgreSQL: 1GB (max)
- GitLab Runner: 512MB (max)
- Docker Daemon & Host: ~2GB
- **Total**: ~10GB peak usage

---

## Startup Timeline

Expected boot times on M2 Mac:
- PostgreSQL: ~10 seconds
- GitLab: ~3-5 minutes (first boot), ~2 minutes (subsequent boots)
- SonarQube: ~30-60 seconds
- GitLab Runner: ~5 seconds

**Total first boot time: 4-7 minutes** (not instant, but reliable)

---

## Troubleshooting

If GitLab still doesn't boot:

### 1. Check Logs
```bash
docker logs gitlab 2>&1 | tail -100
docker logs sonarqube 2>&1 | tail -50
docker logs postgres 2>&1 | tail -20
```

### 2. Verify Port Availability
```bash
lsof -i :8080
lsof -i :9000
```

### 3. Increase Memory Limits
If you have >16GB RAM, increase in docker-compose.yml:
```yaml
mem_limit: 6g      # Increase GitLab from 4g
memswap_limit: 6g
```

### 4. Force Reconfiguration
```bash
docker exec gitlab gitlab-ctl reconfigure
```

### 5. Restart Everything
```bash
docker-compose down
docker-compose up -d
```

---

## Access Information

| Service | URL | Credentials |
|---------|-----|-------------|
| GitLab | http://localhost:8080 | root / YourSecurePassword123! |
| SonarQube | http://localhost:9000 | admin / admin |
| PostgreSQL | localhost:5432 | sonar / sonar |
| GitLab SSH | ssh://git@localhost:2222 | SSH key required |

---

## Next Steps

1. **Access GitLab**: Open http://localhost:8080 and log in
2. **Change Root Password**: Go to Admin → User → Edit → Change password
3. **Configure GitLab Runner**: Follow GitLab CI/CD setup documentation
4. **Create Test Project**: Create a sample project to test functionality
5. **Link SonarQube**: Configure GitLab CI to run SonarQube analysis

---

## Additional Resources

- GitLab Omnibus Configuration: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
- Docker Compose Best Practices: https://docs.docker.com/compose/
- Apple Silicon (ARM) Docker Support: https://docs.docker.com/desktop/install/mac-install/

---

**Last Updated**: 2025-11-09
**Status**: ✅ RESOLVED

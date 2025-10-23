# NATS JetStream Step-By-Step Setup & Best Practices

NATS is a high-performance messaging system with JetStream for streaming and persistence. This guide covers setting up a 3-node NATS JetStream cluster using Docker with 4MB payload support and high availability.

---

## 1. Install Docker and Docker Compose

**Note:** Skip this step if Docker is already installed.

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

```bash
# Install Docker Compose and plugins
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
# Verify installations
docker --version
docker compose version
```

**Important:** Log out and back in for group changes to take effect.

---

## 2. Create NATS Configuration Directory

```bash
mkdir -p ~/nats-cluster
cd ~/nats-cluster
```

---

## 3. Create NATS Server Configuration

Copy content from `nats-server.conf`:

```bash
nano nats-server.conf
# Copy content from nats-server.conf
```

**Key Configuration Features:**
- Client port: `4222`
- HTTP monitoring: `8222`
- Cluster port: `6222`
- Max payload: `4MB` (default is 1MB)
- JetStream enabled with 1GB memory, 10GB file storage
- 3-node cluster configuration

---

## 4. Create Docker Compose Configuration

Copy content from `docker-compose.yml`:

```bash
nano docker-compose.yml
# Copy content from docker-compose.yml
```

**Cluster Architecture:**
- **nats-1**: Ports 4222 (client), 8222 (monitoring)
- **nats-2**: Ports 4223 (client), 8223 (monitoring)
- **nats-3**: Ports 4224 (client), 8224 (monitoring)
- All nodes share the same configuration file
- Persistent volumes for each node's JetStream data
- Health checks on all nodes

---

## 5. Start the NATS Cluster

Launch all three NATS nodes:

```bash
docker compose up -d
```

### Verify Cluster Status

```bash
# Check container status
docker compose ps
```

### View Logs

```bash
# View all logs
docker compose logs -f

# View specific node logs
docker compose logs -f nats-1
```

---

## 6. Configure Firewall (UFW)

Choose one of the following configurations based on your security requirements:

### Option A: Localhost Only (Most Secure) ⭐ Recommended

**Use when:** All applications run on the same server as NATS.

```bash
# Don't add any UFW rules - NATS will only be accessible via localhost
# Applications connect using: localhost:4222 or 127.0.0.1:4222
```

**Verification:**
```bash
# From server (works ✅)
nats pub test.subject "hello" --server=localhost:4222

# From external machine (blocked ❌)
nats pub test.subject "hello" --server=YOUR_SERVER_IP:4222
```

See `ufw-nats-localhost-only.conf` for details.

---

### Option B: Restricted Access (Production)

**Use when:** Applications on specific trusted servers need to connect to NATS.

Copy commands from `ufw-nats-restricted.conf` and replace IPs:

```bash
# Allow NATS only from specific application server IPs
sudo ufw allow from YOUR_APP_SERVER_IP to any port 4222 proto tcp comment 'NATS Client (App)'

# Allow monitoring from admin IP only (optional)
sudo ufw allow from YOUR_ADMIN_IP to any port 8222 proto tcp comment 'NATS Monitoring'

# Reload firewall
sudo ufw reload
```

---

### Option C: Public Access (Least Secure)

**Use when:** Testing or when NATS needs to be accessible from anywhere.

⚠️ **Not recommended for production without authentication.**

Run commands from `ufw-nats.conf`:

```bash
# Allow NATS from anywhere
sudo ufw allow 4222/tcp comment 'NATS Client Port'

# Allow monitoring from anywhere (optional)
sudo ufw allow 8222/tcp comment 'NATS HTTP Monitoring'

# Reload firewall
sudo ufw reload
```

**Expected output:**
```
20220/tcp                  LIMIT       Anywhere                   # Rate limit SSH connections
443/tcp                    ALLOW       Anywhere                   # HTTPS
4222/tcp                   ALLOW       Anywhere                   # NATS Client Port
8222/tcp                   ALLOW       Anywhere                   # NATS HTTP Monitoring
```

**Note:** Consider adding authentication to NATS configuration for production use.

---

### Verify Firewall Status

```bash
sudo ufw status numbered
```

### Whitelist Localhost (Important)

To prevent UFW from blocking local NATS connections, add localhost whitelist rules.

**Method 1: Simple UFW commands**

```bash
# Allow all traffic from localhost
sudo ufw allow from 127.0.0.1 to any
sudo ufw allow from ::1 to any
```

**Method 2: Add to UFW before.rules** (Recommended for persistent config)

```bash
sudo nano /etc/ufw/before.rules
# Copy rules from ufw-localhost.conf after the initial comment block
```

---

## 7. Configure Fail2ban Whitelist

Prevent Fail2ban from blocking NATS connections from localhost and trusted sources.

### Create NATS Whitelist

```bash
sudo nano /etc/fail2ban/jail.d/nats-whitelist.conf
# Copy content from fail2ban-whitelist.conf
```

**Important:** This ensures:
- Localhost connections to NATS are never banned
- Docker container connections to NATS work properly
- Application server connections are whitelisted

**Optional:** Add your application server IPs to the `ignoreip` list in the file.

### Restart Fail2ban

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status
```

---

## 8. Install NATS CLI

Install the NATS CLI tool for testing and management:

```bash
# Download and install NATS CLI
curl -sf https://binaries.nats.dev/nats-io/natscli/nats@latest | sh
```

```bash
# Move to system path
sudo mv nats /usr/local/bin/
```

```bash
# Verify installation
nats --version
```

---

## 9. Test JetStream Functionality

### Create a Test Stream

```bash
nats stream add TEST_STREAM \
  --subjects="test.>" \
  --storage=file \
  --replicas=3 \
  --retention=limits \
  --max-msgs=-1 \
  --max-age=24h \
  --server=localhost:4222
```

### Verify Stream

```bash
# List all streams
nats stream ls --server=localhost:4222

# Get detailed stream info
nats stream info TEST_STREAM --server=localhost:4222
```

---

## 10. Test 4MB Payload Support

NATS is configured to support 4MB payloads (default is 1MB).

### Create Test File

```bash
dd if=/dev/urandom of=/tmp/test-4mb.bin bs=1M count=4
```

### Test Publishing and Subscribing

**Terminal 1** - Subscribe to messages:

```bash
nats sub test.large --server=localhost:4222
```

**Terminal 2** - Publish large payload:

```bash
nats pub test.large --server=localhost:4222 < /tmp/test-4mb.bin
```

**Expected:** Subscriber receives the 4MB message successfully.

---

## 11. Test High Availability (Cluster Failover)

Test that the cluster continues working when a node fails:

### Check Current Leader

```bash
nats stream info TEST_STREAM
```

**Example output:**
```
Leader: nats-3
Replica: nats-1, current
Replica: nats-2, current
```

### Simulate Node Failure

```bash
# Stop the leader node
docker stop nats-3
```

### Verify Automatic Failover

```bash
nats stream info TEST_STREAM
```

**Example output:**
```
Leader: nats-1 (3.68s)
Replica: nats-2, current, seen 679ms ago
Replica: nats-3, outdated, OFFLINE, seen 3.70s ago, 5 operations behind
```

### Test Message Publishing During Failure

```bash
echo "test message" | nats pub test.hello --server=localhost:4222
```

**Expected:** Message is published successfully despite one node being down.

### Restore Failed Node

```bash
docker start nats-3
```

### Verify Cluster Recovery

```bash
nats stream info TEST_STREAM
```

**Example output:**
```
Leader: nats-1 (1m17s)
Replica: nats-2, current, seen 753ms ago
Replica: nats-3, current, seen 753ms ago
```

---

## 12. Monitor the Cluster

### HTTP Monitoring Endpoints

```bash
# Check server health
curl http://localhost:8222/healthz

# Get server info (connections, messages, etc.)
curl http://localhost:8222/varz

# Get connection info
curl http://localhost:8222/connz

# Get cluster route info
curl http://localhost:8222/routez

# Get JetStream info
curl http://localhost:8222/jsz
```

### Real-time Monitoring

```bash
# Monitor container resource usage
docker stats nats-1 nats-2 nats-3

# View container logs
docker compose logs -f

# Check disk usage
docker system df -v
```

---

## 13. Cluster Management Commands

### Stop and Start Cluster

```bash
# Stop the cluster
docker compose down

# Start the cluster
docker compose up -d

# Restart the cluster
docker compose restart
```

### View Logs

```bash
# View all logs
docker compose logs -f

# View specific node logs
docker compose logs -f nats-1
```

### Data Management

```bash
# Backup JetStream data
mkdir -p backup
docker run --rm -v nats-cluster_nats-1-data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/nats-1-backup.tar.gz -C /data .
```

**⚠️ Warning:** This command will delete all data:

```bash
# Stop and remove volumes (CAUTION: deletes all data)
docker compose down -v
```

---

## Troubleshooting

### Check Cluster Communication

```bash
# Check cluster routes
docker exec nats-1 nats-server --signal routes

# View route logs
docker compose logs nats-1 | grep route
```

### Verify JetStream Status

```bash
# Execute inside container
docker exec -it nats-1 /bin/sh

# Check JetStream configuration
nats-server --help
```

### Check Resource Usage

```bash
# Monitor container resource usage
docker stats nats-1 nats-2 nats-3

# Check disk usage
docker system df -v
```

### Common Issues

**Nodes not connecting:**
- Check that all containers are running: `docker compose ps`
- Verify network connectivity: `docker network inspect nats-cluster_nats-cluster`
- Check logs for errors: `docker compose logs`

**JetStream not working:**
- Verify JetStream is enabled: `curl http://localhost:8222/jsz`
- Check storage permissions: `docker compose logs | grep jetstream`
- Ensure sufficient disk space: `df -h`

**Large messages failing:**
- Verify max_payload is set to 4MB in `nats-server.conf`
- Check client connection is using the correct server
- Review logs for payload size errors

---

## Security & Access Summary

### Access Patterns by Configuration:

| Configuration | Server Access | Docker (same host) | External Apps | Security Level |
|---------------|---------------|-------------------|---------------|----------------|
| **Localhost Only** | ✅ localhost:4222 | ✅ Via host network | ❌ Blocked by UFW | ⭐⭐⭐ High |
| **Restricted IPs** | ✅ localhost:4222 | ✅ Via host network | ✅ Whitelisted IPs only | ⭐⭐ Medium |
| **Public Access** | ✅ localhost:4222 | ✅ Via host network | ✅ Anyone (no auth) | ⭐ Low |

### Connection Examples:

**From applications on the same server:**
```typescript
// Node.js example
import { connect } from 'nats';
const nc = await connect({ servers: 'localhost:4222' });
```

**From Docker containers (same host):**
```yaml
# docker-compose.yml
services:
  app:
    environment:
      - NATS_URL=nats://host.docker.internal:4222  # Linux: use gateway IP
```

**From external servers (if allowed):**
```typescript
// Requires Option B (Restricted) or C (Public)
const nc = await connect({ servers: 'nats://YOUR_SERVER_IP:4222' });
```

---

## Additional Tips

- **Always test failover** before deploying to production
- **Monitor disk usage** - JetStream stores messages on disk
- **Backup data regularly** using the backup command above
- **Use 3+ replicas** for production high availability
- **Secure monitoring endpoints** - restrict access to port 8222 in production
- **Add authentication for public access** - required if using Option C
- **Use TLS encryption** for external connections in production
- For more configuration options, see [NATS Documentation](https://docs.nats.io/)

---

## Useful NATS CLI Commands

```bash
# List all streams
nats stream ls

# Delete a stream
nats stream rm STREAM_NAME

# Create a consumer
nats consumer add STREAM_NAME CONSUMER_NAME

# View server info
nats server info

# Publish a message
nats pub subject.name "message content"

# Subscribe to a subject
nats sub subject.name

# Request-reply pattern
nats req subject.name "request message"
```

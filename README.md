# Redis Documentation

| Author | Created on | Version | Last updated by | Last edited on |
|--------|------------|---------|-----------------|----------------|
| Meenu Chauhan | 15-01-2025 | Version 1.0 | Meenu Chauhan | 15-01-2025 |

## Introduction

This document provides comprehensive documentation for Redis, an open-source, in-memory data structure store that can be used as a database, cache, and message broker. Redis supports various data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries, and streams.

## Application Flow Diagram

### Basic Redis Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Client    │───▶│   Web Server    │───▶│   Redis Server  │
│   (Browser)     │    │   (Application) │    │   (Cache/DB)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Database      │
                       │   (MySQL/PostgreSQL) │
                       └─────────────────┘
```

### Caching Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Request  │───▶│   Application   │───▶│   Redis Cache   │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Cache Hit?    │    │   Return Data   │
                       │   (Yes/No)      │    │   to Client     │
                       └─────────────────┘    └─────────────────┘
                                │
                                ▼ (No)
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Database      │───▶│   Store in      │
                       │   Query         │    │   Redis Cache   │
                       └─────────────────┘    └─────────────────┘
```

### Session Management Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Login    │───▶│   Authentication│───▶│   Store Session │
│                 │    │   Service       │    │   in Redis      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Return        │
                       │   Session ID    │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Subsequent    │───▶│   Validate      │
                       │   Requests      │    │   Session in    │
                       └─────────────────┘    │   Redis         │
                                              └─────────────────┘
```

### Pub/Sub Message Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Publisher     │───▶│   Redis Server  │───▶│   Subscriber 1  │
│   (Producer)    │    │   (Message      │    │   (Consumer)    │
│                 │    │   Broker)       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Subscriber 2  │
                       │   (Consumer)    │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Subscriber 3  │
                       │   (Consumer)    │
                       └─────────────────┘
```

### High Availability Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │───▶│   Redis Master  │───▶│   Redis Slave 1 │
│   (HAProxy)     │    │   (Primary)     │    │   (Replica)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Redis Slave 2 │
                       │   (Replica)     │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Redis Slave 3 │
                       │   (Replica)     │
                       └─────────────────┘
```

### Data Flow in Redis Cluster

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client 1      │───▶│   Redis Node 1  │───▶│   Hash Slot 0-5460 │
│                 │    │   (Master)      │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Redis Node 2  │
                       │   (Master)      │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Redis Node 3  │
                       │   (Master)      │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Client 2      │───▶│   Hash Slot 5461-10922 │
                       │                 │    │                 │
                       └─────────────────┘    └─────────────────┘
```

### Complete Application Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Client    │───▶│   Load Balancer │───▶│   Web Server 1  │
│   (Browser)     │    │   (Nginx)       │    │   (Application) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Web Server 2  │    │   Redis Cache   │
                       │   (Application) │    │   (Session/Data)│
                       └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Database      │    │   Redis Queue   │
                       │   (MySQL)       │    │   (Jobs/Tasks)  │
                       └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Background    │
                       │   Workers       │
                       └─────────────────┘
```

### Data Persistence Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Write Request │───▶│   Redis Memory  │───▶│   AOF Log       │
│                 │    │   (RAM)         │    │   (Append-Only) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   RDB Snapshot  │
                       │   (Periodic)    │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Disk Storage  │
                       │   (Backup)      │
                       └─────────────────┘
```

### Monitoring and Alerting Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Redis Server  │───▶│   Monitoring    │───▶│   Alert System  │
│   (Metrics)     │    │   (Prometheus)  │    │   (Email/SMS)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Dashboard     │
                       │   (Grafana)     │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Log Analysis  │
                       │   (ELK Stack)   │
                       └─────────────────┘
```

### Key Components Explanation:

1. **Web Client**: Browser or mobile app making requests
2. **Load Balancer**: Distributes traffic across multiple servers
3. **Web Server**: Application server (Node.js, Python, Java, etc.)
4. **Redis Cache**: In-memory data store for fast access
5. **Database**: Persistent storage (MySQL, PostgreSQL, etc.)
6. **Redis Queue**: Message queue for background jobs
7. **Background Workers**: Process jobs from the queue
8. **Monitoring**: Track performance and health metrics

### Data Flow Patterns:

- **Cache-Aside**: Application checks cache first, then database
- **Write-Through**: Data written to both cache and database
- **Write-Behind**: Data written to cache, database updated later
- **Refresh-Ahead**: Cache refreshed before expiration
- **Pub/Sub**: Publishers send messages to subscribers via Redis

## Purposes

- **Caching**: Store frequently accessed data in memory for faster retrieval
- **Session Management**: Store user sessions and authentication tokens
- **Real-time Analytics**: Process and store real-time data streams
- **Message Broker**: Handle pub/sub messaging patterns
- **Database**: Primary or secondary database for applications
- **Rate Limiting**: Implement API rate limiting and throttling
- **Leaderboards**: Store and manage gaming or application rankings
- **Job Queues**: Manage background job processing

## Key Features

- **In-Memory Storage**: Ultra-fast data access with sub-millisecond response times
- **Data Structures**: Support for strings, hashes, lists, sets, sorted sets, and more
- **Persistence**: Optional disk persistence with RDB snapshots and AOF logging
- **Replication**: Master-slave replication for high availability
- **Clustering**: Automatic sharding across multiple nodes
- **Pub/Sub**: Real-time messaging capabilities
- **Lua Scripting**: Server-side scripting for complex operations
- **Transactions**: Atomic operations with MULTI/EXEC
- **Expiration**: Automatic key expiration with TTL support
- **Security**: Authentication and SSL/TLS encryption

## Getting Started

### Pre-requisites

| License Type | Description | Commercial Use | Open Source |
|--------------|-------------|----------------|-------------|
| BSD 3-Clause | Free and open for public use and modification | Yes | Yes |

### Software Overview

| Software | Version |
|----------|---------|
| Redis | 7.2.0 |

### System Requirements

| Requirement | Minimum | Recommendation |
|-------------|---------|----------------|
| Processor/Instance Type | Single-Core/t3.micro instance | Dual-Core/t3.medium instance |
| RAM | 1 Gigabyte | 4 Gigabyte or Higher |
| ROM(Disk Space) | 2 Gigabyte | 10 Gigabyte or Higher |
| OS Required | Linux (Ubuntu 20.04+, CentOS 7+) | Linux (Ubuntu 22.04 LTS) |

### Important Ports

| Ports | Description |
|-------|-------------|
| 6379 | Default Redis server port for client connections |
| 16379 | Redis Cluster bus port (default + 10000) |
| 22 | SSH port for server administration |

### Dependencies

#### Run-time Dependency

| Run-time Dependency | Version | Description |
|---------------------|---------|-------------|
| GCC | 4.9+ | C compiler for building Redis |
| Make | 3.81+ | Build automation tool |
| TCL | 8.5+ | Testing framework for Redis |

#### Other Dependency

| Other Dependency | Version | Description |
|------------------|---------|-------------|
| Systemd | 230+ | Service management (optional) |
| Logrotate | 3.8+ | Log rotation (optional) |

## How to Setup/Install Redis

### Step-by-step Installation Instruction

#### Method 1: Package Manager Installation (Ubuntu/Debian)

```bash
# Update package list
sudo apt update

# Install Redis
sudo apt install redis-server

# Start Redis service
sudo systemctl start redis-server

# Enable Redis to start on boot
sudo systemctl enable redis-server
```

#### Method 2: Source Code Installation

```bash
# Download Redis source
wget https://download.redis.io/redis-stable.tar.gz

# Extract the archive
tar xzf redis-stable.tar.gz

# Navigate to Redis directory
cd redis-stable

# Compile Redis
make

# Install Redis
sudo make install

# Create Redis configuration directory
sudo mkdir /etc/redis

# Copy configuration file
sudo cp redis.conf /etc/redis/

# Create Redis data directory
sudo mkdir /var/lib/redis

# Create Redis user
sudo adduser --system --group --no-create-home redis

# Set permissions
sudo chown redis:redis /var/lib/redis
sudo chmod 770 /var/lib/redis
```

### Run command to start software

```bash
# Start Redis server
redis-server

# Start Redis with configuration file
redis-server /etc/redis/redis.conf

# Start Redis as a service
sudo systemctl start redis-server

# Check Redis status
sudo systemctl status redis-server
```

## Configuration

### Basic Configuration

Edit `/etc/redis/redis.conf`:

```bash
# Network configuration
bind 127.0.0.1  # Bind to localhost only
port 6379       # Default Redis port

# Memory configuration
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence configuration
save 900 1      # Save if at least 1 key changed in 900 seconds
save 300 10     # Save if at least 10 keys changed in 300 seconds
save 60 10000   # Save if at least 10000 keys changed in 60 seconds

# Security configuration
requirepass your_strong_password

# Logging configuration
loglevel notice
logfile /var/log/redis/redis-server.log
```

### Advanced Configuration

```bash
# Enable AOF persistence
appendonly yes
appendfsync everysec

# Configure replication
replicaof <master-ip> <master-port>

# Configure clustering
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

## Maintenance

### Update Commands

```bash
# Update Redis package
sudo apt update && sudo apt upgrade redis-server

# Restart Redis service
sudo systemctl restart redis-server

# Reload configuration
sudo systemctl reload redis-server
```

### Upgrade Software Version

```bash
# Stop Redis service
sudo systemctl stop redis-server

# Backup data
sudo cp -r /var/lib/redis /var/lib/redis.backup

# Install new version
sudo apt update && sudo apt install redis-server

# Start Redis service
sudo systemctl start redis-server

# Verify version
redis-server --version
```

### Restart Commands

```bash
# Restart Redis service
sudo systemctl restart redis-server

# Restart with configuration changes
sudo systemctl reload redis-server

# Force restart
sudo systemctl stop redis-server && sudo systemctl start redis-server
```

## Monitoring

### Check Service Status

```bash
# Check if Redis is running
sudo systemctl status redis-server

# Check Redis process
ps aux | grep redis

# Check Redis port
netstat -tlnp | grep 6379
```

### Redis CLI Commands

```bash
# Connect to Redis
redis-cli

# Test Redis connection
redis-cli ping

# Check Redis info
redis-cli info

# Monitor Redis commands
redis-cli monitor

# Check memory usage
redis-cli info memory

# Check connected clients
redis-cli client list
```

### Log Files

```bash
# View Redis logs
sudo tail -f /var/log/redis/redis-server.log

# Check system logs
sudo journalctl -u redis-server -f

# View error logs
sudo grep ERROR /var/log/redis/redis-server.log
```

## Disaster Recovery

### What is Disaster Recovery?

Disaster Recovery (DR) is a set of procedures and tools to recover your Redis data and service in case of:
- Server crashes
- Data corruption
- Hardware failures
- Accidental data deletion
- Natural disasters

### Backup Strategies

#### 1. Automatic RDB Backup (Recommended for Beginners)

RDB (Redis Database) creates point-in-time snapshots of your data.

```bash
# Create an immediate RDB backup (runs in background)
redis-cli BGSAVE

# Check if backup is complete
redis-cli info persistence

# Verify backup file exists
ls -la /var/lib/redis/dump.rdb
```

**What this does:**
- Saves your current Redis data to `/var/lib/redis/dump.rdb`
- Runs in background (doesn't block Redis)
- Creates a complete snapshot of all data

#### 2. AOF Backup (Append-Only File)

AOF logs every write operation for better data recovery.

```bash
# Rewrite AOF file to optimize size
redis-cli BGREWRITEAOF

# Check AOF file
ls -la /var/lib/redis/appendonly.aof
```

**What this does:**
- Optimizes the AOF file size
- Removes redundant operations
- Maintains data integrity

#### 3. Manual Backup (For Important Data)

Create manual backups with timestamps for important data.

```bash
# Create backup directory (if it doesn't exist)
sudo mkdir -p /backup/redis

# Create timestamped backup
sudo cp /var/lib/redis/dump.rdb /backup/redis/redis-$(date +%Y%m%d-%H%M%S).rdb

# List all backups
ls -la /backup/redis/
```

**What this does:**
- Creates a copy with date and time in filename
- Stores backup in `/backup/redis/` directory
- Allows you to keep multiple backup versions

#### 4. Automated Backup Script (Advanced)

Create a script for automatic daily backups.

```bash
# Create backup script
sudo nano /usr/local/bin/redis-backup.sh
```

**Add this content to the script:**
```bash
#!/bin/bash
# Redis Backup Script

BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d-%H%M%S)
REDIS_DATA="/var/lib/redis/dump.rdb"

# Create backup directory
mkdir -p $BACKUP_DIR

# Create RDB backup
redis-cli BGSAVE

# Wait for backup to complete
sleep 10

# Copy backup file
cp $REDIS_DATA $BACKUP_DIR/redis-$DATE.rdb

# Keep only last 7 days of backups
find $BACKUP_DIR -name "redis-*.rdb" -mtime +7 -delete

echo "Backup completed: redis-$DATE.rdb"
```

**Make script executable:**
```bash
sudo chmod +x /usr/local/bin/redis-backup.sh

# Test the script
sudo /usr/local/bin/redis-backup.sh
```

**Schedule daily backups:**
```bash
# Add to crontab for daily backups at 2 AM
sudo crontab -e

# Add this line:
0 2 * * * /usr/local/bin/redis-backup.sh
```

### Recovery Procedures

#### Step 1: Prepare for Recovery

```bash
# Check current Redis status
sudo systemctl status redis-server

# List available backups
ls -la /backup/redis/

# Check backup file size (should not be 0)
ls -lh /backup/redis/redis-20250115-143022.rdb
```

#### Step 2: Stop Redis Service

```bash
# Stop Redis safely
sudo systemctl stop redis-server

# Verify Redis is stopped
sudo systemctl status redis-server
```

#### Step 3: Backup Current Data (Safety First)

```bash
# Backup current data before recovery
sudo cp /var/lib/redis/dump.rdb /var/lib/redis/dump.rdb.before-recovery

# Also backup AOF if it exists
sudo cp /var/lib/redis/appendonly.aof /var/lib/redis/appendonly.aof.before-recovery
```

#### Step 4: Restore from Backup

```bash
# Copy your backup file to Redis data directory
sudo cp /backup/redis/redis-20250115-143022.rdb /var/lib/redis/dump.rdb

# Set proper ownership and permissions
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo chmod 660 /var/lib/redis/dump.rdb
```

#### Step 5: Start Redis and Verify

```bash
# Start Redis service
sudo systemctl start redis-server

# Check if Redis started successfully
sudo systemctl status redis-server

# Test Redis connection
redis-cli ping

# Check data integrity
redis-cli info keyspace
```

#### Step 6: Verify Data Recovery

```bash
# Connect to Redis
redis-cli

# Check number of keys
dbsize

# List some keys to verify data
keys *

# Test a specific key (replace 'your-key' with actual key)
get your-key

# Exit Redis CLI
exit
```

### Recovery from Different Scenarios

#### Scenario 1: Complete Server Failure

```bash
# On new server, install Redis first
sudo apt update && sudo apt install redis-server

# Stop Redis
sudo systemctl stop redis-server

# Copy backup to new server
scp /backup/redis/redis-backup.rdb user@new-server:/tmp/

# On new server, restore backup
sudo cp /tmp/redis-backup.rdb /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo systemctl start redis-server
```

#### Scenario 2: Data Corruption

```bash
# Stop Redis
sudo systemctl stop redis-server

# Check for corrupted files
redis-check-rdb /var/lib/redis/dump.rdb

# If corrupted, restore from backup
sudo cp /backup/redis/redis-latest.rdb /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo systemctl start redis-server
```

#### Scenario 3: Accidental Data Deletion

```bash
# Stop Redis immediately to prevent overwriting
sudo systemctl stop redis-server

# Restore from most recent backup
sudo cp /backup/redis/redis-$(ls -t /backup/redis/ | head -1) /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo systemctl start redis-server
```

### Best Practices

#### 1. Regular Backups
- **Daily backups**: Schedule automatic daily backups
- **Before updates**: Always backup before system updates
- **Before major changes**: Backup before configuration changes

#### 2. Backup Storage
- **Multiple locations**: Store backups on different servers/disks
- **Off-site storage**: Keep backups in different physical locations
- **Cloud storage**: Use cloud services for additional safety

#### 3. Testing Recovery
- **Monthly testing**: Test recovery procedures monthly
- **Documentation**: Keep detailed recovery procedures
- **Team training**: Train team members on recovery procedures

#### 4. Monitoring and Alerts
- **Backup monitoring**: Set up alerts for backup failures
- **Disk space**: Monitor backup storage space
- **Backup verification**: Verify backup integrity regularly

#### 5. Security
- **Encrypt backups**: Encrypt sensitive backup data
- **Access control**: Limit access to backup files
- **Audit logs**: Keep logs of backup and recovery operations

### Troubleshooting Recovery Issues

#### Issue: Backup file is empty (0 bytes)
```bash
# Check if Redis has data
redis-cli dbsize

# Check Redis configuration
redis-cli config get save

# Force a backup
redis-cli BGSAVE
```

#### Issue: Permission denied during recovery
```bash
# Check file permissions
ls -la /var/lib/redis/dump.rdb

# Fix permissions
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo chmod 660 /var/lib/redis/dump.rdb
```

#### Issue: Redis won't start after recovery
```bash
# Check Redis logs
sudo tail -f /var/log/redis/redis-server.log

# Check configuration
redis-server /etc/redis/redis.conf --test

# Try starting with default config
redis-server --daemonize yes
```

This comprehensive disaster recovery guide provides step-by-step instructions that even beginners can follow to protect and recover their Redis data.

## High Availability

### Replication Setup

```bash
# On master server (redis.conf)
bind 0.0.0.0
port 6379

# On slave server (redis.conf)
bind 0.0.0.0
port 6379
replicaof <master-ip> 6379
```

### Sentinel Configuration

```bash
# Create sentinel configuration
sudo nano /etc/redis/sentinel.conf

# Add configuration
port 26379
sentinel monitor mymaster <master-ip> 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
```

### Cluster Setup

```bash
# Create cluster configuration
redis-cli --cluster create <node1-ip>:6379 <node2-ip>:6379 <node3-ip>:6379 \
  <node4-ip>:6379 <node5-ip>:6379 <node6-ip>:6379 --cluster-replicas 1
```

### Load Balancing

```bash
# Using HAProxy configuration
global
    daemon

defaults
    mode tcp
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend redis_frontend
    bind *:6379
    default_backend redis_backend

backend redis_backend
    balance roundrobin
    server redis1 <redis1-ip>:6379 check
    server redis2 <redis2-ip>:6379 check
```

## Troubleshooting

### Common Issues

#### 1. Redis Won't Start

**Issue**: Redis service fails to start
```bash
# Check error logs
sudo tail -f /var/log/redis/redis-server.log

# Check configuration syntax
redis-server /etc/redis/redis.conf --test
```

**Solution**: Verify configuration file syntax and permissions

#### 2. Memory Issues

**Issue**: Redis runs out of memory
```bash
# Check memory usage
redis-cli info memory

# Check memory policy
redis-cli config get maxmemory-policy
```

**Solution**: Adjust `maxmemory` and `maxmemory-policy` settings

#### 3. Connection Refused

**Issue**: Cannot connect to Redis
```bash
# Check if Redis is running
sudo systemctl status redis-server

# Check port binding
netstat -tlnp | grep 6379

# Check firewall
sudo ufw status
```

**Solution**: Ensure Redis is running and port 6379 is accessible

#### 4. Slow Performance

**Issue**: Redis response times are slow
```bash
# Check Redis info
redis-cli info stats

# Monitor commands
redis-cli monitor

# Check memory fragmentation
redis-cli info memory
```

**Solution**: Optimize configuration, check for memory fragmentation

#### 5. Authentication Errors

**Issue**: Authentication failed
```bash
# Check password configuration
redis-cli config get requirepass

# Connect with password
redis-cli -a your_password
```

**Solution**: Verify password configuration and use correct authentication

### Performance Optimization

```bash
# Enable persistence optimization
save 900 1
save 300 10
save 60 10000

# Optimize memory usage
maxmemory 256mb
maxmemory-policy allkeys-lru

# Enable compression
rdbcompression yes
```

### Security Hardening

```bash
# Bind to specific interface
bind 127.0.0.1

# Set strong password
requirepass your_strong_password

# Disable dangerous commands
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG ""

# Enable protected mode
protected-mode yes
```

This comprehensive Redis documentation provides all the necessary information for installation, configuration, maintenance, and troubleshooting of Redis in production environments.

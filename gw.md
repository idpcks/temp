mkdir -p ~/gateway/docker/postgres/primary ~/gateway/docker/postgres/replica

cat > ~/gateway/docker/postgres/primary/postgresql.conf << 'EOF'
listen_addresses = '*'
port = 5432
max_connections = 100
shared_buffers = 256MB
effective_cache_size = 1GB
work_mem = 4MB
log_min_duration_statement = 1000

# Replication
wal_level = replica
max_wal_senders = 5
max_replication_slots = 5
hot_standby = on
EOF

cat > ~/gateway/docker/postgres/primary/pg_hba.conf << 'EOF'
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             172.30.0.0/24           scram-sha-256
host    replication     replicator      172.30.0.0/24           scram-sha-256
EOF

cat > ~/gateway/docker/postgres/primary/init-replication.sh << 'EOF'
#!/bin/bash
set -e
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD '$REPLICATION_PASSWORD';
    SELECT pg_create_physical_replication_slot('replication_slot');
EOSQL
EOF

chmod +x ~/gateway/docker/postgres/primary/init-replication.sh

echo "✅ Done:"
find ~/gateway/docker -type f

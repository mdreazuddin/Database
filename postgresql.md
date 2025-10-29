## PostgreSQL Cluster

### Manual step-by-step setup for a 3-node PostgreSQL 13 cluster (Master + 2 Standbys, with cascading replication) on Ubuntu 22.04.

**Cluster Overview**
- **Master Node:** Postgresql-One **IP:** 192.168.68.128
- **Standby 1 Node: Direct replica of master** Postgresql-Two **IP** 192.168.68.129
- **Standby 2 Node: Cascading replica (replicates from Standby 1)** Postgresql-Three **IP** 192.168.68.130


### Step 1 — Install PostgreSQL 13 on all three nodes
> Below commands for all 3 Nodes.
```
sudo apt update
sudo apt install wget gnupg2 -y          # Install wget and gnupg2 (needed to add PostgreSQL official repository)

# Add the official PostgreSQL repository signing key
wget -qO - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Add PostgreSQL APT repository for Ubuntu 22.04
echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

sudo apt update
sudo apt install postgresql-13 postgresql-client-13 -y       # Install PostgreSQL 13 and client tools
sudo systemctl enable postgresql                             # Enable PostgreSQL service to start on boot
sudo systemctl start postgresql                              # Start the PostgreSQL service now
```

> Check the service:
`sudo systemctl status postgresql`

### Step 2 — Configure Primary Node (Master) Postgresql-One (192.168.68.128)

- Edit PostgreSQL configuration file

`sudo vim /etc/postgresql/13/main/postgresql.conf`

- Find the below lines and update/add these lines:

```
listen_addresses = '*'      # Allow PostgreSQL to listen on all IPs (needed for replication)

wal_level = replica         # Enable write-ahead log data to be sent to replicas

max_wal_senders = 10        # Maximum number of concurrent replication connections

wal_keep_size = 64          # Amount of WAL logs to keep for replicas (in MB)

archive_mode = on           # Enable archiving for backup or log shipping (optional but useful)

archive_command = 'cp %p /var/lib/postgresql/13/main/archive/%f'        # Command used to archive WAL files (they will be copied to archive folder)

hot_standby = on          # Allow read queries on standby servers
```
- Create the archive folder:

`sudo mkdir /var/lib/postgresql/13/main/archive` # Create directory to store archived WAL files

`sudo chown postgres:postgres /var/lib/postgresql/13/main/archive` # Change ownership to postgres user

- Allow replication connections from standby nodes

`sudo vim /etc/postgresql/13/main/pg_hba.conf`        # Edit the host-based authentication file

- Add these lines at the end:
```
# Allow replication user to connect from standby servers
host replication replicator 192.168.68.129/32 md5
host replication replicator 192.168.68.130/32 md5
```

**Create replication user**

`sudo -u postgres psql`           # Switch to postgres user and open PostgreSQL shell

- Inside PostgreSQL prompt, run: Create a dedicated replication user
```
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'replicator@123';
\q
```
- Restart PostgreSQL service to apply changes:
`sudo systemctl restart postgresql`

### Step 3 — Configure Standby 1 (Replica 1) Postgresql-Two (192.168.68.129)

- Stop PostgreSQL service

`sudo systemctl stop postgresql`        # Stop the PostgreSQL service before taking backup

- Remove existing data directory

`sudo rm -rf /var/lib/postgresql/13/main/*`     # Remove all existing data (will be replaced with master's copy)

- Take base backup from master
```
# -h : master IP
# -D : destination directory
# -U : replication user
# -P : show progress
# -R : automatically create replication config files
sudo -u postgres pg_basebackup -h 192.168.68.128 -D /var/lib/postgresql/13/main -U replicator -P -R
```

It will ask for password: `replicator@123`


- Verify replication settings
`sudo vim /var/lib/postgresql/13/main/postgresql.auto.conf`          # Check the generated replication config file


**Should have the info:** `primary_conninfo = 'host=192.168.68.128 port=5432 user=replicator password=replicator@123'`

- Start PostgreSQL: `sudo systemctl start postgresql`


### Step 4 — Configure Standby 2 (Cascade Replica) Postgresql-Three (192.168.68.130)

- Stop PostgreSQL service `sudo systemctl stop postgresql`
- Remove existing data directory `sudo rm -rf /var/lib/postgresql/13/main/*`

**Take base backup from Standby 1 (not directly from master)**

`sudo -u postgres pg_basebackup -h 192.168.68.129 -D /var/lib/postgresql/13/main -U replicator -P -R`      # This enables cascading replication — standby2 replicates from standby1

- Verify replication settings
`sudo vim /var/lib/postgresql/13/main/postgresql.auto.conf`

- there data will show: `primary_conninfo = 'host=192.168.68.129 port=5432 user=replicator password=replicator@123'`

- Start PostgreSQL: `sudo systemctl start postgresql`

### Step 5 — Verify replication status

- Master Node On Postgresql-One (192.168.68.128):

`sudo -u postgres psql -c "SELECT pg_is_in_recovery();"`

output:

<img width="1286" height="220" alt="image" src="https://github.com/user-attachments/assets/2f2ee136-e494-4874-9535-e83fe4a0e178" />


`sudo -u postgres psql -c "SELECT client_addr, state FROM pg_stat_replication;"`     # Check streaming replication status

output:

<img width="2257" height="244" alt="image" src="https://github.com/user-attachments/assets/ed59fdfc-8f94-4916-99ce-8c252974a3b7" />


- Verify replication connection on the Replica nodes: Execute below command on node 2 & 3

 `sudo -u postgres psql -x -c "SELECT * FROM pg_stat_wal_receiver;"`

 and

 `sudo -u postgres psql -c "SELECT pid, client_addr, state, sync_state, write_lag, flush_lag, replay_lag FROM pg_stat_replication;"`

 Outpout will be as below:

 <img width="2272" height="249" alt="image" src="https://github.com/user-attachments/assets/877771b9-acb7-4c9c-ad78-b04968c36bde" />


<img width="2273" height="749" alt="image" src="https://github.com/user-attachments/assets/51841fd3-fee8-4b84-9ff4-d4209ec91b2e" />

# PostgreSQL Docker Development Environment

This repository contains a Docker-based PostgreSQL development environment, perfect for local development on macOS. It provides a consistent and isolated database environment that's easy to set up and tear down.

## Project Overview

This setup provides a PostgreSQL 17.2 instance running in a Docker container, configured with:
- Persistent data storage using Docker volumes
- Standard port mapping (5432)
- Default database creation
- Customizable environment variables

## Prerequisites

Before you begin, ensure you have the following installed on your macOS:
- [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
- Terminal access

## Installation Steps

1. Clone this repository:
```bash
git clone <repository-url>
cd docker-postgres
```

2. Start the PostgreSQL container:
```bash
docker-compose up -d
```

3. Verify the container is running:
```bash
docker ps --filter "name=dev-postgres"
```

## Connection Details

Use these connection details in your applications:

```javascript
{
host: 'localhost',
port: 5432,
database: 'mydatabase',
user: 'postgres',
password: 'postgrespw'
}
```

## Usage Instructions

### Start the Container
```bash
docker-compose up -d
```

### Stop the Container
```bash
docker-compose down
```

### Access PostgreSQL CLI
```bash
# Connect from inside the container (recommended for first-time testing)
docker exec -it dev-postgres psql -U postgres -d mydatabase

# Connect from your local machine
PGPASSWORD=postgrespw psql -h localhost -U postgres -d mydatabase
```

## Verification Steps

You can verify your installation by running this test command:

```bash
docker exec -it dev-postgres psql -U postgres -d mydatabase -c "
CREATE TABLE IF NOT EXISTS test_table (id SERIAL PRIMARY KEY, message TEXT);
INSERT INTO test_table (message) VALUES ('Hello from CLI test');
SELECT * FROM test_table;"
```

Expected output should show:
- Table creation confirmation
- 1 row inserted
- Query result showing the inserted message

## Connecting from Different Clients

### Node.js (using 'pg' package)
```javascript
const { Pool } = require('pg')
const pool = new Pool({
host: 'localhost',
port: 5432,
database: 'mydatabase',
user: 'postgres',
password: 'postgrespw'
})
```

### Using psql CLI
```bash
PGPASSWORD=postgrespw psql -h localhost -U postgres -d mydatabase
```

## Troubleshooting

1. If you can't connect from your local machine but the container is running:
- Verify Docker Desktop is running
- Check if the container is healthy: `docker ps`
- Ensure no other services are using port 5432

2. To view container logs:
```bash
docker logs dev-postgres
```

## Data Persistence

Data is persisted through Docker volumes. To completely reset the database:
```bash
docker-compose down -v
docker-compose up -d
```


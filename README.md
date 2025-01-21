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

## Using with Node.js

This section guides you through setting up and connecting your Node.js application to the PostgreSQL container.

### Setting Up a New Node.js Project

1. Initialize a new Node.js project:
```bash
mkdir my-postgres-app
cd my-postgres-app
npm init -y
```

### PostgreSQL Client Options

#### Option 1: node-postgres (pg)
```bash
npm install pg
```
```javascript
const { Pool } = require('pg');
const pool = new Pool({
host: 'localhost',
port: 5432,
database: 'mydatabase',
user: 'postgres',
password: 'postgrespw'
});

// Example query
async function testConnection() {
const client = await pool.connect();
try {
    const result = await client.query('SELECT NOW()');
    console.log('Connection successful:', result.rows[0]);
} finally {
    client.release();
}
}
```

#### Option 2: Sequelize ORM
```bash
npm install sequelize pg pg-hstore
```
```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('mydatabase', 'postgres', 'postgrespw', {
host: 'localhost',
dialect: 'postgres',
port: 5432
});

// Test connection
async function testSequelize() {
try {
    await sequelize.authenticate();
    console.log('Connection successful');
} catch (error) {
    console.error('Unable to connect:', error);
}
}
```

#### Option 3: Prisma ORM
```bash
npm install @prisma/client prisma
npx prisma init
```

Configure your `.env` file:
```
DATABASE_URL="postgresql://postgres:postgrespw@localhost:5432/mydatabase"
```

Example schema.prisma:
```prisma
datasource db {
provider = "postgresql"
url      = env("DATABASE_URL")
}

model User {
id    Int     @id @default(autoincrement())
email String  @unique
name  String?
}
```

### Testing the Connection

Create a test file (test-connection.js):
```javascript
const { Pool } = require('pg');

async function testDatabaseConnection() {
const pool = new Pool({
    host: 'localhost',
    port: 5432,
    database: 'mydatabase',
    user: 'postgres',
    password: 'postgrespw'
});

try {
    const client = await pool.connect();
    const result = await client.query('SELECT NOW()');
    console.log('Database connection successful!');
    console.log('Current timestamp:', result.rows[0].now);
    client.release();
} catch (err) {
    console.error('Database connection error:', err);
} finally {
    await pool.end();
}
}

testDatabaseConnection();
```

Run the test:
```bash
node test-connection.js
```

### Common Issues and Solutions

1. Connection Refused
- Ensure Docker container is running
- Check if port 5432 is not being used by another service
- Verify network settings in docker-compose.yml

2. Authentication Failed
- Double-check credentials in connection string
- Ensure database name matches
- Verify user permissions

3. Connection Pool Issues
- Implement proper pool management
- Always release connections after use
- Use connection pooling with reasonable limits:
```javascript
const pool = new Pool({
    ...connectionConfig,
    max: 20, // maximum number of clients
    idleTimeoutMillis: 30000
});
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


# UniswapV4 Indexer

A blockchain indexer for Uniswap V4 swap events on Arbitrum using [rindexer](https://rindexer.xyz/).

## Overview

This project indexes Uniswap V4 `Swap` events from the PoolManager contract on Arbitrum mainnet and stores them in a PostgreSQL database.

**Contract Details:**
- Network: Arbitrum (Chain ID: 42161)
- Contract: PoolManager (`0x360E68faCcca8cA495c1B759Fd9EEe466db9FB32`)
- Events: Swap events with rapid polling (~50ms)

## Prerequisites

- Docker and Docker Compose
- [rindexer CLI](https://rindexer.xyz/) installed locally,
- [Redis](https://redis.io/docs/latest/operate/oss_and_stack/install/archive/install-redis/install-redis-on-mac-os/) installed locally,
- Arbitrum RPC endpoint (e.g., Alchemy, Infura)

## Setup

1. **Clone the repository and navigate to the project directory**

2. **Set up environment variables:**
   ```bash
   cp .env.example .env
   ```

3. **Edit `.env` file and add your RPC endpoint:**
   ```bash
   DATABASE_URL=postgresql://postgres:postgres@localhost:5432/rindexer_uniswapv4indexer
   ARB_MAINNET_RPC=https://arb-mainnet.g.alchemy.com/v2/your-api-key-here
   REDIS_CONNECTION_URI=redis://localhost:6379
   ```

## Running the Indexer

### Step 1: Start PostgreSQL Database

Start the PostgreSQL database in Docker:

```bash
docker compose up -d
```

This will:
- Create a PostgreSQL 16 container
- Set up the `rindexer_uniswapv4indexer` database
- Expose the database on `localhost:5432`

### Step 2: Run Redis server

```bash
redis-server
```


Depending on the port, update REDIS_CONNECTION_URI accordingly. Default port: redis://localhost:6379

### Step 3: Run the Indexer

Run rindexer locally (connects to the containerized database):

```bash
rindexer start indexer
```

The indexer will:
- Connect to the PostgreSQL database running in Docker
- Start indexing Uniswap V4 swap events from Arbitrum
- Create necessary tables automatically
- Begin rapid polling for new events

## Monitoring

### Check Database Status
```bash
docker ps
```

### View Database Logs
```bash
docker logs uniswap_indexer_postgres
```

### Connect to Database
```bash
PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d rindexer_uniswapv4indexer
```

### Stop Services
```bash
# Stop indexer
# Press Ctrl+C in the terminal running rindexer

# Stop PostgreSQL
docker compose down
```

## Project Structure

```
.
├── README.md
├── docker-compose.yml          # PostgreSQL container configuration
├── rindexer.yaml              # Indexer configuration
├── abis/
│   └── PoolManager.abi.json   # Uniswap V4 PoolManager ABI
├── .env                       # Environment variables
└── .env.example               # Environment template
```

## Configuration

### rindexer.yaml
- **Network**: Arbitrum mainnet with rapid block polling
- **Contract**: Uniswap V4 PoolManager
- **Events**: Swap events only
- **Storage**: PostgreSQL backend

### Environment Variables
- `DATABASE_URL`: PostgreSQL connection string
- `ARB_MAINNET_RPC`: Arbitrum RPC endpoint

## Troubleshooting

### Database Connection Issues
1. Ensure PostgreSQL container is running: `docker ps`
2. Check container health: `docker logs uniswap_indexer_postgres`
3. Verify connection: `PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d rindexer_uniswapv4indexer`

### Reset Database
If you need to start fresh:
```bash
docker compose down
docker volume rm uniswapv4indexer_postgres_data
docker compose up -d
```

### Port Conflicts
If port 5432 is already in use:
1. Stop other PostgreSQL services
2. Or modify the port in `docker-compose.yml` and update `DATABASE_URL` accordingly

## Development

The indexer uses a no-code approach with rindexer. To modify:

1. **Add new events**: Edit `include_events` in `rindexer.yaml`
2. **Change networks**: Update `networks` section in `rindexer.yaml`
3. **Modify contracts**: Update contract details in `rindexer.yaml`

After configuration changes, restart the indexer:
```bash
rindexer start indexer
```
# UniswapV4 Indexer

A blockchain indexer for Uniswap V4 swap events on Arbitrum using [rindexer](https://rindexer.xyz/).

## Overview

This project indexes Uniswap V4 `Swap` events from the PoolManager contract on Arbitrum mainnet and stores them in a PostgreSQL database while streaming events to Redis.

**Contract Details:**
- Network: Arbitrum (Chain ID: 42161)
- Contract: PoolManager (`0x360E68faCcca8cA495c1B759Fd9EEe466db9FB32`)
- Events: Swap events from specific pools (ETH/USDC, WBTC/USDC, USDT/USDC)

## Prerequisites

- Docker and Docker Compose
- Arbitrum RPC endpoint (e.g., Alchemy, Infura, DRPC)

**Optional (for local setup only):**
- [rindexer CLI](https://rindexer.xyz/) installed locally
- [Redis](https://redis.io/docs/latest/operate/oss_and_stack/install/archive/install-redis/install-redis-on-mac-os/) installed locally

## Setup

1. **Clone the repository and navigate to the project directory**

2. **Set up environment variables:**
   ```bash
   cp .env.example .env
   ```

3. **Edit `.env` file and add your RPC endpoint:**
   ```bash
   ARB_MAINNET_RPC=https://your-rpc-endpoint-here
   ```

## Running Options

### Option 1: Full Docker Setup (Recommended)

Run everything in Docker containers:

```bash
docker compose up -d
```

This will start:
- PostgreSQL database
- Redis server
- Rindexer indexer

**View indexer logs:**
```bash
docker compose logs rindexer --follow
```

### Option 2: Local Development Setup

For development with local rindexer:

1. **Start only the database and Redis:**
   ```bash
   docker compose up postgres redis -d
   ```

2. **Update your `.env` file for local connections:**
   ```bash
   ARB_MAINNET_RPC=https://your-rpc-endpoint-here
   DATABASE_URL=postgresql://postgres:postgres@localhost:5432/rindexer_uniswapv4indexer
   REDIS_CONNECTION_URI=redis://localhost:6380
   ```

3. **Run rindexer locally:**
   ```bash
   rindexer start indexer
   ```

## Monitoring

### Check Running Services
```bash
docker compose ps
```

### View Logs
```bash
# Indexer logs
docker compose logs rindexer --follow

# Database logs
docker compose logs postgres

# Redis logs
docker compose logs redis

# All services
docker compose logs --follow
```

### Connect to Database
```bash
PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d rindexer_uniswapv4indexer
```

### Connect to Redis
```bash
# If using Docker Redis (port 6380)
redis-cli -p 6380

# If using local Redis (port 6379)
redis-cli -p 6379
```

### Stop Services
```bash
# Stop all services
docker compose down

# Stop specific service
docker compose stop rindexer
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
- `ARB_MAINNET_RPC`: Arbitrum RPC endpoint (required)
- `DATABASE_URL`: PostgreSQL connection string (only for local setup)
- `REDIS_CONNECTION_URI`: Redis connection string (only for local setup)

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
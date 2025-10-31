# M4 Docker Tutorial: Running KZero Full Stack Locally

A comprehensive guide for running the complete KZero stack locally using Docker, including all six essential components required for the full authentication and wallet workflow.

## 1. Overview

This tutorial provides step-by-step instructions for setting up and running the complete KZero ecosystem locally. The setup consists of six components working together:

1. **POSTGRES_DB** - PostgreSQL database for storing authentication and user data
2. **auth-server** - OAuth2 authentication server with zkLogin support (from kzero-service)
3. **proof-server** - WebSocket proof generation task server (from kzero-service)
4. **proof-worker** - Zero-knowledge proof generation worker (from kzero-service)
5. **wallet** - KZero wallet application (from kzero-wallet)
6. **example** - Example application demonstrating KZero integration (from kzero-wallet)

These components work together to provide a complete end-to-end zkLogin authentication flow, allowing users to authenticate with their Google accounts and interact with the blockchain through zero-knowledge proofs.

## 2. Prerequisites

Before starting, ensure you have the following installed:

- **Node.js** >= 20
- **pnpm** >= 10.17.1
- **Docker** >= 20.10 and **Docker Compose** >= 2.0
- **Git** for cloning repositories
- **Google OAuth Client ID** - You need to register an OAuth application in Google Cloud Console and obtain a Client ID

## 3. Step 1: Setting Up kzero-service

### 3.1 Clone the Repository

First, clone the kzero-service repository from the `feature/auth-server` branch:

```bash
git clone https://github.com/kzero-xyz/kzero-service.git
cd kzero-service
git checkout feature/auth-server
```

### 3.2 Prepare Assets(Same as the `DOCKER.md` in `kzero-service/feature/auth-server`)

Download the ZK proof assets for the proof-worker. This step can be run before `pnpm install` as it doesn't require dependencies:

```bash
# From project root (no dependencies required)
pnpm setup:proof-worker
```

This will download `zkLogin.zkey` (588MB) to `apps/proof-worker/assets/`.

> **Note**: This is a large file download. Ensure you have sufficient disk space and a stable internet connection.

### 3.3 Configure Environment

Copy the example environment files and configure them:

```bash
cp apps/auth-server/.env.example apps/auth-server/.env
cp apps/proof-server/.env.example apps/proof-server/.env
cp apps/proof-worker/.env.example apps/proof-worker/.env
```

#### Important Configuration Steps

Edit `apps/auth-server/.env` and make the following critical configurations:

1. **Set Google Client ID**: Replace the placeholder Google OAuth Client ID with your actual Client ID obtained from Google Cloud Console:
   ```env
    GOOGLE_CLIENT_ID=your-google-client-id
    GOOGLE_CLIENT_SECRET=your-google-client-secret
   ```

2. **Set Frontend Origin**: Configure `FRONTEND_ORIGIN` to match the port where kzero-wallet will run. It's recommended to set this to port `5176`:
   ```env
   FRONTEND_ORIGIN=http://localhost:5176
   ```

> **⚠️ Important**: The `FRONTEND_ORIGIN` must match the port where the wallet will run (5176), otherwise CORS errors will occur during authentication.

3. **Comment Out**: If you are running locally, make sure to keep the following line in `apps/auth-server/.env` commented out:
    ```env
    # SALT_SERVER_URL=
    ```
> Do not set any value for `SALT_SERVER_URL` unless you intend to connect to a remote Salt Server.



### 3.4 Run Database Migration

Before starting the services, run the database migrations to set up the PostgreSQL schema:

```bash
docker compose run --rm migrate
```

This will initialize the database schema required by the authentication services.

### 3.5 Start Services

Start all services in detached mode:

```bash
docker compose up -d auth-server
docker compose up -d proof-server proof-worker
```


#### Verify Services Are Running

Check that all services are up and running:

```bash
docker compose ps
```

You should see the following services running:
- `POSTGRES_DB` (postgres)
- `auth-server`
- `proof-server`
- `proof-worker`

#### Check Service Logs

If you need to troubleshoot, you can view logs:

```bash
# View all service logs
docker compose logs -f

# View logs for a specific service
docker compose logs -f auth-server
docker compose logs -f proof-server
docker compose logs -f proof-worker
```

At this point, the four service components are running:
- ✅ POSTGRES_DB
- ✅ auth-server
- ✅ proof-server
- ✅ proof-worker

## 4. Step 2: Setting Up kzero-wallet

### 4.1 Clone the Repository

Open a new terminal window and clone the kzero-wallet repository from the `fix-prepareCall` branch:

```bash
git clone https://github.com/kzero-xyz/kzero-wallet.git
cd kzero-wallet
git checkout fix-prepareCall
```

### 4.2 Install Dependencies and Build

Install all dependencies and build the packages:

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm build
```

> **Note**: The build process may take a few minutes depending on your machine's performance.

### 4.3 Start the Example Application

From the kzero-wallet root directory, start the example application on port 5175:

```bash
pnpm dev:example --port 5175
```

This will start the example application that demonstrates KZero integration. Keep this terminal window open and running.

### 4.4 Start the Wallet Application

Open a **new terminal window**, navigate to the kzero-wallet root directory, and start the wallet application on port 5176:

```bash
cd /path/to/kzero-wallet
pnpm dev:wallet --port=5176
```

This will start the wallet application that handles the authentication flow and wallet interactions.

> **⚠️ Important**: The wallet must run on port 5176 (or match the `FRONTEND_ORIGIN` configured in `auth-server/.env`), as this is what the auth-server expects for CORS validation.

At this point, all six components are running:
- ✅ POSTGRES_DB
- ✅ auth-server
- ✅ proof-server
- ✅ proof-worker
- ✅ wallet (on port 5176)
- ✅ example (on port 5175)

## 5. Testing the Complete Flow

Once all components are running, you can test the complete KZero authentication and wallet flow:

1. **Open your browser** and navigate to:
   ```
   http://localhost:5175
   ```

2. **Experience the full workflow**:
   - The example application will guide you through the authentication process
   - You'll authenticate with your Google account (using the Client ID you configured)
   - The system will generate zero-knowledge proofs for authentication
   - You can interact with wallet functionality through the authenticated session

> **⚠️ Note**: When generating proofs locally, you may experience longer pending times. This is because local execution doesn't use a GPU environment - proof generation uses the Groth16 JavaScript library instead. The compilation time depends on your machine's performance. For reference, on a 2021 MacBook Pro M1, proof generation may take around 40 seconds.

### Verification Checklist

To ensure everything is working correctly, verify:

- ✅ All Docker containers are running (`docker compose ps`)
- ✅ auth-server is accessible (check logs for startup messages)
- ✅ proof-server and proof-worker are running (check logs for WebSocket connections)
- ✅ Wallet is running on `http://localhost:5176`
- ✅ Example app is running on `http://localhost:5175`
- ✅ Browser console shows no CORS errors
- ✅ Google OAuth authentication popup appears correctly

## 6. Troubleshooting

### Common Issues

#### CORS Errors

**Problem**: Browser console shows CORS errors when trying to authenticate.

**Solution**: 
- Verify that `FRONTEND_ORIGIN` in `apps/auth-server/.env` matches the wallet port (should be `http://localhost:5176`)
- Restart the auth-server after changing the environment: `docker compose restart auth-server`

#### Database Connection Issues

**Problem**: Services fail to connect to PostgreSQL.

**Solution**:
- Verify PostgreSQL container is running: `docker compose ps`
- Check database logs: `docker compose logs postgres`
- Ensure migrations ran successfully: `docker compose run --rm migrate`

#### Proof Generation Failures

**Problem**: Proof generation fails or times out.

**Solution**:
- Verify proof-worker assets are downloaded correctly (check `apps/proof-worker/assets/zkLogin.zkey` exists)
- Check proof-worker logs: `docker compose logs -f proof-worker`
- Ensure proof-server is running and accessible

### Stopping Services

To stop all services:

```bash
# Stop kzero-service Docker containers
cd kzero-service
docker compose down

# Stop kzero-wallet applications
# Press Ctrl+C in both terminal windows running wallet and example
```

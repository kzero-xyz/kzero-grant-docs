# M4 Docker Tutorial: Running KZero Full Stack Locally

A comprehensive guide for running the complete KZero stack locally using Docker, enabling you to experience the full end-to-end zkLogin authentication and wallet workflow.

## 1. Overview

This tutorial provides step-by-step instructions for setting up and running the complete KZero ecosystem locally. The setup consists of two main parts:

1. **kzero-service** - Backend services including database, authentication, and proof generation
2. **kzero-wallet** - Frontend wallet application and example implementation

These components work together to provide a complete end-to-end zkLogin authentication flow, allowing users to authenticate with their Google accounts and interact with the blockchain through zero-knowledge proofs.

## 2. Prerequisites

Before starting, ensure you have the following installed:

- **Node.js** >= 20
- **pnpm** >= 10.17.1
- **Docker** >= 20.10 and **Docker Compose** >= 2.0
- **Git** for cloning repositories
- **Google OAuth Client ID** - You need to register an OAuth application in Google Cloud Console and obtain a Client ID and Client Secret

## 3. Step 1: Starting kzero-service

The kzero-service provides a ready-to-use Docker setup that allows you to start all backend services with a single command.

### 3.1 Clone the Repository

Clone the kzero-service-docker repository:

```bash
git clone https://github.com/kzero-xyz/kzero-service-docker.git
cd kzero-service-docker
```

### 3.2 Start All Services

Execute the startup script to automatically launch all required services:

```bash
./start-services.sh
```

This script will automatically start the following services:

- **PostgreSQL** - Database server for storing application data
- **auth-server** - OAuth2 authentication service with zkLogin support
- **proof-server** - WebSocket-based proof task scheduler
- **proof-worker** - Zero-knowledge proof generation worker

#### Verify Services Are Running

Check that all services are up and running:

```bash
docker compose ps
```

You should see all four services listed and running:
- PostgreSQL
- auth-server
- proof-server
- proof-worker

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

> **Note**: The startup script handles all necessary configurations, including environment setup and database migrations. You may need to configure your Google OAuth Client ID in the environment files if required by your setup.

At this point, the backend service components are running:
- ✅ PostgreSQL
- ✅ auth-server
- ✅ proof-server
- ✅ proof-worker

## 4. Step 2: Starting kzero-wallet

The kzero-wallet provides the frontend application and example implementation for interacting with KZero.

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

From the kzero-wallet root directory, start the example application on port 5175 in one terminal window:

```bash
pnpm dev:example --port 5175
```

Keep this terminal window open and running. The example application demonstrates KZero integration and serves as the main user interface.

### 4.4 Start the Wallet Application

Open a **new terminal window**, navigate to the kzero-wallet root directory, and start the wallet application on port 5176:

```bash
cd /path/to/kzero-wallet
pnpm dev:wallet --port=5176
```

This will start the wallet application that handles the authentication flow and wallet interactions. Keep this terminal window open as well.

> **⚠️ Important**: The wallet must run on port 5176, as this is the expected port configured in the auth-server for CORS validation.

At this point, all components are running:
- ✅ PostgreSQL
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
   - You'll authenticate with your Google account (using the OAuth Client ID configured in the services)
   - The system will generate zero-knowledge proofs for authentication
   - You can interact with wallet functionality through the authenticated session

> **⚠️ Note**: When generating proofs locally, you may experience longer pending times. This is because local execution doesn't use a GPU environment - proof generation uses the Groth16 JavaScript library instead. The compilation time depends on your machine's performance. For reference, on a 2021 MacBook Pro M1, proof generation may take around 40 seconds.

### Verification Checklist

To ensure everything is working correctly, verify:

- ✅ All Docker containers are running (`docker compose ps` in kzero-service-docker directory)
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
- Verify that the `FRONTEND_ORIGIN` in the auth-server configuration matches the wallet port (`http://localhost:5176`)
- Check the auth-server logs for CORS-related messages
- Restart the auth-server service if needed

#### Database Connection Issues

**Problem**: Services fail to connect to PostgreSQL.

**Solution**:
- Verify PostgreSQL container is running: `docker compose ps` (in kzero-service-docker directory)
- Check database logs: `docker compose logs postgres`
- Ensure the startup script completed successfully

#### Proof Generation Failures

**Problem**: Proof generation fails or times out.

**Solution**:
- Check proof-worker logs: `docker compose logs -f proof-worker`
- Ensure proof-server is running and accessible
- Verify that all required assets are properly configured

#### Port Conflicts

**Problem**: Ports 5175 or 5176 are already in use.

**Solution**:
- Find the process using the port: `lsof -i :5175` or `lsof -i :5176`
- Kill the process or change the ports in the startup commands
- If changing ports, update the `FRONTEND_ORIGIN` configuration in auth-server accordingly

#### Google OAuth Errors

**Problem**: Google authentication fails with "invalid client" error.

**Solution**:
- Verify your Google Client ID and Client Secret are correctly configured
- Ensure the OAuth redirect URI matches what's configured in Google Cloud Console
- Check auth-server logs for detailed error messages

### Stopping Services

To stop all services:

```bash
# Stop kzero-service Docker containers
cd kzero-service-docker
docker compose down

# Stop kzero-wallet applications
# Press Ctrl+C in both terminal windows running wallet and example
```

## 7. Architecture Overview

The complete KZero stack works together as follows:

```
┌─────────────┐
│   Browser   │
│  (User)     │
└──────┬──────┘
       │
       │ HTTP/WebSocket
       │
       v
┌─────────────────────────────────────────┐
│         Example App (5175)              │
│         Wallet App (5176)                │
└──────┬────────────────┬──────────────────┘
       │                │
       │ Auth Requests  │ Proof Requests
       │                │
       v                v
┌──────────────┐  ┌──────────────────┐
│ auth-server  │  │  proof-server    │
│              │  │                  │
└──────┬───────┘  └──────────┬───────┘
       │                     │
       │                     │ WebSocket
       │                     │
       v                     v
┌──────────────┐      ┌──────────────┐
│ PostgreSQL   │      │proof-worker  │
│              │      │              │
└──────────────┘      └──────────────┘
```

1. User interacts with the **Example App** (port 5175)
2. **Wallet App** (port 5176) handles authentication requests
3. **auth-server** processes OAuth authentication and manages sessions
4. **proof-server** coordinates proof generation tasks
5. **proof-worker** generates zero-knowledge proofs
6. **PostgreSQL** stores authentication and user data

## 8. Additional Resources

For more detailed information and implementation details, please refer to the following KZero repositories:

- **kzero-service**: [https://github.com/kzero-xyz/kzero-service](https://github.com/kzero-xyz/kzero-service) - Backend services including auth-server, proof-server, and proof-worker
- **kzero-service-docker**: [https://github.com/kzero-xyz/kzero-service-docker](https://github.com/kzero-xyz/kzero-service-docker) - Docker setup for easy local deployment
- **kzero-wallet**: [https://github.com/kzero-xyz/kzero-wallet](https://github.com/kzero-xyz/kzero-wallet) - Wallet SDK and example applications
- **kzero**: [https://github.com/kzero-xyz/kzero](https://github.com/kzero-xyz/kzero) - Core KZero project repository
- **kzero-salt-enclave-service**: [https://github.com/kzero-xyz/kzero-salt-enclave-service](https://github.com/kzero-xyz/kzero-salt-enclave-service) - Salt generation service with Intel SGX enclave support
- **kzero-mvp-demo**: [https://github.com/kzero-xyz/kzero-mvp-demo](https://github.com/kzero-xyz/kzero-mvp-demo) - MVP demonstration and example implementations

---

**Note**: This tutorial is designed for local development and testing purposes. For production deployments, ensure proper security configurations, use production-grade credentials, and follow security best practices for OAuth and database configurations.


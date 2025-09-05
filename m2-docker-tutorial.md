# M2 Docker Tutorial: Running Kzero Node and MVP Frontend Demo

## Overview

This tutorial guides you through setting up and running the Kzero blockchain node and MVP frontend demo using Docker containers. The setup provides a complete development environment to showcase Kzero services and functionality.

> **⚠️ Important Notice**: This is an MVP demo version that only supports local development. Please ensure you use this tutorial in a local environment.

## Prerequisites

Before starting, ensure you have:

- **Operating System**: Linux (recommended)
- Docker installed on your system ([Install Docker](https://docs.docker.com/get-docker/))
- Available ports: 9944, 30333, and 5173
- Active internet connection for pulling Docker images

> **⚠️ Mac Users Notice**: macOS has limited support for Docker's `--network=host` mode. If you encounter networking issues, consider using a Linux environment or Docker Desktop with port forwarding configurations.

## Architecture Overview

The setup consists of two main components:

1. **Kzero Node**: A blockchain node running in development mode
2. **MVP Frontend Demo**: A web interface to interact with the Kzero node

## Step-by-Step Instructions

### Step 1: Pull Docker Images

First, download the required Docker images from Docker Hub:

```bash
# Pull the Kzero node image
docker pull kzeroxyz/kzero-node:v0.1.0-polkadot-stable2407

# Pull the MVP frontend demo image
docker pull kzeroxyz/kzero-mvp-demo:v0.1.0
```

These commands download the pre-built container images containing all necessary dependencies and configurations.

### Step 2: Start the Kzero Node

Launch the Kzero blockchain node in development mode:

```bash
docker run --network=host -d --rm \
  --name kzero-node \
  -p 9944:9944 \
  -p 30333:30333 \
  kzeroxyz/kzero-node:v0.1.0-polkadot-stable2407 \
  --dev \
  --rpc-external \
  -d=/tmp/dev

```

**Command Breakdown:**

- `--network=host`: Uses the host network directly for better performance
- `-d`: Run in background
- `--rm`: Automatically removes the container when stopped
- `--name kzero-node`: Names the container for easy reference
- `-p 9944:9944`: Exposes WebSocket RPC port for frontend communication
- `-p 30333:30333`: Exposes P2P networking port
- `--dev`: Runs the node in development mode
- `--rpc-external`: Allows external RPC connections

**Expected Output:**
You should see logs indicating the node is producing blocks and ready to accept connections.

### Step 3: Add Offchain Keys

You need to add offchain keys to enable the offchain worker to submit JWK (JSON Web Key) updates for authentication providers like Google, Twitch, and Apple. This is crucial for verifying user authenticity in the zkLogin system.

#### 3.1 Generate a New Key

First, generate a new key pair using the following command:

```bash
docker run --network=host -it --rm \
  --name kzero-node \
  -p 9944:9944 \
  -p 30333:30333 \
  kzeroxyz/kzero-node:v0.1.0-polkadot-stable2407 \
  key generate
```

**Output Example**

```bash
WARNING: Published ports are discarded when using host network mode
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
Secret phrase:       undo donor degree recycle slice test chase butter pony always diamond harvest
  Network ID:        substrate
  Secret seed:       0xe8eea138986517ed85f9d6cfdc966ef97f8489eca54292cb8b9701177494a454
  Public key (hex):  0x0a51c9091c08fe555346e04eb8afa9c4acd0d62a17595382ece74764bd1f5f60
  Account ID:        0x0a51c9091c08fe555346e04eb8afa9c4acd0d62a17595382ece74764bd1f5f60
  Public key (SS58): 5CJEdZzmF7UCo5c3zSGuWTmQQLHFtPgZomQPJQ7bfVVm1F8i
  SS58 Address:      5CJEdZzmF7UCo5c3zSGuWTmQQLHFtPgZomQPJQ7bfVVm1F8i
```

**Important**: Save the `Secret phrase` from the output as you'll need it for the next step.

#### 3.2 Insert the Key into Node Keystore

Add the generated key to the node's keystore to enable JWK update transactions:

```bash
docker exec kzero-node node-template key insert \
  -d=/tmp/dev \
  --chain=dev \
  --suri="undo donor degree recycle slice test chase butter pony always diamond harvest" \
  --key-type=zklo \
  --scheme=sr25519
```

**Note**: Replace the `--suri` value with your actual secret phrase generated in the previous step.

This key insertion allows the offchain worker to automatically submit JWK updates from authentication providers, ensuring the system can verify user credentials from OAuth providers.

#### 3.3 Register Offchain Keys On-Chain

The blockchain only allows trusted offchain worker operators to update JWK (JSON Web Keys). Therefore, you need to register the generated account on-chain using sudo operations.

1. **Open Polkadot.js Apps Interface**

   Navigate to the Polkadot.js Apps interface and connect to your local node:

   ```
   https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer
   ```

2. **Access Sudo Operations**

   - Go to **Developer** → **Sudo** in the navigation menu
   - This allows you to perform privileged operations on the development chain

3. **Register the Offchain Worker Account**

   - In the sudo interface, you'll need to call the appropriate extrinsic to register your offchain worker account
   - Use the **SS58 Address** from Step 3.1 (e.g., `5CJEdZzmF7UCo5c3zSGuWTmQQLHFtPgZomQPJQ7bfVVm1F8i`)
   - This registration ensures that only your trusted offchain worker can submit JWK updates

   ![key-register](https://p.ipic.vip/pi169m.png)

   After a few seconds, you will see JWK stored onchain by querying zklogin->jwks:

![jwk-storage](https://p.ipic.vip/kmz5nd.png)
### Step 4: Start the MVP Frontend Demo

In a new terminal window, launch the frontend application:

```bash
docker run --network=host -d --rm \
  -p 5173:5173 \
  --name kzero-mvp-demo \
  kzeroxyz/kzero-mvp-demo:v0.1.0
```

### Step 5: Access the Application

Open your web browser and navigate to:

```
http://localhost:5173
```

You should now see the Kzero MVP demo interface connected to your local node.

![kzero-demo-interface](https://p.ipic.vip/smg71t.png)

**Using the Demo Interface:**
Simply follow the on-page logic and instructions to interact with the demo. The interface will guide you through the zkLogin process.

> **⚠️ Important Note**: During JWT token capture, the process may be unstable. If the token is not captured correctly, please retry the operation several times until successful.

## Verifying zkLogin Transactions

Finally, on the demo page, you will see the following display style, which indicates that the zk transaction has been sent successfully:

![zk-transaction-success](https://p.ipic.vip/udi8m5.jpg)

Then, you can return to the Polkadot.js Apps interface to verify and inspect the transaction details, as shown in the image below:

![zklogin-tx](https://p.ipic.vip/cql8hr.png)

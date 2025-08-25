# M2 Docker Tutorial: Running Kzero Node and MVP Frontend Demo

## Overview

This tutorial guides you through setting up and running the Kzero blockchain node and MVP frontend demo using Docker containers. The setup provides a complete development environment to showcase Kzero services and functionality.

> **⚠️ Important Notice**: This is an MVP demo version that only supports local development. Please ensure you use this tutorial in a local environment.

## Prerequisites

Before starting, ensure you have:

- Docker installed on your system ([Install Docker](https://docs.docker.com/get-docker/))
- Available ports: 9944, 30333, and 5173
- Active internet connection for pulling Docker images

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
docker run --network=host -it --rm \
  --name kzero-node \
  -p 9944:9944 \
  -p 30333:30333 \
  kzeroxyz/kzero-node:v0.1.0-polkadot-stable2407 \
  --dev \
  --rpc-external
```

**Command Breakdown:**

- `--network=host`: Uses the host network directly for better performance
- `-it`: Interactive terminal mode for viewing logs
- `--rm`: Automatically removes the container when stopped
- `--name kzero-node`: Names the container for easy reference
- `-p 9944:9944`: Exposes WebSocket RPC port for frontend communication
- `-p 30333:30333`: Exposes P2P networking port
- `--dev`: Runs the node in development mode
- `--rpc-external`: Allows external RPC connections

**Expected Output:**
You should see logs indicating the node is producing blocks and ready to accept connections.

### Step 3: Start the MVP Frontend Demo

In a new terminal window, launch the frontend application:

```bash
docker run --network=host -it --rm \
  -p 5173:5173 \
  --name kzero-mvp-demo \
  kzeroxyz/kzero-mvp-demo:v0.1.0
```

**Command Breakdown:**

- `--network=host`: Uses the host network to communicate with the node
- `-it`: Interactive terminal mode for viewing logs
- `--rm`: Automatically removes the container when stopped
- `-p 5173:5173`: Exposes the web server port
- `--name kzero-mvp-demo`: Names the container for easy reference

### Step 4: Access the Application

Open your web browser and navigate to:

```
http://localhost:5173
```

You should now see the Kzero MVP demo interface connected to your local node.

![kzero-demo-interface](https://p.ipic.vip/smg71t.png)

## Network Configuration Details

### Ports Used

| Port  | Service       | Description                            |
| ----- | ------------- | -------------------------------------- |
| 9944  | WebSocket RPC | Frontend-to-node communication         |
| 30333 | P2P           | Node networking (optional in dev mode) |
| 5173  | Web Server    | Frontend application                   |

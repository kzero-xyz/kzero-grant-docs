# M3 Docker Tutorial: Running Kzero Salt Service (Prod & Dev)

A comprehensive guide for running Intel SGX enclave applications in both production (Hardware) and development (SIM) environments using Docker.

## 1. Overview

This tutorial provides two deployment options for the Kzero Salt Service:

- **Production Environment**: Uses actual Intel SGX hardware on Alibaba Cloud vSGX instances
- **Development Environment**: Uses Docker-based SGX simulation mode for local development and testing

The service processes JWT tokens and returns corresponding salt information using hardware-based confidential computing or simulation mode.

## 2. Prerequisites

### Production Environment (Hardware)
- **Cloud Provider**: Alibaba Cloud ECS
- **Instance Type**: vSGX instances (g7t, c7t, or r7t families)
- **Operating System**: Ubuntu 22.04 UEFI (recommended)
- **Architecture**: x86_64
- **Storage**: At least 8GB free disk space

### Development Environment (SIM)
- **Operating System**: Ubuntu 20.04+ (recommended)
- **Architecture**: x86_64
- **Docker**: Version 20.10+
- **Storage**: At least 8GB free disk space

## 3. Production Environment Setup (Hardware Mode)
> ⚠️ **Note**: For dev usage, we recommend you to use the [SIM Mode](#4-development-environment-setup-sim-mode) for easier development, because it doesn't require SGX hardware support, which is detailed in the next section.

### Step 1: Create vSGX Instance

1. Log in to the [Alibaba Cloud ECS Console](https://ecs.console.aliyun.com/)
2. Create a new ECS instance with the following specifications:
   - **Instance Family**: g7t, c7t, or r7t (vSGX instances)
   - **Image**: Ubuntu 22.04 UEFI

### Step 2: Initialize SGX Environment

Follow the [Alibaba Cloud SGX Setup Guide](https://www.alibabacloud.com/help/en/ecs/user-guide/build-an-sgx-encrypted-computing-environment#03e6d895betxu) to initialize the SGX environment:

#### 2.1 Check SGX Status
```bash
# Install cpuid
sudo apt-get update && sudo apt-get install -y --no-install-recommends cpuid

# Check if SGX is enabled
cpuid -1 -l 0x7 |grep SGX
```

#### 2.2 Install SGX Driver & Build the SGX confidential computing environment
```bash
# Create installation script
cat <<'EOF' > install_sgx_dcap.sh
#!/bin/bash
version_id=$(cat /etc/os-release|grep "VERSION_ID"|cut -d"=" -f2|tr -d "\"")
version_codename=$(cat /etc/os-release|grep "VERSION_CODENAME"|cut -d"=" -f2)
apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential dkms curl wget
if [ ! -e /dev/sgx/enclave -a ! -e /dev/sgx_enclave ]; then
  dcap_version=$(curl -s https://download.01.org/intel-sgx/latest/version.xml |grep dcap| sed -r 's/.*>(.*)<.*/\1/')
  dcap_files=$(curl -s https://download.01.org/intel-sgx/latest/dcap-latest/linux/SHA256SUM_dcap_${dcap_version}.cfg)
  echo "${dcap_files}" | grep "ubuntu${version_id}-server" |grep "sgx_linux_x64_driver" | awk '{print $2}' | xargs -I{} curl -O -J https://download.01.org/intel-sgx/latest/dcap-latest/linux/{}
      
  bash sgx_linux_x64_driver*.bin
else
  echo "driver already installed"
fi
EOF

# Run the script
sudo bash ./install_sgx_dcap.sh

# Verify driver installation
ls -l /dev/{sgx_enclave,sgx_provision}

cat <<'EOF' > install_sgx_sdk.sh
#!/bin/bash

version_id=$(cat /etc/os-release|grep "VERSION_ID"|cut -d"=" -f2|tr -d "\"")
version_codename=$(cat /etc/os-release|grep "VERSION_CODENAME"|cut -d"=" -f2)
apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential dkms curl wget

dcap_version=$(curl -s https://download.01.org/intel-sgx/latest/version.xml |grep dcap| sed -r 's/.*>(.*)<.*/\1/')
linux_version=$(curl -s https://download.01.org/intel-sgx/latest/version.xml |grep linux| sed -r 's/.*>(.*)<.*/\1/')
dcap_files=$(curl -s https://download.01.org/intel-sgx/latest/dcap-latest/linux/SHA256SUM_dcap_${dcap_version}.cfg)
echo "${dcap_files}" | grep "ubuntu${version_id}-server" | awk '{print $2}' | xargs -I{} curl -O -J https://download.01.org/intel-sgx/latest/dcap-latest/linux/{}

# install sgx_sdk
bash sgx_linux_x64_sdk*.bin --prefix /opt/intel
source /opt/intel/sgxsdk/environment

# install psw
echo "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu ${version_codename} main" |  tee /etc/apt/sources.list.d/intelsgx.list
wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | apt-key add -
apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y libsgx-launch libsgx-urts libsgx-epid libsgx-quote-ex libsgx-dcap-ql libsgx-dcap-ql-dev
systemctl enable --now aesmd.service
EOF

sudo bash ./install_sgx_sdk.sh
```

#### 2.3 Install SGX SDK and Runtime
```bash
# Update package list
sudo apt update

# Install build tools
sudo apt install -y build-essential

# Install Intel SGX SDK (follow the official Intel installation guide)
# This step may vary based on the specific SGX SDK version
```

### Step 3: Install Additional Dependencies

#### 3.1 Install System Dependencies
```bash
# Update package list
sudo apt update

# Install required libraries
sudo apt install -y libcurl4-openssl-dev libjson-c-dev libjwt-dev libssl-dev libmicrohttpd-dev
```

#### 3.2 Install JWT-CPP Library
```bash
# Clone and build jwt-cpp
cd /tmp
git clone https://github.com/Thalhammer/jwt-cpp.git
cd jwt-cpp
mkdir build && cd build
cmake ..
make -j4
sudo make install

# Update library cache
sudo ldconfig
```

### Step 4: Project Setup

#### 4.1 Clone and Build
```bash
# Clone the project (replace with your repository URL)
git clone git@github.com:kzero-xyz/kzero-salt-enclave-service.git
cd kzero-salt-enclave-service
git checkout enclave-hw-mode
# Build the project
make
```

#### 4.2 Run the Service
```bash
# Run the application
sudo ./app
```

The service will start on port 8080 and display startup messages.

## 4. Development Environment Setup (SIM Mode)

### Step 1: Build the Docker Image
> **Note:** Build time depends on your machine's performance. For reference, on a 4-CPU Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz (CPU MHz: 2792.998), the build process takes approximately 50 minutes.

```bash
docker pull kzeroxyz/kzero-salt-enclave-service:v0.1.2
```

### Step 2: Run the Container

```bash
docker run -d -p 8080:8080 --name test-enclave-new -e SGX_MODE=SIM kzeroxyz/kzero-salt-enclave-service:v0.1.2
```

### Test

```bash
docker run --rm --name test-enclave-test -e SGX_MODE=SIM kzeroxyz/kzero-salt-enclave-service:v0.1.2 make test-app
```

### Test Coverage
```bash
docker run --rm --name test-enclave-test -e SGX_MODE=SIM kzeroxyz/kzero-salt-enclave-service:v0.1.2 make test-coverage-app
```

You should see the following coverage report:
```bash
=== Test Results ===
All tests PASSED!
=== Test Suite Complete ===
Generating coverage report...
File 'App/App.cpp'
Lines executed:93.49% of 568
```
## 5. API Usage & Testing(Same for HW/SIM Mode)

### POST /get_salt

Processes JWT tokens and returns salt information.

**Request:**
```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"message": "your_jwt_token_here", "provider": "your_jwk_provider_here"}' \
  http://localhost:8080/get_salt
```

**Response:**
```json
{
  "salt": "generated_salt_value",
  "status": "success"
}
```

### Example with Google OAuth JWT

```bash
curl -s -X POST http://localhost:8080/get_salt \
  -H 'Content-Type: application/json' \
  -d '{"message":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjA3ZjA3OGYyNjQ3ZThjZDAxOWM0MGRhOTU2OWU0ZjUyNDc5OTEwOTQiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiI1NjA2MjkzNjU1MTctbXQ5ajlhcmZsY2dpMzVpOGhwb3B0cjY2cWdvMWxtZm0uYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiI1NjA2MjkzNjU1MTctbXQ5ajlhcmZsY2dpMzVpOGhwb3B0cjY2cWdvMWxtZm0uYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTExNDA0NjE1MzAyNDYxNjQ1MjYiLCJub25jZSI6InlwanZ6TXB6d09qelcycUlrVnBiQU9UTUZuVSIsIm5iZiI6MTc1Nzc1MjA2NCwiaWF0IjoxNzU3NzUyMzY0LCJleHAiOjE3NTc3NTU5NjQsImp0aSI6ImZkYzRmNTc3YWI0NWViZjhiMjU3NjkwMjQwZmUzMTYyOGFkOGI4ZmMifQ.D4NVKogzU76ZGV5HsUDTOHRwSSG1I3lgG4bUEWAeMW8G-QDnXBNY6QDFmYnVEWWx5VlejyQhvmdtJrXF2eDOMKGeOwnFlm1INQuneELbLz0sbKnDw62IKshgQGNP5jv5ij-HEKj3jkx8D1zof83duVDhFOUmDud0VZKPODfBRLbqoTJKz0cp0RwZ5k-SiT_aSeL-y_FodYcCt5VtXIZfvgWj_NbcscqPaIBMvjJ9-wFx8yD-6C5dIQDVgyhZGtLzwxRLZMr6yotBuz_49BlKquuPA6TgNdUvMRu35QRYEQYPx3RigYtKw_8GGW-LVbmZTKSBOKu8QMEweR9CCaBHvg","provider":"test_google"}'
```
> ⚠️ **Note**: The JWT used in the above example should return an error like `{"error":"JWT token has expired","status":"failed"}` because the JWT has a valid period, it may become expired. You can follow the process in [m2-docker-tutorial.md](m2-docker-tutorial.md) to generate a fresh, valid JWT and replace the `"message"` field in the request body to ensure a successful API call.



## References

- [Alibaba Cloud SGX Setup Guide](https://www.alibabacloud.com/help/en/ecs/user-guide/build-an-sgx-encrypted-computing-environment)
- [Intel SGX Documentation](https://software.intel.com/content/www/us/en/develop/topics/software-guard-extensions.html)
- [JWT-CPP Library](https://github.com/Thalhammer/jwt-cpp)
- [Kzero-Salt-Enclave-Service](git@github.com:kzero-xyz/kzero-salt-enclave-service.git): It contains 2 branches, which includes the detailed process of building in HW & SIM Mode.
---

**Note**: 
- The production environment requires actual SGX hardware and should be deployed on Alibaba Cloud vSGX instances or other SGX hardware supported instances for production use.
- The development environment is designed for development and testing purposes. For production deployments, ensure proper security configurations and consider using actual SGX hardware when available.

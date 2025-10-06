# Testing Guide for KZero Wallet SDK

## Overview

This document provides a comprehensive guide for testing the KZero Wallet SDK, specifically focusing on the Transaction Part and related components. Our testing strategy ensures full coverage of core functionality, robustness, and reliability.

## Testing Framework

We use **Vitest** as our primary testing framework, which provides:

- ⚡ **Fast execution** with Vite's native ES modules support
- 🔧 **TypeScript support** out of the box
- 🎯 **Jest-compatible API** for familiar testing patterns
- 📊 **Built-in coverage reporting**
- 🔄 **Watch mode** for development

### Configuration

The testing configuration is defined in `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    environment: 'node',
    globals: true
  }
});
```
## Running Tests

### Prerequisites

Ensure you have the required dependencies installed:

```bash
# Install dependencies
pnpm install

# Build the project first
pnpm build
```

### Test Commands

#### Run All Tests
```bash
# From root directory
pnpm test

# Or specifically for example-wallet
pnpm --filter @kzero/example-wallet test
```

#### Run Tests in Watch Mode
```bash
# Watch mode for development
pnpm test:watch

# Or for specific package
pnpm --filter @kzero/example-wallet test:watch
```

#### Run Type Checking
```bash
# Check TypeScript types
pnpm check-types
```


## Test Coverage

Our test suite provides comprehensive coverage across multiple dimensions:

### Core Functionality Tests

| Test Case | Purpose | Coverage |
|-----------|---------|----------|
| `should prepare call successfully` | Basic functionality | Return structure validation |
| `should return correct structure` | Output validation | Complete response object |

### ZK Authentication Tests

| Test Case | Purpose | Coverage |
|-----------|---------|----------|
| `should create JWK provider as Google` | Identity provider setup | Provider configuration |
| `should create ZK inputs with correct structure` | ZK proof processing | Input data structure |
| `should create ZK material with V1 structure` | ZK material generation | Versioned material format |

### Blockchain Interaction Tests

| Test Case | Purpose | Coverage |
|-----------|---------|----------|
| `should create transaction with correct call` | Transaction creation | Call parameter validation |
| `should query account nonce` | Account state query | Nonce retrieval |
| `should sign transaction with correct parameters` | Transaction signing | Signature process |

### Data Processing Tests

| Test Case | Purpose | Coverage |
|-----------|---------|----------|
| `should handle BigInt conversion for proof values` | Number conversion | BigInt handling |
| `should handle different proof point structures` | Data flexibility | Various input formats |

## Mock Strategy

Our testing approach uses comprehensive mocking to isolate the function under test:

### Mocked Dependencies

#### 1. ApiPromise Mock
```typescript
const mockApi = {
  tx: mockTx,
  registry: {
    createType: mockRegistryCreateType
  },
  createType: mockCreateType,
  query: {
    system: {
      account: mockQueryAccount
    }
  },
  genesisHash: '0x1234',
  runtimeVersion: { specVersion: 1 }
} as unknown as ApiPromise;
```

#### 2. KeyringPair Mock
```typescript
const mockPair = {
  address: '0x1234567890abcdef',
  publicKey: new Uint8Array(32).fill(1),
  secretKey: new Uint8Array(64).fill(2)
} as unknown as KeyringPair;
```

#### 3. Proof Data Mock
```typescript
const mockProof = {
  kid: 1,
  proof: {
    proof_points: {
      a: ['1', '2', '3'],
      b: [['1', '2'], ['3', '4'], ['5', '6']],
      c: ['7', '8', '9']
    },
    iss_base64_details: {
      value: '123456789',
      index_mod_4: 0
    },
    header: '987654321'
  },
  maxEpoch: '1000',
  zkAddress: '0xabcdef1234567890'
};
```

### Mock Data Distinction

We use distinct values for different purposes to ensure clarity:

| Type | Value | Purpose |
|------|-------|---------|
| **genesisHash** | `0x1234` | Blockchain network identifier |
| **address** | `0x1234567890abcdef` | User wallet address |
| **method** | `0xabcd` | Transaction method call |
| **zkAddress** | `0xabcdef1234567890` | ZK authentication address |


## Support

For technical support and questions:

- **GitHub Issues**: [https://github.com/kzero-xyz/kzero-wallet/issues](https://github.com/kzero-xyz/kzero-wallet/issues)
- **Github Repo**: [https://github.com/kzero-xyz/kzero-wallet](https://github.com/kzero-xyz/kzero-wallet)

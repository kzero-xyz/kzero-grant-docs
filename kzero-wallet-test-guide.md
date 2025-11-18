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
- 🌐 **Happy DOM environment** for browser API simulation


### Test Structure
Tests are organized using Vitest's describe and it blocks:
- **describe** - Groups related test cases
- **it** - Individual test cases
- **beforeEach / afterEach** - Setup and teardown hooks
- **vi** - Vitest's mocking utilities (e.g., vi.fn(), vi.spyOn(), vi.clearAllMocks())

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

# Or specifically for message-port
pnpm --filter @kzero/message-port test
```

#### Run Tests in Watch Mode
```bash
# For specific package
pnpm --filter @kzero/message-port test:watch
```

#### Run Tests with Coverage
```bash
# For getting the coverage report of tests
pnpm test:cov
```

#### Run Type Checking
```bash
# Check TypeScript types
pnpm check-types
```


## Test Coverage

Our test suite provides comprehensive coverage across multiple dimensions:

### Test Statistics

The current test suite includes:

| Package | Test Files | Tests | Status |
|---------|------------|-------|--------|
| `@kzero/zk-core` | 2 | 40 | ✅ All passing |
| `@kzero/message-port` | 2 | 25 | ✅ All passing |
| `@kzero/wallet` | 12 | 236 | ✅ All passing |
| `@kzero/zk-react` | 7 | 99 | ✅ All passing |
| **Total** | **23** | **400** | ✅ **100% passing** |

All tests are passing successfully with comprehensive coverage across:
- Core ZK functionality and proof generation
- Message port communication and security
- Wallet operations (account storage, authentication, encryption, session management)
- React hooks and provider components

### Core Functionality Tests

#### @kzero/zk-core Tests

| Test File | Test Cases | Coverage |
|-----------|------------|----------|
| `logger.test.ts` | 25 tests | Logger functionality, log levels, formatting |
| `getProof.test.ts` | 15 tests | ZK proof generation, input validation, error handling |

#### @kzero/message-port Tests

| Test File | Test Cases | Coverage |
|-----------|------------|----------|
| `BidirectionalPort.test.ts` | 17 tests | Message port communication, cleanup on destroy |
| `factory.test.ts` | 8 tests | Port factory functions, origin validation, security checks |

#### @kzero/wallet Tests

| Test File | Test Cases | Coverage |
|-----------|------------|----------|
| `accountStorage.test.ts` | 23 tests | Account storage, session management, data persistence |
| `authService.test.ts` | 18 tests | Authentication flow, API integration, error handling |
| `cardStore.test.ts` | 18 tests | Card data management and storage |
| `proofPollingService.test.ts` | 14 tests | Proof polling mechanism, status tracking, error recovery |
| `messageHandler.test.ts` | 17 tests | Message handling, request resolution/rejection |
| `urlParams.test.ts` | 25 tests | URL parameter parsing, theme handling, validation |
| `stateRestoration.test.ts` | 8 tests | Application state restoration and persistence |
| `applyTheme.test.ts` | 7 tests | Theme application and customization |
| `units.test.ts` | 28 tests | Unit conversion utilities and formatting |
| `ephemeralKeyUtils.test.ts` | 7 tests | Ephemeral key generation and management |
| `sessionManager.test.ts` | 48 tests | Session management, PIN encryption/decryption, keypair handling |
| `encryptionUtils.test.ts` | 23 tests | Encryption/decryption with PIN, data security, error handling |

#### @kzero/zk-react Tests

| Test File | Test Cases | Coverage |
|-----------|------------|----------|
| `errors.test.ts` | 4 tests | Error types and error handling |
| `helpers.test.ts` | 10 tests | Utility helper functions |
| `providersSerializer.test.ts` | 10 tests | Provider data serialization |
| `themeSerializer.test.ts` | 11 tests | Theme data serialization |
| `authWindow.test.ts` | 16 tests | Authentication window management |
| `WalletConnection.test.ts` | 26 tests | Wallet connection provider, auth flow |
| `useKzero.test.tsx` | 22 tests | React hooks for wallet integration |


## Support

For technical support and questions:

- **GitHub Issues**: [https://github.com/kzero-xyz/kzero-wallet/issues](https://github.com/kzero-xyz/kzero-wallet/issues)
- **Github Repo**: [https://github.com/kzero-xyz/kzero-wallet](https://github.com/kzero-xyz/kzero-wallet)

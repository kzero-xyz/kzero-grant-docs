# KZero Runtime Documentation

## Overview

The KZero runtime is a Substrate-based blockchain runtime that implements zkLogin authentication and provides two distinct account recovery mechanisms. This runtime enables users to authenticate using OAuth providers (Google, Facebook, etc.) through zero-knowledge proofs while maintaining robust account recovery capabilities.

## Core Components

### 1. ZkLogin Authentication System

The runtime integrates a custom `pallet-zklogin` that enables:
- **OAuth-based authentication** using zero-knowledge proofs
- **JWK (JSON Web Key) management** for verifying OAuth provider signatures
- **Ephemeral key verification** for secure transaction signing
- **Multi-provider support** including Google, Facebook, Twitch, Kakao, Apple, Slack, and GitHub

![workflow](./assets/login-flow.png)

### 2. Account Recovery Mechanisms

The runtime implements two complementary account recovery models:

#### A. Proxy-based Recovery (`pallet-proxy`)
> More details about pallet-proxy : https://docs.openzeppelin.com/substrate-runtimes/1.0.0/pallets/proxy

**Purpose**: Provides immediate delegation capabilities for account management.

**Key Features**:
- **Direct delegation**: Account owners can delegate specific permissions to trusted parties
- **Flexible proxy types**: Supports different proxy types (Any, NonTransfer, Governance, Staking)
- **Immediate effect**: No waiting period required
- **Granular control**: Can specify delay periods and proxy types

**Use Cases**:
- Temporary account management delegation
- Multi-signature setups
- Governance participation through proxies
- Immediate access transfer

**Implementation in Tests**:
```rust
// Add proxy delegation
let proxy_call = pallet_proxy::Call::add_proxy {
    delegate: MultiAddress::Id(delegatee),
    proxy_type: (),
    delay: 0u32,
};

// Execute calls through proxy
let proxy_call = pallet_proxy::Call::proxy {
    real: MultiAddress::Id(delegator),
    force_proxy_type: None,
    call: Box::new(transfer_call),
};
```

#### B. Social Recovery (`pallet-recovery`)
> More details about pallet-recovery : https://github.com/paritytech/substrate/blob/master/frame/recovery/README.md

**Purpose**: Provides M-of-N social recovery for lost account access.

**Key Features**:
- **Social recovery**: Requires threshold number of friends to approve recovery
- **Configurable parameters**: 
  - `friends`: List of trusted recovery contacts
  - `threshold`: Minimum number of friends needed for recovery
  - `delay_period`: Waiting period before recovery can be claimed
- **Deposit-based security**: Requires deposits to prevent spam
- **Multi-step process**: Initiate → Vouch → Claim → Execute

**Recovery Flow**:
1. **Setup**: Account owner creates recovery configuration with friends and threshold
2. **Initiation**: Rescuer initiates recovery process
3. **Vouching**: Friends vouch for the recovery attempt
4. **Claiming**: Rescuer claims recovery after delay period
5. **Execution**: Rescuer can execute calls on behalf of recovered account

**Implementation in Tests**:
```rust
// Create recovery configuration
let create_recovery_call = pallet_recovery::Call::create_recovery {
    friends: friends.clone(),
    threshold,
    delay_period,
};

// Initiate recovery
let initiate_call = pallet_recovery::Call::initiate_recovery {
    account: MultiAddress::Id(lost_account),
};

// Vouch for recovery
let vouch_call = pallet_recovery::Call::vouch_recovery {
    lost: MultiAddress::Id(lost_account),
    rescuer: MultiAddress::Id(rescuer_account),
};

// Claim recovery
let claim_call = pallet_recovery::Call::claim_recovery {
    account: MultiAddress::Id(lost_account),
};

// Execute as recovered account
let as_recovered_call = pallet_recovery::Call::as_recovered {
    account: MultiAddress::Id(lost_account),
    call: Box::new(transfer_call),
};
```

## Runtime Configuration

### Core Parameters

```rust
// Block time configuration
pub const MILLISECS_PER_BLOCK: u64 = 6000;
pub const SLOT_DURATION: u64 = MILLISECS_PER_BLOCK;

// Economic parameters
pub const EXISTENTIAL_DEPOSIT: u128 = 500;
pub const DOLLARS: Balance = 100 * CENTS;
pub const CENTS: Balance = 1_000 * MILLICENTS;
pub const MILLICENTS: Balance = 1_000_000_000;

// Recovery parameters
pub const ConfigDepositBase: Balance = 5 * DOLLARS;
pub const FriendDepositFactor: Balance = 50 * CENTS;
pub const MaxFriends: u32 = 9;
pub const RecoveryDeposit: Balance = 5 * DOLLARS;

// Proxy parameters
pub const ProxyDepositBase: Balance = 100 * MILLICENTS;
pub const ProxyDepositFactor: Balance = 5 * MILLICENTS;
pub const MaxProxies: u32 = 32;
```

### Pallet Configurations

#### ZkLogin Configuration
```rust
// For the inner transaction, we avoid the checkWeight, only do the charge
// When create the innerSignedPayload, we use the innerSignedExtra(which use the chargeTransactionPayment only)
pub type InnerSignedExtra = (
    frame_system::CheckNonZeroSender<Runtime>,
    frame_system::CheckSpecVersion<Runtime>,
    frame_system::CheckTxVersion<Runtime>,
    frame_system::CheckGenesis<Runtime>,
    frame_system::CheckEra<Runtime>,
    frame_system::CheckNonce<Runtime>,
    pallet_transaction_payment::ChargeTransactionPayment<Runtime>,
);

pub type InnerUncheckedExtrinsic = 
    generic::UncheckedExtrinsic<Address, RuntimeCall, Signature, InnerSignedExtra>;

// for inner transaction, we use the innerSignedPayload(which use the innerSignedExtra)
pub type InnerSignedPayload = generic::SignedPayload<RuntimeCall, InnerSignedExtra>;

impl pallet_zklogin::Config for Runtime {
    type AuthorityId = pallet_zklogin::crypto::ZkLoginAuthId;
    type MaxKeys = MaxKeys;
    type RuntimeEvent = RuntimeEvent;
    type Extrinsic = InnerUncheckedExtrinsic;
    type CheckedExtrinsic =
        <InnerUncheckedExtrinsic as sp_runtime::traits::Checkable<Self::Context>>::Checked;
    type UnsignedValidator = Runtime;
    type Context = frame_system::ChainContext<Runtime>;
    type Time = Timestamp;
    type WeightInfo = pallet_zklogin::weights::SubstrateWeight<Runtime>;
}
```

#### Recovery Configuration
```rust
impl pallet_recovery::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type WeightInfo = pallet_recovery::weights::SubstrateWeight<Runtime>;
	type RuntimeCall = RuntimeCall;
	type Currency = Balances;
	type ConfigDepositBase = ConfigDepositBase;
	type FriendDepositFactor = FriendDepositFactor;
	type MaxFriends = MaxFriends;
	type RecoveryDeposit = RecoveryDeposit;
}
```

#### Proxy Configuration
```rust
impl pallet_proxy::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type RuntimeCall = RuntimeCall;
	type Currency = Balances;
	type ProxyType = ProxyType;
	type ProxyDepositBase = ProxyDepositBase;
	type ProxyDepositFactor = ProxyDepositFactor;
	type MaxProxies = ConstU32<32>;
	type WeightInfo = pallet_proxy::weights::SubstrateWeight<Runtime>;
	type MaxPending = ConstU32<32>;
	type CallHasher = BlakeTwo256;
	type AnnouncementDepositBase = AnnouncementDepositBase;
	type AnnouncementDepositFactor = AnnouncementDepositFactor;
}
```

## Usage Examples

### Setting up ZkLogin with Recovery

```rust
// 1. Create zkLogin account
let zk_account = generate_zk_address(oauth_provider, jwk);

// 2. Set up proxy recovery
let proxy_call = pallet_proxy::Call::add_proxy {
    delegate: MultiAddress::Id(trusted_account),
    proxy_type: ProxyType::Any,
    delay: 0,
};

// 3. Set up social recovery
let recovery_call = pallet_recovery::Call::create_recovery {
    friends: vec![friend1, friend2, friend3],
    threshold: 2,
    delay_period: 100,
};
```

### Recovery Scenarios

**Scenario 1: Immediate Access (Proxy)**
```rust
// Delegate immediately needs access
pallet_proxy::Pallet::<Runtime>::proxy(
    RawOrigin::Signed(delegate).into(),
    MultiAddress::Id(zk_account),
    None,
    Box::new(transfer_call),
);
```

**Scenario 2: Lost Access Recovery (Social)**
```rust
// 1. Initiate recovery
pallet_recovery::Pallet::<Runtime>::initiate_recovery(
    RawOrigin::Signed(rescuer).into(),
    MultiAddress::Id(lost_zk_account),
);

// 2. Friends vouch
pallet_recovery::Pallet::<Runtime>::vouch_recovery(
    RawOrigin::Signed(friend1).into(),
    MultiAddress::Id(lost_zk_account),
    MultiAddress::Id(rescuer),
);

// 3. Claim recovery
pallet_recovery::Pallet::<Runtime>::claim_recovery(
    RawOrigin::Signed(rescuer).into(),
    MultiAddress::Id(lost_zk_account),
);

// 4. Execute as recovered
pallet_recovery::Pallet::<Runtime>::as_recovered(
    RawOrigin::Signed(rescuer).into(),
    MultiAddress::Id(lost_zk_account),
    Box::new(transfer_call),
);
```

## Conclusion

The KZero runtime provides a comprehensive solution for secure blockchain authentication and account recovery. By combining zkLogin authentication with dual recovery mechanisms, it offers:

- **Secure authentication** through OAuth providers with zero-knowledge proofs
- **Flexible recovery options** through both proxy and social recovery
- **Robust security** with deposit requirements and delay periods
- **User-friendly experience** with familiar OAuth login flows

This architecture ensures that users can maintain control over their accounts while providing reliable recovery mechanisms for various scenarios. 
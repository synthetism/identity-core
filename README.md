# @synet/identity-core

Core TypeScript types and interfaces for the Synet identity system. This package provides shared type definitions used across the Synet network for decentralized identity management, cryptographic keys, and verifiable credentials.

## Installation

```bash
npm install @synet/identity-core
```

## Overview

This package contains the foundational types for:
- **Decentralized Identifiers (DIDs)** - W3C standard for decentralized identity
- **Cryptographic Key Management** - Multi-algorithm key support including WireGuard (Curve25519)
- **Verifiable Credentials** - W3C standard for digital credentials
- **Service Endpoints** - DID service configuration

## API Reference

### Core Types

#### `Identity`
Main identity object containing DID, keys, and credentials.

```typescript
interface Identity {
  alias: string
  did: string
  kid: string
  publicKeyHex: string
  provider: string // 'did:key' | 'did:web'
  credential: SynetVerifiableCredential<BaseCredentialSubject>
  metadata?: Record<string, unknown>
  createdAt: Date
  version?: string
}
```

#### `IIdentifier`
Complete identifier with managed keys and services.

```typescript
interface IIdentifier {
  did: string
  alias?: string
  provider: string
  controllerKeyId?: string
  keys: IKey[]
  services: IService[]
}
```

### Cryptographic Key Types

#### `TKeyType`
Supported cryptographic key algorithms:

```typescript
type TKeyType = 
  | 'Ed25519'      // EdDSA signatures
  | 'Secp256k1'    // Bitcoin/Ethereum
  | 'Secp256r1'    // P-256 NIST curve
  | 'X25519'       // ECDH key exchange
  | 'Bls12381G1'   // BLS signatures (G1)
  | 'Bls12381G2'   // BLS signatures (G2)
  | 'Curve25519'   // WireGuard VPN keys
```

#### `IKey`
Cryptographic key with metadata:

```typescript
interface IKey {
  kid: string
  kms: string
  type: TKeyType
  publicKeyHex: string
  privateKeyHex?: string
  meta?: KeyMetadata | null
}
```

#### `ManagedPrivateKey`
Private key storage format:

```typescript
interface ManagedPrivateKey {
  alias: string
  privateKeyHex: string
  type: TKeyType
}
```

### Service Configuration

#### `IService`
DID service endpoint definition:

```typescript
interface IService {
  id: string
  type: string
  serviceEndpoint: IServiceEndpoint | IServiceEndpoint[]
  description?: string
}
```

#### `IServiceEndpoint`
Service endpoint URL or configuration:

```typescript
type IServiceEndpoint = string | Record<string, unknown>
```

### Algorithm Support

#### `TAlg`
Supported cryptographic algorithms:

```typescript
type TAlg = 
  | 'ES256K'    // ECDSA using secp256k1
  | 'ES256K-R'  // ECDSA using secp256k1 with recovery
  | 'ES256'     // ECDSA using P-256
  | 'EdDSA'     // EdDSA signatures
  | 'ECDH'      // Elliptic Curve Diffie-Hellman
  | 'ECDH-ES'   // ECDH Ephemeral Static
  | 'ECDH-1PU'  // ECDH One-Pass Unified Model
  | string      // Custom algorithms
```

#### `KeyMetadata`
Extended key information:

```typescript
interface KeyMetadata {
  algorithms?: TAlg[]
  [x: string]: unknown
}
```

## Key Type Compatibility

### WireGuard Support (`Curve25519`)

This package includes `Curve25519` support for WireGuard VPN keys, which extends beyond the standard Veramo key types. While Veramo's key management system may not natively support Curve25519, you have several options:

#### Option 1: Hybrid Approach (Recommended)
- Use Veramo for standard cryptographic operations (Ed25519, Secp256k1, etc.)
- Implement custom key management for Curve25519/WireGuard keys
- Store WireGuard keys separately in your vault system

#### Option 2: Extended Key Management
Create a custom key manager that:
```typescript
// Custom WireGuard key manager
interface WireGuardKeyManager {
  generateKeyPair(): Promise<{ publicKey: string; privateKey: string }>
  importKey(privateKeyHex: string): Promise<IKey>
  exportKey(kid: string): Promise<ManagedPrivateKey>
}
```

#### Option 3: Algorithm Mapping
Map Curve25519 to the closest supported algorithm:
- `X25519` for key exchange operations
- Custom handling for WireGuard-specific operations

### Key Type Usage Matrix

| Key Type | Use Case | Veramo Support | Custom Implementation |
|----------|----------|----------------|----------------------|
| Ed25519 | Signatures, DIDs | ✅ Native | Not needed |
| Secp256k1 | Bitcoin, Ethereum | ✅ Native | Not needed |
| Secp256r1 | Enterprise PKI | ✅ Native | Not needed |
| X25519 | Key Exchange | ✅ Native | Not needed |
| Bls12381G1/G2 | Advanced Signatures | ✅ Native | Not needed |
| Curve25519 | WireGuard VPN | ❌ Limited | ✅ Required |

## Usage Examples

### Basic Identity Creation

```typescript
import type { Identity, TKeyType } from '@synet/identity-core'

const identity: Identity = {
  alias: 'user-001',
  did: 'did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK',
  kid: 'key-1',
  publicKeyHex: '0x...',
  provider: 'did:key',
  credential: verifiableCredential,
  createdAt: new Date()
}
```

### Multi-Algorithm Key Management

```typescript
import type { IKey, TKeyType } from '@synet/identity-core'

const keys: IKey[] = [
  {
    kid: 'signing-key',
    kms: 'local',
    type: 'Ed25519',
    publicKeyHex: '0x...',
    meta: { algorithms: ['EdDSA'] }
  },
  {
    kid: 'vpn-key',
    kms: 'wireguard',
    type: 'Curve25519',
    publicKeyHex: '0x...',
    meta: { algorithms: ['ECDH'] }
  }
]
```

## Related Packages

- `@synet/identity` - Main identity service implementation
- `@synet/credentials` - Verifiable credentials handling
- `@synet/vault-core` - Secure key storage
- `@synet/patterns` - Common patterns and utilities

## License

MIT

## Contributing

This package contains shared types used across the Synet ecosystem. Changes should maintain backward compatibility and follow semantic versioning.

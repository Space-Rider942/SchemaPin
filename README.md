# SchemaPin 🧷

A cryptographic protocol for ensuring the integrity and authenticity of tool schemas used by AI agents. SchemaPin prevents "MCP Rug Pull" attacks by enabling developers to cryptographically sign their tool schemas and allowing clients to verify that schemas have not been altered since publication.

## Overview

SchemaPin provides a robust defense against supply-chain attacks where benign schemas are maliciously replaced after being approved. The protocol uses:

- **ECDSA P-256** signatures for cryptographic verification
- **SHA-256** hashing for schema integrity
- **Trust-On-First-Use (TOFU)** key pinning for ongoing security
- **RFC 8615** `.well-known` URIs for public key discovery

## Features

- ✅ **Strong Security**: ECDSA P-256 signatures with SHA-256 hashing
- ✅ **Cross-Language Support**: Python and JavaScript implementations
- ✅ **Simple Integration**: High-level APIs for both developers and clients
- ✅ **Key Pinning**: TOFU mechanism prevents key substitution attacks
- ✅ **Standard Compliance**: Follows RFC 8615 for key discovery
- ✅ **Comprehensive Testing**: Full test suite with security validation

```mermaid
flowchart TD
    A[Tool Developer] -->|Publishes| B["/.well-known/schemapin.json (Public Key)"]
    A -->|Signs| C["Tool Schema + Signature"]

    subgraph "AI Agent"
        D["Fetch Schema + Signature"]
        E["Fetch or Cache Public Key"]
        F["Verify Signature"]
        G{"Signature Valid?"}
        H["Accept & Use Tool Schema"]
        I["Reject / Block Tool"]
    end

    C --> D
    B --> E
    D --> F
    E --> F
    F --> G
    G -- Yes --> H
    G -- No --> I
```

## Quick Start

### For Tool Developers (Signing Schemas)

```python
from schemapin.utils import SchemaSigningWorkflow, create_well_known_response
from schemapin.crypto import KeyManager

# Generate key pair
private_key, public_key = KeyManager.generate_keypair()
private_key_pem = KeyManager.export_private_key_pem(private_key)

# Sign your tool schema
workflow = SchemaSigningWorkflow(private_key_pem)
schema = {
    "name": "calculate_sum",
    "description": "Calculates the sum of two numbers",
    "parameters": {
        "type": "object",
        "properties": {
            "a": {"type": "number", "description": "First number"},
            "b": {"type": "number", "description": "Second number"}
        },
        "required": ["a", "b"]
    }
}
signature = workflow.sign_schema(schema)

print(f"Signature: {signature}")
```

### For AI Clients (Verifying Schemas)

```python
from schemapin.utils import SchemaVerificationWorkflow

# Initialize verification
workflow = SchemaVerificationWorkflow()

# Verify schema (auto-pins key on first use)
result = workflow.verify_schema(
    schema=schema,
    signature_b64=signature,
    tool_id="example.com/calculate_sum",
    domain="example.com",
    auto_pin=True
)

if result['valid']:
    print("✅ Schema signature is valid")
    # Safe to use the tool
else:
    print("❌ Schema signature is invalid")
    # Reject the tool
```

## Installation

### Python

```bash
cd python
pip install -e .
```

### JavaScript/Node.js

```bash
cd javascript
npm install
```

### Development Setup

```bash
# Clone repository
git clone https://github.com/thirdkey/schemapin.git
cd schemapin

# Set up Python environment
python3 -m venv .venv
source .venv/bin/activate
pip install -r python/requirements.txt

# Install Python package in development mode
cd python
pip install -e .

# Run Python tests
python -m pytest tests/ -v

# Run JavaScript tests
cd ../javascript
npm test
```

## Examples

### Complete Workflow Demo

```bash
# Run tool developer example
cd python/examples
python tool_developer.py

# Run client verification example
python client_verification.py
```

The examples demonstrate:
- Key pair generation
- Schema signing
- Public key publishing (`.well-known` format)
- Client verification with key pinning
- Invalid signature detection

## Architecture

### Core Components

- **[`SchemaPinCore`](python/schemapin/core.py)**: Schema canonicalization and hashing
- **[`KeyManager`](python/schemapin/crypto.py)**: ECDSA key generation and serialization
- **[`SignatureManager`](python/schemapin/crypto.py)**: Signature creation and verification
- **[`PublicKeyDiscovery`](python/schemapin/discovery.py)**: `.well-known` endpoint discovery
- **[`KeyPinning`](python/schemapin/pinning.py)**: TOFU key storage and management

### Workflow

```mermaid
graph LR
    A[Tool Schema] --> B[Canonicalize]
    B --> C[SHA-256 Hash]
    C --> D[ECDSA Sign]
    D --> E[Base64 Signature]
    
    F[Client] --> G[Fetch Schema + Signature]
    G --> H[Discover Public Key]
    H --> I[Verify Signature]
    I --> J{Valid?}
    J -->|Yes| K[Use Tool]
    J -->|No| L[Reject Tool]
```

## Security

### Cryptographic Standards

- **Signature Algorithm**: ECDSA with P-256 curve (secp256r1)
- **Hash Algorithm**: SHA-256
- **Key Format**: PEM encoding
- **Signature Format**: Base64 encoding

### Schema Canonicalization

Schemas are canonicalized before signing to ensure consistent verification:

1. UTF-8 encoding
2. Remove insignificant whitespace
3. Sort JSON keys lexicographically (recursive)
4. Strict JSON serialization

### Key Pinning

SchemaPin uses Trust-On-First-Use (TOFU) key pinning:

- Keys are pinned on first successful verification
- Subsequent verifications use pinned keys
- Users are prompted before trusting new keys
- Pinned keys are stored securely with metadata

## Protocol Specification

See [`TECHNICAL_SPECIFICATION.md`](TECHNICAL_SPECIFICATION.md) for complete protocol details.

## Implementation Plan

See [`IMPLEMENTATION_PLAN.md`](IMPLEMENTATION_PLAN.md) for development roadmap and architecture decisions.

## Testing

```bash
# Run all tests
cd python
python -m pytest tests/ -v

# Run code quality checks
ruff check .
bandit -r . --exclude tests/

# Run examples
cd examples
python tool_developer.py
python client_verification.py
```

## Project Structure

```
SchemaPin/
├── README.md                          # This file
├── TECHNICAL_SPECIFICATION.md         # Protocol specification
├── IMPLEMENTATION_PLAN.md             # Development plan
├── LICENSE                            # MIT License
├── python/                            # Python reference implementation
│   ├── README.md                      # Python-specific documentation
│   ├── schemapin/                     # Core library
│   │   ├── __init__.py                # Package exports
│   │   ├── core.py                    # Schema canonicalization
│   │   ├── crypto.py                  # Cryptographic operations
│   │   ├── discovery.py               # Public key discovery
│   │   ├── pinning.py                 # Key pinning storage
│   │   └── utils.py                   # High-level workflows
│   ├── tests/                         # Test suite
│   ├── examples/                      # Usage examples
│   ├── requirements.txt               # Dependencies
│   └── setup.py                       # Package configuration
└── javascript/                        # JavaScript implementation
    ├── README.md                      # JavaScript-specific documentation
    ├── package.json                   # NPM package configuration
    ├── src/                           # Core library
    │   ├── index.js                   # Package exports
    │   ├── core.js                    # Schema canonicalization
    │   ├── crypto.js                  # Cryptographic operations
    │   ├── discovery.js               # Public key discovery
    │   ├── pinning.js                 # Key pinning storage
    │   └── utils.js                   # High-level workflows
    ├── tests/                         # Test suite
    └── examples/                      # Usage examples
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests and quality checks
5. Submit a pull request

## License

MIT License - see [`LICENSE`](LICENSE) file for details.

## Security Considerations

- Keep private keys secure and never commit them to version control
- Verify signatures before using any tool schema
- Pin keys on first use and validate key changes
- Use HTTPS for `.well-known` endpoint discovery
- Consider certificate pinning for additional security

## Contact

- **Author**: Jascha Wanger / ThirdKey.ai
- **Email**: jascha@thirdkey.ai
- **Repository**: https://github.com/thirdkey/schemapin

---

**SchemaPin**: Cryptographic integrity for AI tool schemas. Prevent MCP Rug Pull attacks with digital signatures and key pinning.

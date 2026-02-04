## [1.1.0] - 2026-02-04

### 🔐 Security
- Authorization codes are now single-use
- Implemented refresh token expiry
- Added refresh token rotation for improved token lifecycle security

### ⚡ Performance
- RSA keys are now preloaded during application startup to reduce runtime overhead

### 🧹 Maintenance
- Removed deprecated `body-parser` dependency

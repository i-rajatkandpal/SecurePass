# 🔐 SecurePass

> A professional-grade Java library for secure password hashing, built from the ground up with security-first principles.

SecurePass implements **PBKDF2-HMAC-SHA256** with a clean, fluent API — making it trivial to hash and verify passwords correctly, without needing to understand the underlying cryptographic machinery.

---

## ✨ Features

- **PBKDF2-HMAC-SHA256** — OWASP-recommended algorithm with 120,000 iterations by default
- **Cryptographically secure salts** — generated via `java.security.SecureRandom`
- **Constant-time comparison** — prevents timing attacks during verification
- **Fluent Builder API** — fully configurable iterations, salt length, and hash length
- **Modular hash format** — serializable/parse-able `$pbkdf2-sha256$i=...` format
- **Strict input validation** — rejects null, empty, or out-of-range parameters with clear error messages
- **Zero dependencies** — uses only the Java standard library (`javax.crypto`, `java.security`)

---

## � Quick Start

### Hash a password
```java
PasswordHashResult result = SecurePass.hash("myPassword123");
String stored = result.toString();
// → $pbkdf2-sha256$i=120000$<base64-salt>$<base64-hash>
```

### Verify a password
```java
boolean isValid = SecurePass.verify("myPassword123", result);  // true
boolean isWrong = SecurePass.verify("wrongPassword", result);  // false
```

### Custom configuration (Builder API)
```java
PasswordHashResult result = SecurePass.with()
        .iterations(200_000)   // higher = slower = more secure
        .saltLength(32)        // bytes (default: 16)
        .hashLength(64)        // bytes (default: 32)
        .hash("myPassword123");
```

### Serialize & deserialize
```java
// Serialize to string (for storing in DB)
String stored = result.toString();

// Reconstruct from stored string
PasswordHashResult restored = PasswordHashResult.fromString(stored);
boolean valid = SecurePass.verify("myPassword123", restored);  // true
```

---

## 🏗️ Architecture

```
SecurePass              ← Public entry point (Simple API + Builder factory)
SecurePassBuilder       ← Fluent builder for custom configuration
PBKDF2Hasher            ← Core hashing & verification logic
PasswordHashResult      ← Immutable result object with serialization
ValidationUtils         ← Input validation with descriptive error messages
CryptoBasics            ← Utility helpers (SecureRandom, SHA-256, Base64)
```

### Hash Format
```
$pbkdf2-sha256$i=120000$<Base64-salt>$<Base64-hash>
```

This format is inspired by the **Modular Crypt Format (MCF)**, making stored hashes self-describing and forward-compatible.

---

## 🔒 Security Design Decisions

| Concern | Solution |
|---|---|
| Brute-force / dictionary attacks | PBKDF2 with 120,000 iterations (OWASP 2023 minimum) |
| Rainbow table attacks | Unique random salt per password via `SecureRandom` |
| Timing attacks | XOR-based constant-time byte comparison |
| Hash leakage | Defensive copies of byte arrays in `PasswordHashResult` |
| Password in memory | `char[]` cleared with `spec.clearPassword()` after use |

---

## ⚙️ Configuration Defaults & Limits

| Parameter | Default | Min | Max |
|---|---|---|---|
| Iterations | 120,000 | 1,000 | 10,000,000 |
| Salt length | 16 bytes | 8 bytes | 128 bytes |
| Hash length | 32 bytes | 16 bytes | 512 bytes |
| Password length | — | 1 char | 1,000 chars |

---

## 📦 Project Structure

```
Daaju-secure/
├── SecurePass.java          # Main public API
├── SecurePassBuilder.java   # Fluent builder
├── PBKDF2Hasher.java        # Core PBKDF2 implementation
├── PasswordHashResult.java  # Immutable result + serialization
├── ValidationUtils.java     # Input validation
└── CryptoBasics.java        # Crypto utility helpers
```

---

## 🧪 Running Tests

The `SecurePass.main()` method includes a built-in test suite demonstrating:

1. **Simple hash & verify** — correct and incorrect password
2. **Custom config hash** — builder API with 150K iterations
3. **Salt uniqueness** — same password produces different hashes
4. **Performance benchmark** — default vs. custom iteration counts

Run it directly with:
```bash
javac *.java && java SecurePass
```

Expected output:
```
=== SecurePass Test ===

Test 1: Simple hash
Hash: $pbkdf2-sha256$i=120000$...
Verify 'Rajat': true
Verify 'rajat': false

Test 2: Custom config
...

Test 3: Different salts
Are different: true

Test 4: Performance
Default (120K): ~800ms
Custom (200K): ~1300ms
```

---

## 🛣️ Roadmap

- [ ] BCrypt algorithm support (`BCryptHasher implements Hasher`)
- [ ] Algorithm factory & migration utilities
- [ ] Password strength validator (entropy, common password detection)
- [ ] Secure token generator (hex/Base64 random tokens)
- [ ] Maven packaging for library distribution
- [ ] GitHub Actions CI with 85%+ test coverage

---

## 📚 References

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [RFC 2898 — PBKDF2 Specification](https://www.rfc-editor.org/rfc/rfc2898)
- [Java Cryptography Architecture (JCA)](https://docs.oracle.com/en/java/docs/books/security/SecureCodeGuidelines.pdf)

---

## 👤 Author

**Rajat Kandpal** — built as a deep-dive into Java cryptography and security engineering.

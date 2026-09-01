# Implementation Plan - Fix Unresolved Reference 'Pareamento'

The issue is that `Pareamento.kt` contains Python code instead of Kotlin. This causes the Kotlin compiler to fail to resolve the `Pareamento` object/class in `MainActivity.kt`.

## User Review Required

> [!IMPORTANT]
> `Pareamento.kt` was found to contain Python code. I will rewrite it in Kotlin. The original Python code suggests using asymmetric encryption (ECDSA with SECP256R1) for pairing. I will implement this using standard Android security APIs.

## Proposed Changes

### :app Component

#### [MODIFY] [Pareamento.kt](file:///home/taylor/AndroidStudioProjects/PentestLink/app/src/main/java/com/taylor/pentestlink/Pareamento.kt)
- Create a Kotlin `object Pareamento`.
- Implement `gerarOuCarregarChave()` using `KeyPairGenerator` with `secp256r1`.
- Store the generated keys securely (using `EncryptedSharedPreferences` or internal files).
- Implement `obterChavePublicaPem()` to return the public key in standard PEM format.
- Implement `salvarChaveNotebook()` to store the notebook's public key.
- Implement `calcularCodigoConfirmacao()` with the same logic as the Python version (SHA-256 hash of sorted PEMs, first 8 hex chars to int, last 6 digits).

## Verification Plan

### Automated Tests
- Run `./gradlew :app:compileDebugKotlin` to ensure the project builds without the "Unresolved reference" error.

### Manual Verification
- Inspect the logic of `calcularCodigoConfirmacao` to ensure it matches the Python side implementation exactly, as this is critical for the pairing process.

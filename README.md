# CelestialForge  
A Deterministic, Air-Gapped Key Derivation Engine Using Local Physical Entropy and Celestial Anchors on Legacy x86 Hardware

**Author**  
The Architect (with technical contributions from collaborative review)

**Date**  
January 15, 2026

**Version**  
0.2.2

**License**  
CC0 1.0 Universal (Public Domain) — No rights reserved.  
This work is dedicated to the public domain under CC0 1.0 (https://creativecommons.org/publicdomain/zero/1.0/).

## Abstract

CelestialForge is an open, auditable software system that derives 256-bit cryptographic keys from a combination of local physical entropy sources (CPU timing jitter, cache-memory access variance, and cosmic-ray induced single-event upsets) and deterministic celestial references (solar position via fixed-point CORDIC and satellite ephemeris via simplified SGP4 propagation). The system is designed to run entirely air-gapped on surplus 2006–2010 x86 hardware (Intel Core 2 Duo era), with no reliance on internet time, external RNGs, or trusted platform modules. All security-critical paths use constant-time primitives where possible, triple-pass memory shredding with memory barriers, and fixed-point arithmetic to eliminate non-determinism from floating-point units.

The entropy chain is validated using standard statistical test suites (ENT, DIEHARDER subset), achieving >7.999 bits/byte with Chi-square p-values in the 0.01–0.95 range. Key derivation employs PBKDF2-HMAC-SHA256 with dynamic iteration counts modulated by satellite velocity components. The entire codebase compiles natively with Free Pascal, targets legacy hardware with no modern management engine, and enforces a strict "no dynamic allocation" rule in the cryptographic core.

This report describes the design, implementation, threat model, validation results, and deployment considerations. Use cases include off-grid secure storage, air-gapped transaction signing, and legacy hardware revitalization for privacy-focused computing. Code samples are provided throughout for reproducibility.

## Table of Contents

1. [Introduction](#1-introduction)  
   1.1 Motivation: Sovereign Computing in an Untrusted World  
   1.2 Threat Model Overview  
   1.3 Design Goals  
   1.4 Explicit Assumptions & Confidence Levels  
2. [Hardware Platform](#2-hardware-platform)  
   2.1 Rationale for 2006–2010 Dell Latitude (Core 2 Duo)  
   2.2 Sensor Stack (Photodiode, Compass, ADC)  
   2.3 Power Subsystem (Solar + MPPT)  
   2.4 Communications Bridge (BlackBerry OBEX)  
3. [Entropy Sources](#3-entropy-sources)  
   3.1 Timing Jitter (Primary Source)  
   3.2 Cache Thrash Variance (Supplemental Source)  
   3.3 Muon-Induced Single-Event Upsets (Spice Source)  
   3.4 Entropy Mixing & Diffusion  
   3.5 Statistical Validation  
4. [Celestial Root-of-Trust](#4-celestial-root-of-trust)  
   4.1 Solar Position (Fixed-Point CORDIC Algorithm)  
   4.2 Orbital Ephemeris (SGP4 Simplified)  
   4.3 Satellite Salt Generation  
   4.4 Dawn-Cross & Nadir Triggers  
5. [Key Derivation Engine](#5-key-derivation-engine)  
   5.1 PBKDF2-HMAC-SHA256 Implementation  
   5.2 Constant-Time SHA-256 (Audit Results)  
   5.3 Dynamic Iteration Count  
   5.4 Performance Benchmarks  
   5.5 Benchmark Methodology & Reproducibility  
6. [Amnesia & Physical Security](#6-amnesia--physical-security)  
   6.1 Triple-Shred Protocol with mfence Barriers  
   6.2 Tamper Detection  
   6.3 Degraded Mode (GPS Jam/Spoof Fallback)  
7. [Implementation & Testing](#7-implementation--testing)  
   7.1 Build Scripts  
   7.2 Integration Test Results  
   7.3 Assembly Verification  
8. [Limitations & Future Work](#8-limitations--future-work)  
   8.1 Low Muon Rate on 65nm Silicon  
   8.2 GPS Vulnerability  
   8.3 Potential Side-Channel Leakage  
   8.4 Future: RISC-V Port, Argon2 Option  
   8.5 Failure Modes & Recovery  
9. [Off the Grid Network Capability](#9-off-the-grid-network-capability)  
   9.1 Rationale for Air-Gapped Networking  
   9.2 Low-Power Short-Range Protocols  
   9.3 Optical Data Transfer  
   9.4 Multi-Node Synchronization  
   9.5 Security Considerations  
10. [Conclusion & References](#10-conclusion--references)

## 1. Introduction

In an era of pervasive surveillance, centralized trust failures, and supply-chain compromises, traditional cryptographic systems rely too heavily on opaque hardware RNGs, remote attestation, and always-on connectivity. These dependencies create exploitable weaknesses: backdoored firmware, coerced certificates, and remote timing attacks.

CelestialForge addresses this by shifting the root-of-trust to **local physical phenomena** and **publicly verifiable celestial mechanics**. The system generates keys solely from the interaction of the user's hardware with the physical world, eliminating external dependencies. Built on surplus legacy x86 hardware, it leverages the predictability of old silicon for entropy harvesting while avoiding modern backdoors like Intel Management Engine (ME) or AMD Platform Security Processor (PSP).

### 1.1 Motivation: Sovereign Computing in an Untrusted World

Sovereign computing requires keys that are:
- Generated without trusted third parties  
- Resistant to remote spoofing  
- Auditable from source code to binary  
- Reproducible on minimal hardware

**Use case**: An individual in a remote location needs to derive a master key for an air-gapped wallet. Traditional methods (phone RNG, cloud HSM) expose the key to seizure or coercion. CelestialForge uses the sun's position and cosmic rays as anchors, ensuring the key is unique to that time and place.

**Code sample 1.1**: Basic key birth test

```pascal
program simple_key_birth;

uses
  entropy.integrate,
  crypto.keyderive,
  orbital.salt;

var
  Pool: TEntropyPool;
  Salt: TSalt;
  Key: TDerivedKey;
begin
  IntegrateAllEntropy(Pool, 30);  // Harvest 30s entropy
  GetSatelliteSalt(Salt);  
  DeriveKey(Pool, Salt, 200000, Key);  // PBKDF2 with 200k iterations
  // Key is now ready for use
end;

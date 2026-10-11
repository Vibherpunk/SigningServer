## Description

### Summary
Demonstrates the coordinator-side Miniscript approach for hardware wallets that don't natively understand Miniscript (Satochip, TAPSIGNER).

The coordinator (wallet app) parses the Miniscript descriptor, determines the spending path, and constructs the PSBT with the correct witness script. The hardware device just signs the sighash digest — no policy understanding required on the device.

This keeps simple, durable hardware usable in Miniscript wallets instead of restricting Miniscript to devices that fully understand the policy.

### What's included (Reference Only)
This initial draft includes only **Rust reference files** adapted from Bitme, to aid discussion of the pattern:
- `docs/miniscript-reference/descriptor-reference.rs`
- `docs/miniscript-reference/invariants-reference.rs`
- `docs/miniscript-reference/coordinator-pattern.md`
- `docs/miniscript-reference/README.md`

**Note on Bitme vs. Keeper Policies:** The provided Rust references and the sequence diagram in `coordinator-pattern.md` encode Bitme's specific policy (exactly one immediate path, older(N) timelocks, server-side hold/veto). **These are explicitly marked as Bitme-only reference material.** Any implementation for Keeper will strictly use Keeper's actual wallet policy (e.g., 2-of-3 base + Inheritance Key paths). 

### Explicit Inheritance Boundary
It is important to emphasize that the Inheritance Key and the conditions for its spending path belong purely in the **wallet's Miniscript policy**. They are **not** an Inheritance Key approval or timelock implemented by SigningServer. The server's notification, delay, and veto mechanics govern *only the server's own signature*. The server participates in a spend only if the chosen path uses its key. 

### Planned Follow-up Work (Server-Side Scope)
The actual implementation will be focused entirely on missing server cases (app-side coordinator/Satochip implementation will be a separate app PR).
Since `SigningServer` already has Miniscript descriptor handling and PSBT address checks, the planned TypeScript follow-up work will include:
- Focused tests for any missing server cases when dealing with Miniscript descriptors (such as independently recomputing/verifying `witness_script` and `witness_utxo` from a server-owned descriptor to ensure the server isn't simply trusting the PSBT's metadata).
- `src/miniscript/descriptor.ts` and `src/miniscript/invariants.ts`: To perform semantic analysis and formal policy checks (ensuring no single key can spend alone, exactly one immediate path if applicable, etc.) for Keeper's actual policy.
- `tests/miniscript.test.ts`: Runnable invariant tests using Keeper's policy.

### The coordinator pattern
From Bitme's approach:
1. Coordinator parses the Miniscript descriptor for the wallet
2. Coordinator determines which spending path applies
3. Coordinator constructs the PSBT with the correct `witness_script` for that path
4. Hardware device signs the sighash digest (a hash — no Miniscript understanding needed)
5. Coordinator collects signatures and finalizes the transaction

### References
- Technique adapted from [TDH-Labs/Bitme](https://github.com/TDH-Labs/Bitme)
- Related discussion with maintainer on Miniscript coordinator approach

---

## Open questions for maintainer (before full implementation)

1. **Miniscript library:** Which JS/TS library should we use? Options:
   - `bitcoinjs-lib` (has partial Miniscript support)
   - Dedicated Miniscript TS library (if one exists and is audited)
   - FFI to rust-miniscript via WASM?

2. **Directory placement:** New `src/miniscript/` or inside existing `src/wallet/`?

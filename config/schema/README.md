---

## v1-baseline

v1-baseline represents the current, acoustic-only data contract.

It matches the capabilities of the existing Acoustic ML model and includes
only the required fields needed for decision-making based on acoustic events.

Characteristics:
- Acoustic input only
- Event-based
- Time-aware
- No fusion with other modalities
- Intended for current system operation

---

## v1-fusion

v1-fusion represents an optional extension of the same v1 contract.

It preserves full backward compatibility with v1-baseline and adds
support for additional context and modalities.

Characteristics:
- Acoustic input (mandatory)
- Optional human interaction (dictionary-based)
- Optional video observations
- Optional environment / background context
- Intended for future extensions and experimentation

---

## Versioning Notes

- v1-baseline and v1-fusion belong to the same contract version (v1).
- Baseline is the reference contract for current integration.
- Fusion schemas are optional and must not be required by the decision layer.

---

## Design Principles

- Clear separation between ML inference and decision logic
- Strict, explicit data contracts
- Optional extensions without breaking baseline compatibility
- Privacy-aware human interaction
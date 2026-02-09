# Protocol alignment: spec vs prototype

## Spec (WIREFRAME_POC_SPEC.md)

| In scope | Out of scope (for now) |
|----------|------------------------|
| **Radical Prostatectomy**, **EBRT**, **Brachytherapy** | ADT, Watchful Waiting / Active Surveillance |

## Extension plan (NEXT_STEPS.md)

- **Next candidates:** ADT, then **Watchful Waiting**, **Active Surveillance** (different rules: doubling time, rise > 0.75).

## Prototype (index.html)

| Protocol | In app | Filter | Wizard | At least one patient |
|----------|--------|--------|--------|----------------------|
| Radical Prostatectomy (RP) | ✓ | ✓ | ✓ | ✓ (e.g. John Smith, David Brown, Michael Clark, Peter Green, James Taylor, George Walsh, Kevin Shaw) |
| EBRT | ✓ | ✓ | ✓ | ✓ (Alan Jones, Robert Wilson, Terence Bennett, Philip Hayes) |
| Brachytherapy (Brachy) | ✓ | ✓ | ✓ | ✓ (Nicholas Bell, Stephen Webb) |
| Primary ADT | ✓ | ✓ | ✓ | ✓ (Ian Collins) |
| Intermittent hormones | ✓ | ✓ | ✓ | ✓ (David Patel) |
| Watchful waiting | ✓ | ✓ | ✓ | ✓ (Frank Wright) |
| **Active surveillance** | ✓ | ✓ | ✓ | ✓ (one patient added for spec alignment) |

## Alignment summary

- **Spec in-scope (3):** RP, EBRT, Brachy — all implemented and have patients.
- **Spec extension:** ADT ✓, Watchful Waiting ✓, **Active Surveillance** ✓ (added so all spec-mentioned protocols are represented).
- **Extra in app:** Intermittent hormones (not in spec table; kept for clinical coverage).
- **Result:** All protocol cases are visible in Board and List; filter “Protocol” shows each option with at least one row/card. Use **List → All** or **Board** with filter “All protocols” to see every case.

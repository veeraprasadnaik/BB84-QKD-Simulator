<h1 align="center">BB84 Quantum Key Distribution Simulator</h1>

<p align="center">
  <img src="https://img.shields.io/badge/python-3.9%2B-blue.svg" alt="Python">
  <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License">
  <img src="https://img.shields.io/badge/tests-pytest-brightgreen.svg" alt="Tests">
  <img src="https://img.shields.io/badge/protocol-BB84-purple.svg" alt="BB84">
  <a href="../../actions"><img src="../../actions/workflows/ci.yml/badge.svg" alt="CI"></a>
</p>

<p align="center">
A Python simulation of the <b>BB84 protocol</b> (Bennett &amp; Brassard, 1984) that models an
<b>intercept-resend eavesdropper</b>, channel noise, key sifting and <b>QBER-based attack detection</b>.
</p>

---

## Highlights
- Faithful BB84 flow: random bits/bases, sifting, public QBER sample, abort threshold
- Eavesdropper model with adjustable interception probability
- Channel-noise model (bit-flip probability)
- Reproducible runs via seeds; vectorised with NumPy (100k+ qubits in milliseconds)
- CLI, importable library, unit tests, CI, and generated analysis plots

## System Block Diagram
![System architecture](docs/architecture.png)

## Protocol Pipeline
![Protocol pipeline](docs/protocol_flow.png)

### Interactive version (rendered by GitHub)
```mermaid
flowchart LR
    A["Alice<br/>random bits + bases"] -- "qubits (quantum channel)" --> N["Noise<br/>bit-flip p"]
    E["Eve (optional)<br/>intercept-resend"] -. "measures & resends" .-> N
    N --> B["Bob<br/>random measurement bases"]
    A <-. "bases + QBER sample<br/>(classical channel)" .-> B
    B --> S["Sifting"] --> Q{"QBER > 11%?"}
    Q -- Yes --> X["Abort: Eve detected"]
    Q -- No --> K["Shared secret key"]
```

```mermaid
sequenceDiagram
    participant A as Alice
    participant E as Eve
    participant B as Bob
    A->>E: qubits (random bit, random basis)
    E->>B: measured & resent qubits
    B->>B: measure in random bases
    B-->>A: announce bases (classical)
    A-->>B: announce bases (classical)
    Note over A,B: keep positions with matching bases
    A-->>B: reveal random sample, compute QBER
    alt QBER > 11%
        Note over A,B: Abort - eavesdropper detected
    else QBER <= 11%
        Note over A,B: Remaining bits = secret key
    end
```

## Why Eve gets caught
When Eve guesses the wrong basis (50% of the time) she randomises the qubit, so Bob (when his basis matches Alice's) sees an error half the time. Result: **QBER = 25% x (fraction intercepted)**.

![QBER vs Eve](qber_vs_eve.png)

## Installation
```bash
git clone https://github.com/<your-username>/bb84-qkd-simulator.git
cd bb84-qkd-simulator
pip install -e ".[dev]"
```

## Usage
```bash
bb84-sim -n 2000                      # clean channel -> QBER 0%, secure key
bb84-sim -n 2000 --eve 1.0            # full interception -> ~25% QBER, detected
bb84-sim -n 2000 --eve 0.3 --noise 0.02 --seed 42
python examples/qber_vs_eve.py        # regenerate the plot
python docs/make_diagrams.py          # regenerate the block diagrams
```

**Sample output**
```
Qubits sent      : 2000
Sifted key length: 982
Sample compared  : 245
QBER             : 24.90%
Result           : EAVESDROPPER DETECTED - key discarded
```

**Library**
```python
from bb84 import run_bb84
r = run_bb84(n_qubits=5000, eve_prob=0.5, noise=0.01, seed=1)
print(r.qber, r.eve_detected, r.key_length)
```

## Parameters
| Parameter | Description | Default |
|---|---|---|
| `n_qubits` | Qubits Alice sends | 1000 |
| `eve_prob` | Fraction of qubits Eve intercepts | 0.0 |
| `noise` | Channel bit-flip probability | 0.0 |
| `sample_fraction` | Share of sifted key used to estimate QBER | 0.25 |
| `threshold` | Abort if QBER exceeds this | 0.11 |
| `seed` | RNG seed | None |

## Project Structure
```
bb84-qkd-simulator/
├── bb84/                 # core package (simulator + CLI)
├── tests/                # pytest suite
├── examples/             # QBER-vs-Eve analysis
├── docs/                 # block diagrams + generator script
├── .github/workflows/    # CI
├── pyproject.toml
└── README.md
```

## Testing
```bash
pytest -v
```

## Limitations and Roadmap
- [x] Intercept-resend attack, noise, sifting, QBER detection
- [ ] Error correction (Cascade) and privacy amplification
- [ ] Photon-loss and multi-photon (PNS attack) models
- [ ] Decoy-state BB84
- [ ] Streamlit web interface

## References
- C. H. Bennett and G. Brassard, "Quantum cryptography: Public key distribution and coin tossing", 1984
- Gisin et al., "Quantum cryptography", Rev. Mod. Phys. 74, 145 (2002)

## License
MIT. See [LICENSE](LICENSE). Contributions welcome: see [CONTRIBUTING.md](CONTRIBUTING.md).

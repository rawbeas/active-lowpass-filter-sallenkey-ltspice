# Sallen-Key 2nd Order Butterworth Low-Pass Filter

A compact, fully-simulated analog filter project — unity-gain Sallen-Key
topology, 1 op-amp + 2R + 2C, designed for a **1 kHz Butterworth response**.
Simulated end-to-end in ngspice (AC + transient), with an LTspice schematic
included for anyone who wants to open and re-sweep it directly.

![Circuit Schematic](images/circuit_schematic.png)

## Why this circuit

Analog interviewers like filters because the theory is compact but the
follow-up questions go deep fast: order, Q, roll-off, op-amp bandwidth
limitations, topology trade-offs. This project is small enough to fully
understand in an evening, but touches all of that.

## Specifications

| Parameter | Value |
|---|---|
| Topology | Sallen-Key, unity-gain (voltage follower feedback) |
| Order | 2nd order |
| Response | Butterworth (Q = 0.7071, maximally flat passband) |
| Cutoff frequency (fc) | 1 kHz (actual: 995.9 Hz with standard parts) |
| Roll-off | -40 dB/decade past fc |

## Component values

| Component | Value | Role |
|---|---|---|
| R1 | 11.3 kΩ | Input series resistor |
| R2 | 11.3 kΩ | Second series resistor |
| C1 | 20 nF | Feedback capacitor (sets Q together with C2) |
| C2 | 10 nF | Shunt capacitor to ground |
| U1 | Any op-amp (LM358 / TL072 / ideal) | Unity-gain buffer, provides isolation + low output impedance |

Full derivation of these values is in [`docs/design_calculations.md`](docs/design_calculations.md).

## Results

### Bode plot (AC sweep, 1 Hz – 1 MHz)

![Bode Plot](images/bode_plot.png)

- **-3.05 dB at 996 Hz** — matches the -3dB-at-cutoff definition
- **-90.3° phase at fc** — expected for a 2-pole filter at its center frequency
- **-40 dB/decade roll-off** confirmed (-40.07 dB at 10× fc)

### Step response (transient)

![Step Response](images/step_response.png)

~4% overshoot before settling — exactly what you'd expect from Q = 0.707
(slightly underdamped, by design — this is *why* Butterworth looks the way
it does, as opposed to a critically-damped Q=0.5 filter with no overshoot
but a softer knee).

## Repo structure

```
sallen-key-lpf/
├── README.md                          <- you are here
├── schematics/
│   └── sallen_key_lpf.asc             <- LTspice schematic
├── netlist/
│   ├── sallen_key_lpf.cir             <- ngspice netlist (AC sweep)
│   └── step_response.cir              <- ngspice netlist (transient)
├── images/
│   ├── circuit_schematic.png
│   ├── bode_plot.png
│   └── step_response.png
└── docs/
    └── design_calculations.md         <- full hand-derivation of R/C values
```

## How to run it

### Option A — LTspice (what you already have)
1. Open `schematics/sallen_key_lpf.asc` in LTspice.
2. Run (it already has an `.ac dec 200 1 1Meg` directive as a comment —
   add it as a real SPICE directive via **Simulate > Edit Simulation Cmd**
   if it doesn't run automatically).
3. Right-click the output node → you'll get the Bode plot live.
4. Try changing C1/C2 ratio and watch Q (the peaking/overshoot) change —
   this single experiment is the best way to *feel* what Q means before an
   interview.

### Option B — ngspice (used to generate the plots in this repo)
```bash
cd netlist
ngspice -b sallen_key_lpf.cir     # generates bode_data.txt
ngspice -b step_response.cir      # generates step_data.txt
```

## Interview cheat-sheet

| Question | Answer |
|---|---|
| Why active instead of passive filter? | Op-amp gives low output impedance + isolates this stage from whatever loads it — a passive RC would sag under load |
| Why 2nd order instead of 1st? | -40 dB/decade roll-off vs -20 dB/decade — much better rejection of unwanted frequencies close to fc |
| What sets fc? | `fc = 1/(2π·R·√(C1·C2))` — directly from R and C |
| What sets Q (response shape)? | The C1:C2 ratio — `Q = 0.5·√(C1/C2)`. C1=2·C2 gives Butterworth (Q=0.707) |
| Why Butterworth specifically? | Maximally flat passband — no peaking/ripple before the roll-off, unlike Chebyshev |
| What happens if op-amp bandwidth is too low? | Real response deviates from ideal near fc; op-amp's gain-bandwidth product (GBW) must be well above fc |
| How do you convert this to high-pass? | Swap the resistor and capacitor positions in the same topology |
| Why does the step response overshoot? | Q=0.707 is slightly underdamped (critically damped would be Q=0.5, no overshoot) — this is the trade-off Butterworth makes for passband flatness |

## Next steps (if you want to extend this)

- Swap the ideal op-amp for a real SPICE model (e.g. TL072) and watch the
  roll-off bend at high frequency due to finite GBW — great talking point.
- Add Monte Carlo analysis on R/C tolerances to see how much fc and Q drift
  in a real production part (ties directly into TI's "characterization
  across process variation" language).
- Build the high-pass version (swap R↔C) to show both halves of a band-pass
  design.

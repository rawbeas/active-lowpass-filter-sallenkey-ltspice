# Design Calculations — Sallen-Key Butterworth LPF

## 1. Specification
- Filter type: 2nd order, unity-gain, low-pass
- Response: Butterworth (maximally flat passband)
- Target cutoff frequency: **fc = 1 kHz**
- Target Q: **0.7071** (this is what makes it "Butterworth" — any other Q gives peaking or over-damping)

## 2. Topology: Unity-Gain Sallen-Key (equal-R method)

The op-amp is wired as a simple voltage follower (output tied directly to the
inverting input). This keeps the design equations simple because the op-amp's
closed-loop gain is fixed at 1 — all the shaping comes from R1, R2, C1, C2.

For the **equal-resistor** design method (R1 = R2 = R), the standard relations are:

```
fc = 1 / (2*pi*R*sqrt(C1*C2))

Q  = 0.5 * sqrt(C1/C2)
```

## 3. Solving for Q = 0.7071 (Butterworth)

```
Q = 0.5 * sqrt(C1/C2) = 0.7071
=> sqrt(C1/C2) = 1.4142
=> C1/C2 = 2
```

So: **C1 = 2 × C2**. This ratio is what gives the Butterworth response —
it's worth memorizing, since an interviewer may ask "why is C1 twice C2?"

## 4. Picking real component values

Start from a convenient capacitor value and work backward to R (capacitors
have fewer standard values than resistors, so fix C first):

```
C2 = 10 nF
C1 = 2 * C2 = 20 nF

R = 1 / (2*pi*fc*sqrt(C1*C2))
  = 1 / (2*pi*1000*sqrt(20e-9 * 10e-9))
  = 11254 Ω
```

Rounded to the nearest standard (E96) resistor value:

```
R1 = R2 = 11.3 kΩ
```

## 5. Verifying the rounded values

Plugging 11.3 kΩ back in:

```
fc = 1 / (2*pi * 11300 * sqrt(20e-9 * 10e-9)) = 995.9 Hz   (≈1 kHz, 0.4% off — fine)
Q  = 0.5 * sqrt(20n/10n) = 0.7071                          (exact — Q only depends on the C ratio)
```

## 6. Final component values

| Component | Value |
|---|---|
| R1 | 11.3 kΩ |
| R2 | 11.3 kΩ |
| C1 (feedback cap) | 20 nF |
| C2 (shunt cap) | 10 nF |
| Op-amp | Any general-purpose op-amp (LM358, TL072, OP07, or ideal `opamp` in LTspice) |

## 7. What simulation should confirm

| Check | Expected result |
|---|---|
| Magnitude at fc | **-3 dB** (by definition of cutoff) |
| Phase at fc | **-90°** (2nd order filter, right at the pole pair's center) |
| Roll-off far above fc | **-40 dB/decade** (two poles × -20 dB/decade each) |
| Step response | Slight overshoot (~4%), since Q=0.707 is just above critically-damped (Q=0.5) |

All four of these were confirmed by the ngspice simulation in this repo —
see `images/bode_plot.png` and `images/step_response.png`.

## 8. If you change the design later

- **Want a different fc?** Scale R (keep the C1:C2 = 2:1 ratio) — R is inversely
  proportional to fc.
- **Want a different response shape (Chebyshev, Bessel)?** Change the C1:C2
  ratio — that ratio is what sets Q, and Q is what sets the response shape.
- **Want gain > 1?** Switch from the voltage-follower feedback to a resistive
  divider in the feedback path (this changes the design equations — equal-R
  method above only holds for unity gain).

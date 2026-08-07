
# SPITE 0.1.0
### Scattering Parameter Inspection Tool for Engineers

*"The secret ingredient is SPITE."*

---

## 1. Introduction

SPITE is a lightweight single-file Python library for RF/microwave network analysis, 
built so undergraduate students can learn microwave engineering and run simulations 
outside of class or internships without needing expensive, license-gated software.

SPITE is built around the Network class, which represents RF networks using 
frequency data, scattering parameters, reference impedance, and schematic information. 
Python operators are used for 1-port and 2-port network composition:
`@` for cascading and `**` for repeating networks


## 2. Features

- The entire library is in one file
- Conversion between S, Z, Y, and ABCD parameters in both directions
- Schematics for 1-port and 2-port networks
- Cascade 2-port and 1-port networks with `@`
- Repeat a 2-port network with `**`
- Linear, dB, and Smith chart plots for S, Z, and Y parameters
- Time-domain gating
- Touchstone file import and export
- Very important functions such as `esqueleto()` and `merendola()`

## 3. Limitations

Features that haven't been added yet

- Frequency dependent / complex reference impedance
- General N-port topology / circuit connection
- Components such as waveguides, microstrips, etc.
- Noise factor as a network parameter

Try using [scikit-rf](https://scikit-rf.org/) for more advanced RF engineering.

## 4. Installation

SPITE depends on NumPy and Matplotlib. Nothing else.

```bash
pip install spite
```

## 5. Examples

### Quarter-wave matching network

```python
import numpy as np
import spite as sp

f = np.linspace(0.1, 4e9, 100)
matching_network = sp.net2p_tline_series(f=f, f0=1e9, EL_deg=90, Zc=np.sqrt(50 * 75)) @ sp.net1p_R(f=f, R=75)

matching_network.plot_schematic()
matching_network.plot_dB()
```

A quarter-wave transformer of `Zc = sqrt(Z0 * R_L)` matches a 75Ω load to a 50Ω
system exactly at f0 — you can see it in the resulting S11 dip.

### Lumped-element low-pass filter

```python
import numpy as np
import spite as sp

f = np.linspace(0.1, 4e9, 100)
filt = sp.net2p_L_series(f, 3.9785e-9) @ sp.net2p_C_shunt(f, 3.184e-12) @ sp.net2p_L_series(f, 3.9785e-9)

S11_at_2GHz = filt.sample(2e9, param="S", port_indices=(1, 1))
print(20 * np.log10(np.abs(S11_at_2GHz)))

filt.plot_dB(floor=-30, port_indices=[(1, 1), (2, 1)])
filt.plot_smith(port_indices=[(1, 1), (2, 1)], chart_type="Y")
```

This exact filter was cross-checked against an AWR Microwave Office simulation
of the same design. SPITE's predicted response matched to within 0.01 dB at
the 2 GHz cutoff.

### RLGC ladder

```python
import numpy as np
import spite as sp

f = np.linspace(0.1, 4e9, 100)
R, L, G, C = 1.5, 5e-9, 3e-4, 2e-12

ladder = sp.net2p_RLGC(f, R, L, G, C) ** 3
ladder.plot_schematic()
ladder.plot_dB(port_indices=(2, 1))
ladder.plot_smith()
```

## 6. License and Citation

MIT License. Use it however you want.

But if you want to be nice, please put my name somewhere. I don't care about the format. 
SPITE is above citation formats because it is cool.

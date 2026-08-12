
# SPITE 0.1.1
### Scattering Parameter Inspection Tool for Engineers

*"The secret ingredient is SPITE :D"*

---

## 1. Introduction

SPITE is a lightweight single-file Python library for RF/microwave network analysis, 
built so undergraduate students can learn microwave engineering and run simulations 
outside of class or internships without needing expensive, license-gated software.

SPITE is built around the `Network` class, which represents RF networks using 
frequency data, scattering parameters, reference impedance, and schematic information. 
Python operators are used for 1-port and 2-port network composition:
`@` for cascading and `**` for repeating networks.


## 2. Features

- The entire library is in one file
- Conversion between S, Z, Y, and ABCD parameters in both directions
- Lumped R, L, C elements (1-port and 2-port)
- Ideal transmission line elements (1-port and 2-port)
- Schematics for 1-port and 2-port networks
- `@` for cascading 2-port and 1-port networks
- `**` for repeating a 2-port network 
- Linear, dB, and Smith chart plots for S, Z, and Y parameters
- Time-domain gating
- Touchstone file import and export
- Very important functions such as `esqueleto()` and `merendola()` (for reasons beyond microwave engineering)

## 3. Limitations

Features not yet implemented

- Frequency dependent / complex reference impedance
- General N-port topology / circuit connection
- Components such as waveguides, microstrips, etc.
- Noise factor as a network parameter / property

Try using [scikit-rf](https://scikit-rf.org/) for more advanced RF engineering.

## 4. Installation

SPITE depends on NumPy and Matplotlib. Nothing else.

```bash
pip install spite
```
Then start your Python script or notebook with: 

```python 
import numpy as np
import spite as sp
```
SPITE imports Matplotlib internally. So importing Matplotlib again is 
unnecessary unless you want to make custom plots.

## 5. Examples

### Quarter-wave matching network

```python
f = np.linspace(0.1, 4e9, 100)  # frequency sweep

# Zc = sqrt(Z0*R) is the quarter-wave matching condition
matching_network = (
    sp.net2p_tline_series(f=f, f0=1e9, EL_deg=90, Zc=np.sqrt(50 * 75))
    @ sp.net1p_R(f=f, R=75)  # terminates the network with a 75-ohm load
)

matching_network.plot_schematic()  # horizontal stack of SYM2P_TLINE_SERIES + SYM1P_R
matching_network.plot_dB()  # S11 in dB
```

A quarter-wave transformer of `Zc = sqrt(Z0 * R_L)` matches a 75Ω load to a 50Ω
system exactly at `f0`.

### RLGC ladder

```python
f = np.linspace(0.1, 4e9, 100)  # frequency sweep
R, L, G, C = 1.5, 5e-9, 3e-4, 2e-12  # R, L, G, C values of one section

ladder = sp.net2p_RLGC(f, R, L, G, C) ** 3  # ** repeats the section 3 times

ladder.plot_schematic()
ladder.plot_dB(port_indices=(2, 1))  # S21 in dB
ladder.plot_smith()  # color gradient encodes frequency
```

### Lumped-element low-pass filter

```python
f = np.linspace(0.1, 4e9, 100)  # frequency sweep

filter = (
    sp.net2p_L_series(f, 3.9785e-9)
    @ sp.net2p_C_shunt(f, 3.184e-12)
    @ sp.net2p_L_series(f, 3.9785e-9)  
)

S11_at_2GHz = filter.sample(2e9, param="S", port_indices=(1, 1))
print(20 * np.log10(np.abs(S11_at_2GHz)))  # -3.0127 dB 

filter.plot_dB(floor=-30, port_indices=[(1, 1), (2, 1)])
filter.plot_smith(port_indices=[(1, 1), (2, 1)], chart_type="Y")  # Admittance charts are available as well
```

This exact filter was cross-checked against an AWR Microwave Office simulation
of the same design. SPITE's predicted response matched to within 0.01 dB at
the 2 GHz cutoff.


## 6. License and Citation

SPITE is released under the [0BSD License](https://opensource.org/license/0bsd)

- You can use, modify, and share SPITE freely.
- Attribution is appreciated but not required.
- The software comes as is, with no warranty.

A bibtex is provided below for those who would like to use it:

```bibtex
@software{spite,
  author  = {Lee, Anthony Jeongseok},
  title   = {SPITE: Scattering Parameter Inspection Tool for Engineers},
  year    = {2026},
  url     = {https://github.com/anthonyjlee377/spite},
  version = {0.1.1}
}
```



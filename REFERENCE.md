# SPITE Function and Constant Reference

## Matrix format checks and conversions

#### `check(S, n=None)`
Validates that `S` has the required `(Nf, Nports, Nports)` shape. If `n` is given, also enforces that exact port count.
- **S**: `(Nf, Nports, Nports)`, complex array
- **n**: `int`, required port count, optional
- **returns**: `S`, unchanged, if valid; raises `ValueError` otherwise

#### `check_network(f, S, n=None)`
Same validation as `check`, but also confirms `f` and `S` agree in length.
- **f**: scalar or `(Nf,)` array-like, frequency in Hz
- **S**: `(Nf, Nports, Nports)`, complex array
- **n**: `int`, required port count, optional
- **returns**: `(f, S)`, unchanged, if valid

#### `S_to_Z(S, Z0=50)` / `Z_to_S(Z, Z0=50)`
#### `S_to_Y(S, Z0=50)` / `Y_to_S(Y, Z0=50)`
#### `S_to_ABCD(S, Z0=50)` / `ABCD_to_S(ABCD, Z0=50)`
Convert between S, Z, Y, and ABCD representations. `S_to_ABCD`/`ABCD_to_S`
are only defined for 2-port networks.
- **S / Z / Y / ABCD**: `(Nf, Nports, Nports)`, complex array
- **Z0**: reference impedance, float or array-like `(Nports,)`, default 50Ω
- **returns**: the converted matrix, same shape as the input

#### `ZY_to_propagation(f, Z, Y)` / `RLGC_to_propagation(f, R, L, G, C)`
Convert per-unit-length series impedance and shunt admittance, or RLGC parameters, into the propagation constant and characteristic impedance.
- **f**: scalar or `(Nf,)` array-like, frequency in Hz
- **Z, Y**: scalar or array-like matching f, per-unit-length series impedance (Ω/m) and shunt admittance (S/m)
- **R, L, G, C**: scalar or array-like matching f, per-unit-length resistance (Ω/m), inductance (H/m), conductance (S/m), and capacitance (F/m)
- **returns**: `gamma`, `Zc`, where gamma is the complex propagation constant in 1/m and Zc is the complex characteristic impedance in Ω
  
---

## Symbols and schematics

#### Constants
- `DEFAULT_FIGSIZE = (12, 4)`: default matplotlib figure size for schematics
- `DEFAULT_HEIGHT = 64`: default pixel height for schematics 

#### Component symbols (bitmap arrays)
```
SYM2P_R_SERIES,  SYM2P_R_SHUNT,  SYM1P_R,
SYM2P_L_SERIES,  SYM2P_L_SHUNT,  SYM1P_L,
SYM2P_C_SERIES,  SYM2P_C_SHUNT,  SYM1P_C,
SYM2P_RLGC,
SYM2P_TLINE_SERIES,
SYM2P_TLINE_SHUNT_SHORT, SYM1P_TLINE_SHORT,
SYM2P_TLINE_SHUNT_OPEN,  SYM1P_TLINE_OPEN,
SYM2P_CHANNEL, SYM1P_ANTENNA,
SYM2P_BLACKBOX, SYM1P_BLACKBOX
```
Each is a 2D NumPy array (0s and 1s) representing that component's schematic
symbol.

#### `make_bitmap(string_layout)`
Converts a text block of `E` (pixel on) and `.` (pixel off) characters into
a clean binary 2D NumPy array. 
- **string_layout**: `str`, rows of `E`/`.` characters
- **returns**: 2D NumPy array of 0s and 1s

#### `plot_schematic(schematic, figsize=DEFAULT_FIGSIZE, cmap="gray")`
Renders a bitmap array as an image.
- **schematic**: 2D array-like (bitmap)
- **figsize**: `(width, height)` tuple
- **cmap**: matplotlib colormap name
- **returns**: `matplotlib.axes.Axes`

#### `resize_schematic(schematic, new_height=DEFAULT_HEIGHT)`
Resizes a bitmap array to a target pixel height, preserving aspect ratio.

- **schematic**: 2D array-like
- **new_height**: `int`
- **returns**: resized 2D array

---

## `Network(f, S, Z0=50, schematic=np.array([[0]]))`

- **f**: scalar or `(Nf,)` array-like, frequency in Hz
- **S**: `(Nf, Nports, Nports)` complex array
- **Z0**: reference impedance, float or array-like `(Nports,)`, default 50Ω
- **schematic**: 2D array-like bitmap, shown by `plot_schematic()`

**Attributes**: `self.f`, `self.S`, `self.Z0`, `self.schematic`

### Properties
- **`Z`**: Z-parameters, computed via `S_to_Z(self.S, self.Z0)`
- **`Y`**: Y-parameters, computed via `S_to_Y(self.S, self.Z0)`

### Plotting methods
- **`plot_schematic(figsize=DEFAULT_FIGSIZE, cmap="gray")`**
- **`plot_lin(param="S", port_indices=(1,1), cmap="plasma", ax=None)`** 
- **`plot_dB(floor=-40, param="S", port_indices=(1,1), cmap="plasma", ax=None)`**
- **`plot_phase(param="S", port_indices=(1, 1), deg=True, unwrap=False, cmap="plasma", ax=None)`**
- **`plot_group_delay(port_indices=None, cmap="plasma", ax=None)`**
- **`plot_smith(port_indices=(1,1), chart_type="Z", cmap="plasma", ax=None)`** : `chart_type` is `"Z"` (impedance) or `"Y"` (admittance)
- **`plot_ifft(port_indices=(1,1), cmap="plasma_r", ax=None)`**

 `port_indices` accepts either a single `(i, j)` tuple or a list of tuples.

### Data methods
- **`sample(f_eval, param="S", port_indices=None)`**:interpolate any parameter (`"S"`, `"Z"`, `"Y"`, or `"ABCD"` for 2-ports) at arbitrary frequency point(s)
  - **f_eval**: float or array-like, frequencies to evaluate at
  - **returns**: interpolated value(s), shape depends on `port_indices`

- **`gate(t_start, t_stop, alpha=0.05, trim=True)`**: applies a Tukey frequency window, IFFT, time gate over `[t_start, t_stop]`, then FFTs back
  - **returns**: a new `Network` with the gated result

- **`write_touchstone(filepath)`**: saves this network to a Touchstone file (`.sNp`)

### `Network1Port(Network)`
-Adds shortcut methods: **`S11`**, **`Z11`**, **`Y11`** 
-Each method optionally takes `f_eval=None`.

### `Network2Port(Network)`
-Adds the **`ABCD`** property (`S_to_ABCD(self.S, self.Z0)`), 
-Shortcut methods: Matrix entries for S, Z, and Y parameters such as **`S11`**, **`Z11`**, **`Y11`** etc.
-Each method optionally takes `f_eval=None`.

### Operators
- **`@` (`__matmul__`)**
  - `2-port @ 2-port`: cascades the two networks (ABCD matrix multiplication)
  - `2-port @ 1-port`: terminates port 2 in the 1-port load, returns a 1-port network
- **`**` (`__pow__`)**
  - `2-port ** n`: cascades `n` copies of the same network

### Import
#### `read_touchstone(filepath)`
Parses a `.sNp` Touchstone file and returns the appropriate subclass
(`Network1Port`, `Network2Port`, or the base `Network` for N>2 ports).
- **filepath**: `str`
- **returns**: `Network` (or subclass)

---

## Basic lumped components

```
net2p_R_series(f, R, Z0=50.0), net2p_R_shunt(f, R, Z0=50.0), net1p_R(f, R, Z0=50.0),
net2p_L_series(f, L, Z0=50.0), net2p_L_shunt(f, L, Z0=50.0), net1p_L(f, L, Z0=50.0),
net2p_C_series(f, C, Z0=50.0), net2p_C_shunt(f, C, Z0=50.0), net1p_C(f, C, Z0=50.0)
```
Ideal series/shunt/termination R, L, or C elements.
- **f**: scalar or `(Nf,)` array-like, frequency in Hz
- **R / L / C**: float or array-like matching `f`'s shape (ohms / henries / farads)
- **Z0**: reference impedance, float or array-like `(Nports,)`, default 50Ω
- **returns**: `Network2Port` (series/shunt) or `Network1Port` (termination)

#### `net2p_RLGC(f, R, L, G, C, Z0=50.0)`
Textbook RLGC unit-cell ladder: series R, series L, shunt G, shunt C,
cascaded in that order. 
- **R, L, G, C**: float or array-like with respect to `f` (Ω, H, S, F)
- **returns**: `Network2Port`

#### `net2p_ZY(f, Z, Y, Z0=50.0)`
Generic series impedance `Z` followed by shunt admittance `Y`. 
- **Z, Y**: complex, float, or array-like with respect to `f`
- **returns**: `Network2Port`

---

## Transmission line components

```
net2p_tline_series(f, f0, EL_deg, Zc, Z0=50.0), net2p_tline_lossy_series(f, gamma, length, Zc=50.0, Z0=50.0),
net2p_tline_shunt_open(f, f0, EL_deg, Zc, Z0=50.0), net2p_tline_lossy_shunt_open(f, gamma, length, Zc=50.0, Z0=50.0),
net2p_tline_shunt_short(f, f0, EL_deg, Zc, Z0=50.0), net2p_tline_lossy_shunt_short(f, gamma, length, Zc=50.0, Z0=50.0),
net1p_tline_open(f, f0, EL_deg, Zc, Z0=50.0), net1p_tline_lossy_open(f, gamma, length, Zc=50.0, Z0=50.0),
net1p_tline_short(f, f0, EL_deg, Zc, Z0=50.0), net1p_tline_lossy_short(f, gamma, length, Zc=50.0, Z0=50.0)
```

- **f**: scalar or `(Nf,)` array-like, frequency in Hz
- **f0**: `float`, the frequency at which `EL_deg` is defined
- **EL_deg**: `float`, electrical length in degrees, at `f0`
- **gamma**: `complex` or array-like with respect to `f`, propagation constant in 1/m
- **length**: `float`, physical length of a lossy line, meters
- **Zc**: `complex` or array-like with respect to `f`, characteristic impedance of the line, ohms
- **Z0**: reference impedance, float or array-like `(Nports,)`, default 50Ω
- **returns**: `Network2Port` (series/shunt-loaded through-line) or `Network1Port` (bare stub)

---

## Very important functions

#### `esqueleto()`, `merendola()`
:D 

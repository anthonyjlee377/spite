# Changelog

## [0.1.2] - 2026-10-03
### Added
- `net1p_Z` and `net2p_ZY` as backends for network functions
- helper functions `ZY_to_propagation` and `RLGC_to_propagation` 
- Lossy transmission line networks
- Phase response and group delay plots

### Changed
- All series/shunt R, L, C 2-port functions now use `net2p_ZY` as backend
- All 1-port R, L, C and tline functions now use `net1p_Z` as backend
- `_prep_component` refactored: complex dtype, f-first argument order `(f, val, name)`
- `plot_time_domain` is renamed to `plot_ifft`
- `check` and `check_network` now use Nports for explicit port-count validation; check_network now also requires strictly increasing frequencies

### Fixed
- Time and frequency plot units now autoscale; units were previously hardcoded to ns and GHz
- Fixed port-pair indexing for multi-port IFFT plots

## [0.1.1] - 2026-08-13
### Fixed
- Added missing `import os` in `__init__.py`
- Added changelog and reference
- Updated license
  
## [0.1.0] - 2026-08-06
### Added
- Initial release
- Matrix format checks and conversions
- S-parameter network classes
- Two-port network cascading and termination
- Linear, dB, and Smith chart plots 
- Lumped R, L, C elements (1-port and 2-port)
- Ideal transmission line elements (1-port and 2-port)
- Touchstone I/O 
- Easter eggs


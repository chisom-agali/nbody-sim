# N-Body Gravity Simulation + CR3BP Extra Credit

Canales challenge: model gravitational interactions between a few bodies as a system of ODEs, integrate numerically, visualize in 3D, and export a video. Extra credit applies the same approach to a spacecraft in the Earth-Moon circular restricted three-body problem (CR3BP).

## Requirements

- Python 3 with `numpy`, `scipy`, `matplotlib`
- `ffmpeg` installed and on your PATH (used to export the `.mp4` animations)
- 
## Running it

This runs the base N-body case (currently set to the stable orbit condition) and the CR3BP extra credit case, and writes four files into the same folder:

- `nbody_static_check.png` — static 3D check plot, base challenge
- `nbody_simulation.mp4` — animated video, base challenge
- `cr3bp_static_check.png` — static 3D check plot, CR3BP
- `cr3bp_simulation.mp4` — animated video, CR3BP

## Switching between initial conditions

The base N-body case (`run_n_body()`) supports three demo conditions. All three sets of values are kept as a commented block near the top of the file, ready to copy-paste in. Only one condition runs per script execution, so to try a different one:

1. In `run_n_body()`, replace the `masses`, `pos0`, and `vel0` lines with the values for the condition you want (below).
2. **Stable orbit only:** leave the `T` calculation as-is (the four lines computing `r_12`, `v_circular`, `period_est`, `T`), since that math derives the real orbital period from those specific masses/positions. **Escape and chaotic:** delete those four lines and replace them with a single `T = 6e6` instead — the period-based calculation only makes sense for the stable-orbit setup, not these.
3. If you changed `T` as in step 2, also update the two `print(...)` lines just below `final_pos = positions_over_time[-1]` — drop the `period_est` references (they won't exist anymore) and just print `T` directly.
4. Rename the output filenames near the bottom of the function (`fig_static.savefig(...)` and `anim.save(...)`) so each condition's files don't overwrite the last one, e.g. `nbody_escape_simulation.mp4` instead of `nbody_simulation.mp4`.
5. Run the script again.

### Condition 1 — stable orbit (default, already in the file)

```python
masses = np.array([5e24, 7e22, 1e22])
pos0 = np.array([[0, 0, 0], [4e8, 0, 0], [0, 6e8, 1e7]], dtype=float)
vel0 = np.array([[0, 0, 0], [0, 1000, 0], [-800, 0, 0]], dtype=float)
```
Velocities tuned so the bodies stay gravitationally bound; the trajectory closes on itself. Keep the automatic `T` calculation for this one.

### Condition 2 — escape trajectory

```python
masses = np.array([5e24, 7e22])
pos0 = np.array([[0, 0, 0], [4e8, 0, 0]], dtype=float)
vel0 = np.array([[0, 0, 0], [0, 1680, 0]], dtype=float)
T = 6e6
```
Initial speed pushed past escape velocity (circular speed here is ~913 m/s, escape speed ~1292 m/s), so the second body flies off instead of orbiting.

### Condition 3 — chaotic three-body close encounter

```python
masses = np.array([2e24, 2.1e24, 1.8e24])
pos0 = np.array([
    [2e8, 0, 0],
    [-1e8, 1.732e8, 0],
    [-1e8, -1.732e8, 0]
], dtype=float)
vel0 = np.array([
    [0, 50, 0],
    [-40, -20, 0],
    [30, -10, 0]
], dtype=float)
T = 6e6
```
Slightly unequal masses and small velocity nudges (a perfectly symmetric triangle at rest is actually a known special case that stays symmetric, not chaotic, so the asymmetry here is deliberate). Produces a genuine close encounter — bodies swing in tight, then at least one gets flung outward.

## Two-body sanity check (optional)

Before generalizing to N bodies, `run_two_body_sanity()` checks the simplest case against a known answer: a circular two-body orbit should stay closed, and comparing `solve_ivp`'s default tolerances against tighter ones (`rtol=1e-9, atol=1e-6`) shows the orbit visibly drifting under the default tolerance and holding steady under the tighter one.

This function is defined but not called automatically from `__main__`, since it's a standalone check rather than part of the main pipeline. To run it, uncomment the three lines near the bottom of the file under `if __name__ == "__main__":`.

## CR3BP extra credit

`run_cr3bp()` runs a massless spacecraft in the Earth-Moon rotating frame, starting at `state0 = [0.8, 0, 0, 0, 0.2, 0]` (nondimensional units, `mu = 0.0123`). The derivative function (`cr3bp_deriv`) adds Coriolis (velocity-dependent) and centrifugal (position-dependent) terms on top of plain two-body gravity to account for working in a frame that rotates with the Earth-Moon system.

## Limitations

- **Solver tolerance vs. cost:** tightening `solve_ivp`'s `rtol`/`atol` reduces energy drift but costs more function evaluations. For a project at this scale that trade-off is cheap, but it's a real design decision for an actual mission-design tool.
- **O(N²) scaling:** `total_accelerations` loops over every pair of bodies, so the cost grows as the square of the number of bodies. Fine for a handful of bodies (a few pairwise checks), but at scale — say 10,000 bodies — that's 100 million pairwise calculations per time step, which gets slow fast. A real large-N simulation would need a smarter method, such as grouping distant bodies together (e.g. a Barnes-Hut tree) instead of checking every single pair.
- **CR3BP simplifications:** this model assumes a perfectly circular Earth-Moon orbit and a massless spacecraft. Real mission design accounts for the Moon's actual elliptical orbit and other perturbations (solar gravity, radiation pressure, etc.).
- **Further research:** extending this toward real mission trajectory design (how actual lunar missions use Lagrange points and rotating-frame dynamics to plan transfer paths), comparing `solve_ivp`'s adaptive stepping against a simpler fixed-step integrator, and building out a faster force calculation so this could scale past a handful of bodies.

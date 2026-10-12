# week4_240063: PID control with two light sensors (TRIK Studio)

A TRIK Studio project (`week4_240063.qrs`) for the TRIK robot. A PID controller keeps the robot on course using two light sensors, and it drives the two motors with a base speed of 30.

## How it works

1. **Start (runs once).** The first Function block stores the sensor values as the zero position (`left`, `right`) and sets the PID gains and starting values.
2. **Loop.** The second Function block computes the control signal `u`:
   - `err = (sensorA1 - left) - (sensorA2 - right)`, how far the sensors moved from the zero position
   - `u = K_p * err + K_i * integral + K_d * (err - last_error) / dt`
   - the integral is limited to -60..60 (anti-windup) and `u` is limited to -30..30 (`maxCorrection`)
3. **Motors.** M3 gets `30 - u` and M4 gets `30 + u`.
4. **Delay.** A 5 ms timer, then the loop repeats.

## Hardware

| Part | Port |
|---|---|
| Light sensor (left) | A1 (`sensorA1`) |
| Light sensor (right) | A2 (`sensorA2`) |
| Motor | M3, power `30 - u` |
| Motor | M4, power `30 + u` |

## Parameters (tuned on the real robot)

| Name | Value | Meaning |
|---|---|---|
| `K_p` | 1.5 | reaction to the current error |
| `K_i` | 0.06 | removes slow constant drift |
| `K_d` | 7.0 | damps wobble |
| `dt` | 10 | fixed time step used only by the D term |
| `maxCorrection` | 30 | limit for `u` |

## Important notes

- Start the program with the robot in its neutral position, because the zero position is measured at the start.
- Keep `maxCorrection <= 30`. A larger value makes `30 - u` or `30 + u` negative and one motor would reverse.
- `K_d = 7.0` was tuned together with `dt = 10` and the 5 ms delay. Re-test the robot if you change either one.
- Tune one gain at a time, in this order: `K_p`, then `K_d`, then `K_i`.
- TRIK has no `min()`/`max()`, so the limits use `(abs(x + M) - abs(x - M)) / 2`, which clamps `x` to -M..M.
- Everything is built from Function blocks, so the project needs no Python or JavaScript code.

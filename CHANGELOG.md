# v2.0 — averaged motion diagnostics

First numbered release, based on the trace-free SR/NSR benchmark update. Subsequent releases use v2.1, v2.2 and so on, each with its own Firmware, Documentation, changelog and hash manifest. The source banner reports 2.0.

- Sample raw target-velocity readback, actual velocity, Iq (torque-producing current), Id (flux current) and actual position nominally every 20 ms. Accumulate signed 64-bit sums, and print integer arithmetic averages every 2000 ms.
- `MOTION_DEBUG_SAMPLE_INTERVAL_MS` and `MOTION_DEBUG_INTERVAL_MS` configure sampling and logging; log interval 0 compiles out the averaging and output. Valid enabled range: sample >=5 ms, sample <=log <=60000 ms.
- Keep register reads in the motor task and UART output in the debug task. A single latest-average mailbox replaces a logging queue. No new task or heap allocation. Stops and direction changes discard pending/partial windows. Failed reads are excluded and increment `diagnostic_error_count`.
- Keep separate `SR_CW_TORQUE_LIMIT_RAW` and `SR_CCW_TORQUE_LIMIT_RAW` for G/V/W/X/Y, including the CW reference leg when Y needs reference recovery. Zero still selects the profile's validated running limit. No physical torque/current conversions are invented.
- Name all numeric fields of the A/B/C motor profiles in `actuator_profile.h`. Replace SR stop retry/timing and angle/percentage scaling literals with shared named constants. Keep structural zero/one values, enum ordinals and register encodings where naming each literal would only add redundant macros. Remove `ACTUATOR_SOFT_STALL_ENABLE` in favor of existing `ENABLE_SOFT_STALL`.
- Separate application documentation from Firmware. Older documentation and editing scripts are archived; vendor documentation and license files stay with their dependencies. Validation scripts remain with the firmware because they are required to reproduce tests.

Validation: dedicated averaging regression; SR A/B/C and stall-policy tests; directional torque override regression; NSR feature matrix; ARM GCC syntax/application-object checks. Control suites compile diagnostics out for isolation; the dedicated averaging test and default-config ARM syntax checks exercise diagnostics enabled. Hardware timing, current calibration, and IAR target linking remain unverified.

No new call tracing. Preserve the previous zero-IDLE stop, learned-span retention, V/W reversal, unbounded W until Z/protection, and CW re-reference before count-based Y when necessary.

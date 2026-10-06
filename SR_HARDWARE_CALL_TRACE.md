# Spring-return hardware test: complete source walkthrough

This guide explains the supplied B → G → X → Y (rejected) → Z → Y test against the current Act source. It distinguishes events visible in the photographs from paths reconstructed from source. It does not claim that the photographs capture every call or variable value. No firmware behavior was changed and no compilation was performed for this guide.

## 1. What the photographs establish

| Action | Evidence | Meaning |
|---|---|---|
| B | `Q put 0`, `Q get 0`, `SR.select_profile`, state 3 | Profile selection reaches the motor task; configuration completes and the controller is ready but initially uncalibrated. |
| G | `Q put 5`, state 1 → 2 → 3, `SR CAL COMPLETE` | CW endpoint detection and count-based CCW return complete successfully in software. |
| X | `Q put 6`, `SR.begin`, state 4 | Calibrated CW motion starts. |
| First Y | Busy message, `Y cmd rx`, no corresponding `Q put 7` | The debug task rejects Y while SR remains in state 4. It is not a queue-full failure. |
| Z | `SV.request_abort`, `SR.RequestStop`, `Q clr`, `M.StopHold`, state 3 | Stop request bypasses the queue; the motor task performs HOLD and returns to ready. |
| Second Y | `Q put 7`, `Q get 7`, state 5 → 3 | CCW starts, then the controller returns to HOLD/ready. The picture alone does not provide the elapsed travel time or travel distance. |

Z is **not normally required after a completed X**. X should detect its endpoint, HOLD, and become state 3 automatically. Here Y arrived while state 4 was still active. This could mean Y was sent before completion, or X had not yet satisfied its endpoint conditions. The pictures cannot distinguish those possibilities. If the physical actuator had already stopped, inspect the endpoint variables in section 10 before changing the busy guard.

`SR_READY` means available for commands; it does not itself prove zero position or valid calibration. Use `calibration_valid` and the position/reference fields as well.

## 2. Reading the trace correctly

Example: `0 Q put 5` consists of timestamp, event label, and command enum value. It does **not** mean five queue entries. `Fnnn` identifies a function entry, not a return value, stack depth, or CPU backtrace.

| Prefix | Module |
|---|---|
| `TD` / `TM` | Debug task / TMC motor task |
| `SV` | Motor-control service |
| `SR` | Spring-return state machine |
| `P` | Actuator profile |
| `M` | TMC motor operations |
| `T` | TMC register layer |
| `S` | SPI transport driver |
| `U` | Debug UART driver |
| `Q` | Motor command queue |

The complete existing F-ID dictionary is in [TRACE_FUNCTION_MAP.md](TRACE_FUNCTION_MAP.md). Some intermediate transport/callback entries do not appear in the pictures; absence is not proof that those functions did not execute.

The recorder queues entries in RAM; the debug task prints them later. Ordinary messages such as `SR CAL COMPLETE` are printed through a different, direct call to `drv_debug_write_text`. Consequently a direct message can appear before older buffered trace entries. In picture 3, the busy text appears before `Y cmd rx`, although `TRACE_COMMAND(ch)` executes before the guard. This is expected from those two output paths.

Recurring function and synchronization IDs are independently limited to one record per five seconds. Therefore the displayed SPI take/give records are samples, not matched transaction pairs. The log has no transaction ID, function exit, argument dump, or complete return-value capture. The 128-record ring drops records instead of blocking; `T drop n` reports accumulated loss when its reporting interval permits. UART events occurring during trace transmission are suppressed to prevent logging the logger forever; that suppression can also hide concurrent UART RX events during that interval.

### Why every timestamp shown is zero

`CallTrace_Record()` gets time from `HAL_GetTick()`. All visible timestamps are zero, including events after calibration. If that HAL time base remains zero in the baseline, periodic IDs are recorded once and then suppressed forever because `now - last_ms[id]` never reaches 5000. Drop reporting uses the same clock. This is a strong reason to inspect the baseline HAL tick, not proof of its exact fault from photographs alone.

Motor scheduling and SR timeout handling use FreeRTOS `xTaskGetTickCount()`, a separate API. Successful motion does not establish that HAL time is advancing. Observe both time bases at two different running instants, or inspect the baseline implementation and interrupt/time-base integration. Do not interpret the printed zeros as instantaneous motor movement. This guide does not silently change the time base in the working firmware.

## 3. Execution owners and FreeRTOS objects

The debug task interprets characters. The motor task owns motor operations. Interrupt callbacks deliver completion/data events; they do not run the SR state machine.

| Object | Exact location/name | Producer or owner | Consumer / purpose |
|---|---|---|---|
| RX stream buffer | `s_uart.rx_stream`, 128 bytes, trigger 1 | USART2 RX callback writes one byte using `xStreamBufferSendFromISR` | Debug task calls `xStreamBufferReceive` through `DRV_DEBUG_ReadBuffered` |
| Command queue | `s_motor_command_queue`, 12 items | Debug task copies `motor_command_t {type, actuator_type}` using `xQueueSend` | Motor task receives one item per SR loop using `xQueueReceive` |
| SPI mutex | `s_spi[SPI_INTERFACE_PORT_TMC].mutex` | Calling task takes and releases | Exclusive ownership of one SPI DMA frame |
| SPI completion semaphore | `s_spi[...].done`, binary | SPI ISR gives using `xSemaphoreGiveFromISR` | Waiting transfer task takes; resumes after completion/error |
| UART TX mutex | `s_uart.mutex` | Task writing console takes/releases | Prevents interleaving of UART transmissions |
| UART TX completion semaphore | `s_uart.tx_done`, binary | UART TX/error ISR gives | Writer takes; its transmit buffer remains valid while waiting |
| Stop request | `s_sr.stop_requested`, volatile bool | Debug task sets via `ActuatorSR_RequestStop` | Motor poll clears it and performs HOLD |
| Diagnostic view | `g_status`, volatile structure | Primarily motor task | Debug guard and R read cached fields |
| Trace ring | `records`, `head`, `tail`, `dropped` | Instrumented tasks/ISRs | Only debug task drains; short PRIMASK-protected accesses |

A **mutex** represents resource ownership. The task that acquires it releases it; the ISR never releases either UART or SPI mutex. A **binary semaphore** represents completion: the ISR gives one token, the task takes it. A **queue** copies a structured command, whereas the RX **stream buffer** carries raw bytes. Blocking on a semaphore lets FreeRTOS run other ready tasks; it is not a busy-wait.

These objects are distinct even though mutexes and binary semaphores both use `SemaphoreHandle_t`. `ctx` in the SPI file and `ctx` in the UART file point to different structures. `hpw` is a callback-local `BaseType_t` initially `pdFALSE`; the ISR-safe API can set it, and `portYIELD_FROM_ISR(hpw)` requests a context switch if needed.

The standalone reference creates the RTOS objects dynamically during initialization. The trace ring is static and adds no RTOS queue or task. `volatile` on `g_status` does not make a multi-field snapshot atomic: R can show fields from adjacent motor updates. The motor task rechecks command eligibility when dequeuing instead of relying solely on the debug task's cached guard.

## 4. Startup before B

```text
generated peripheral initialization → app_main() [F124]
  xTaskCreate(task_tmc6460_run, priority 4)
  xTaskCreate(task_debug_run, priority 3)
  vTaskStartScheduler()

task_tmc6460_run() [F146]
  wait for task_debug_get_state() using vTaskDelay(1)

task_debug_run() [F131]
  command_task = xTaskGetCurrentTaskHandle()
  DRV_DEBUG_Init() → DRV_DEBUG_Register(mx_usart2_uart_gethandle())
    create UART mutex and tx_done semaphore
  DRV_DEBUG_StartBufferedRx()
    create RX stream → DRV_DEBUG_ArmBufferedRx() → HAL_UART_Receive_IT(..., 1)
  debug_state = 1
  wait for task_tmc6460_get_state()

motor task resumes
  spi_interface_init() [F006]
    get SPI2 handle, validate TX/RX DMA handles
    create SPI mutex and done semaphore; CS high; initialized = 1
  svc_motor_control_init() [F118]
    clear g_status; TMC6460_Init() [F148]
    TMC6460_MotorInitContext() [F162]
    create command queue
    ActuatorSR_Init(&s_motor) [F092] → state 0
  motor_state = 1; startup delay; enter recurring motor loop

debug task resumes → prints banner/help → receive loop
```

The higher-priority motor task yields while waiting for the lower-priority debug task; otherwise startup could starve the UART initialization it needs. `motor_state` in the task file is an initialization status, not `s_sr.state` and not `g_status.motor_state`.

SR motor loop: `svc_motor_control_sample()` → `ActuatorSR_Poll()`; then `svc_motor_control_process_next(0)`; then `vTaskDelayUntil(...)`. SR sampling is nominally 200 Hz (5 ms), with actual duration affected by SPI work, scheduling, and errors. Poll runs before one queued command, so an incoming command backlog cannot prevent endpoint/stop processing.

## 5. Every console character: interrupt → stream → queue

```text
USART2 receives one byte
  generated UART IRQ/HAL processing
  common HAL_UART_RxCpltCallback(handle, size, event)
  drv_debug_on_rx_complete() [F025]
    DRV_DEBUG_FindByHandle() checks it is USART2
    require TC event and size == 1
    xStreamBufferSendFromISR(s_uart.rx_stream, &buffered_rx_byte, 1, &hpw)
    re-arm HAL_UART_Receive_IT(..., 1)
    portYIELD_FROM_ISR(hpw)

task_debug_run()
  DRV_DEBUG_ReadBuffered(&ch, 1, TRACE_RX_WAIT_MS)
    xStreamBufferReceive(...) → ch
  convert lowercase to uppercase
  TRACE_COMMAND(ch)
  validate SR busy/initialized/calibrated conditions
  switch(ch)
    motion/profile command → svc_motor_control_queue_command(type, profile)
      xQueueSend(s_motor_command_queue, &command, 20 ms)
      Q put <type> or Q full <type>

task_tmc6460_run()
  svc_motor_control_process_next(0) [F121]
    xQueueReceive(..., &command, 0) → Q get <type>
    process_motor_command() [F116]
    ActuatorSR_Command(command.type, command.actuator_type) [F098]
```

The queued object is a copy; the debug task's local `command` can go out of scope safely. `Q put` means accepted into the queue, not successful motor movement. State can change between enqueue and dequeue. No UART RX mutex is taken for the single-writer/single-reader stream. The baseline common callback must retain USART1/USART3 dispatch and route only USART2 to this helper.

With trace enabled, an empty RX wait expires after 20 ms and calls `TRACE_PUMP()` once. Commands take priority over draining old trace records. A steady input stream can therefore delay trace output.

## 6. B: selecting the 20 kg profile

`B` → `ActuatorProfile_FromCommand` [F050] → queue command **0** → `ActuatorSR_Command` → `sr_select_profile` [F097].

`sr_select_profile` obtains the profile using `ActuatorProfile_Get` [F049], calls `sr_stop(SR_READY, SR_ERROR_NONE)` before replacing an initialized motor context, and calls `sr_clear_calibration` [F096]. On the first startup, stop skips hardware HOLD if the motor has not yet been initialized.

It copies `profile->motor` into `s_sr.profile`, raises that private profile's maximum velocity to at least `SR_RUN_VELOCITY_RAW`, and calls `TMC6460_MotorConfigure` [F163]. Configuration reads the chip ID, writes motor configuration registers, verifies the driver, and propagates failures. These register calls all follow the SPI path in section 9.

On success it sets `g_status.actuator_initialized`, selected type and TMC status, reads position through `sr_read_position` [F090] → `TMC6460_MotorReadPosition` [F168], and becomes state 3. The displayed position `-43691` is a raw motor coordinate, not negative output degrees. B alone does not establish output zero or enable X/Y.

## 7. G: zero → 90° → zero

`G` → queue command **5** → `ActuatorSR_Command`:

1. Read current motor position; a read failure faults/stops instead of calibrating.
2. Clear previous calibration validity and span.
3. Set `s_sr.zero = s_sr.position` and `g_status.reference_zero_raw`. Sending G confirms that the output is physically at zero; there is no potentiometer proving this.
4. Call `sr_begin(SR_CAL_CW)` [F091]. Select CW torque, reset contact counters, set `leg_start = previous = position`, and call `TMC6460_MotorSetVelocity(+3000000)` [F164]. On success capture the RTOS start tick and enter state **1**.
5. Each motor poll reads position, velocity and torque using `TMC6460_MotorReadMotionSample` [F169], updates cached data, checks direction/timeout, and evaluates contact.
6. During first calibration `stroke == 0`, so there is no calibrated 99.5% arm window. In soft mode this implementation uses the reference-search timing: 400 ms blanking, then 250 ms stable-and-slow confirmation or 600 ms position-only confirmation. At nominal 200 Hz these are 50 or 120 consecutive counted samples. Unstable/rejected votes reset counters through `ActuatorEnd_Count` [F177].
7. `actuator_90_end_reached` [F087] accepts contact. Compute `stroke = position - zero`, require at least 1000 counts and at most `INT32_MAX`, then HOLD through `sr_stop`. Transition to state **2**, store `stroke`, `ninety` and reference fields, and `sr_begin(SR_CAL_CCW)` commands **-3000000**.
8. CCW polling reads position first. `actuator_zero_end_reached` [F085] is `position <= zero`, evaluated in 64-bit arithmetic. It does not wait for a zero-end stall.
9. Successful HOLD at zero sets `calibration_valid`, `calibration_params_loaded`, and `position_reference_valid` to 1; state becomes **3**; print `SR CAL COMPLETE`.

The zero, ninety and span are stored in RAM for this session. This path does not write Flash/EEPROM. B or manual V/W invalidates calibration; a reset loses it. The motor coordinate is not reset to numeric zero: travel is a difference between raw coordinates.

HOLD occurs after detecting the threshold on a sample and completing SPI stop writes. It is not an exact physical-angle guarantee; polling latency, inertia and backlash remain relevant on hardware.

## 8. X, rejected Y, Z, accepted Y

### X: calibrated CW

Queue command **6** reaches `ActuatorSR_Command`. `ActuatorSR_IsReady` [F093] requires state 3, valid calibration and no pending stop. A fresh position is read, then `sr_begin(SR_RUN_CW)` commands positive velocity and enters state **4**.

CW does not simply stop at `position == ninety`. It uses existing contact detection. In soft mode, contact is armed when `(position - zero) * 10000 >= stroke * 9950`. With calibrated span, confirmation uses 180 ms stable-and-slow or 400 ms stable position: 36 or 80 consecutive samples at 200 Hz. The SR soft-mode blanking remains the reference-search 400 ms macro in this implementation. SR does not call the entire NSR `ActuatorSoftStall_Run` creep/boost state machine.

On confirmed contact, HOLD returns to ready, preserves the calibrated span, and re-anchors `ninety = current position`, `zero = ninety - stroke`. This corrects the raw coordinate mapping at the loaded CW endpoint. Timeout, direction, communication and overshoot checks can instead cause FAULT.

### Why the first Y was rejected

In `task_debug_run`, the guard permits motion/profile commands only when cached SR state is BOOT, READY or FAULT; other checks then reject invalid initialization/fault/calibration combinations. State **4** fails the first guard, so it prints busy and executes `continue`. No Y is queued and there is no automatic pending reversal.

This is why `Y cmd rx` can appear without `Q put 7`. Do not remove the guard merely to make Y accepted during X: that would change the working motion policy.

### Z: direct stop request

```text
debug task: Z
  svc_motor_control_request_abort() [F120]
    ActuatorSR_RequestStop() [F094] → stop_requested = true
    xQueueReset(s_motor_command_queue) → Q clr

motor task: next ActuatorSR_Poll()
  clear stop_requested
  sr_calibrating() [F081]
  if aborting calibration: clear validity
  sr_stop(READY, existing error) for normal initialized motion
    TMC6460_MotorStopHold() [F167]
    sr_set_state(READY) [F084]
```

Z bypasses the queue so it is not stuck behind ordinary commands. Queue reset discards pending commands; it cannot cancel an in-flight SPI transfer. The motor task performs the hardware stop on a subsequent poll. Z during normal X preserves valid calibration, which is why the following Y can run. Z during G invalidates calibration. Z in FAULT does not automatically clear the fault state.

### Second Y: calibrated CCW

Queue command **7** is now accepted. The motor owner reads fresh position. If already `position <= zero`, it holds immediately. Otherwise `sr_begin(SR_RUN_CCW)` enters state **5** with negative velocity. Polling checks position before optional extra motion data, stops when `position <= zero`, and returns to state **3**. No CCW stall is required. The photographed state 5 proves the CCW start path was entered; it does not reveal the exact number of counts traveled.

## 9. Every motor register access: SPI mutex and semaphore

```text
motor operation (configure / velocity / hold / read position)
  TMC6460_ReadRegister [F149] or TMC6460_WriteRegister [F150]
    TMC6460_TransportReadRegister [F175] / TransportWriteRegister [F176]
      TMC6460_SPI_BuildFrame [F172]
      spi_interface_transfer [F007] -- first 6-byte frame
      spi_interface_transfer [F007] -- second 6-byte frame
      TMC6460_SPI_ResponseAddress [F173] validates reply address
      TMC6460_SPI_ResponseData [F174] extracts read data
    TMC6460_ConvertTransportStatus [F147] maps status back to motor layer
```

TMC reads send the same request twice because the reply is pipelined. Writes send the write and a follow-up read. Each frame separately takes/releases the SPI mutex. The complete two-frame operation is safe under the current single motor-task owner; the per-frame mutex alone does not make two frames atomic against a future independent register-access task.

| Step | Executing context | Action and trace |
|---|---|---|
| 1 | Motor task | `spi_interface_get_context` validates port, initialized state and buffers. |
| 2 | Motor task | `S mx wait`; `xSemaphoreTake(ctx->mutex, ticks)`; success `S mx take`. Failure returns BUSY. |
| 3 | Motor task | Clear `ctx->error`; `spi_interface_clear_semaphore(ctx->done)` drains stale tokens with zero wait. |
| 4 | Motor task | `S dma start`; short critical section; `tmc6460_select()` makes PB12 low; `HAL_SPI_TransmitReceive_DMA`; exit critical section. |
| 5 | Motor task | `S sem wait`; `xSemaphoreTake(ctx->done, ticks)` blocks if no completion token yet. SPI mutex remains owned. |
| 6 | SPI interrupt path | HAL full-duplex completion → `HAL_SPI_TxRxCpltCallback` → `drv_tmc6460_on_complete` [F010] → find handle [F004]. |
| 7 | SPI callback | `S txrx done`; `tmc6460_deselect()` makes CS high; `xSemaphoreGiveFromISR(ctx->done, &hpw)` → `S sem give`; yield if required. |
| 8 | Motor task | Wait succeeds → `S sem take`; check error; update transfer stats. |
| 9 | Motor task | `xSemaphoreGive(ctx->mutex)` → `S mx rel`; return SPI status. |

Completion may occur before the task begins waiting; the binary semaphore retains its token, so that ordering is valid. A completion semaphore is not released again by the task: taking consumes it. A mutex release does not itself mean a successful transfer; consult the error path/status.

HAL start failure deselects and releases the mutex immediately. A wait timeout aborts SPI, deselects, records timeout and releases the mutex. Error callback sets `ctx->error`, deselects and gives the semaphore; the task wakes, aborts the engine before returning caller-owned buffers, then releases the mutex. Thus a semaphore give may mean completion **or an error wakeup**.

`TMC_COMM_TIMEOUT_MS` is 25 ms in this source. It is used for each wait, not an overall deadline for a whole multi-register operation. DMA/IRQ priorities must obey the baseline FreeRTOS ISR API rules; the guide does not infer those priorities from the photographs.

### Velocity and HOLD beneath the state machine

`TMC6460_MotorSetVelocity` validates the context, clamps to the selected maximum, clears the velocity-fail event, ensures driver enable, writes run torque/flux limits, RUN mode, and velocity target. It delays 3 ms and verifies run state before returning success. On verification failure it requests zero velocity and returns an error.

`TMC6460_MotorStopHold` ensures driver enable, writes the profile HOLD torque/flux limits, retains RUN mode, and writes velocity target zero. It deliberately does not disable the driver or zero the torque/flux target. `sr_stop` retries this operation up to four times, with 2 ms between failed attempts; a final failure becomes `SR_ERROR_STOP`/FAULT.

SR CW/CCW torque macros currently use zero as a **profile fallback selector**, not an instruction to apply zero torque. Positive overrides replace the run limit for that direction in `s_sr.profile`. Both directions use the same configured velocity magnitude. HOLD uses the profile's separate hold limit.

## 10. Variables to follow in IAR Watch

File-static symbols may need IAR's module-qualified selection. Observe values while running if supported; a breakpoint can change timing. Never infer one coherent snapshot from separately read volatile fields.

| Variable | Role and update |
|---|---|
| `s_sr.motor`, `s_sr.profile` | Sole motor context and private active profile; B configures them. |
| `s_sr.state` / `g_status.sr_state` | Authoritative state and published copy; updated by `sr_set_state`. |
| `s_sr.stop_requested` | Debug-to-motor stop flag; consumed before normal endpoint processing. |
| `s_sr.position` / `g_status.position_raw` | Latest successful position read. |
| `s_sr.zero` | Raw coordinate representing physical zero; G captures, X endpoint may re-anchor. |
| `s_sr.ninety` | Last accepted CW endpoint raw coordinate. |
| `s_sr.stroke` | Calibrated positive travel count; zero means no usable span. |
| `s_sr.previous` | Prior CW sample used for per-sample movement delta. |
| `s_sr.leg_start` | Position when this motion started; direction plausibility check. |
| `s_sr.contact_count` | Consecutive armed, stable-and-slow sample count. |
| `s_sr.position_count` | Consecutive armed stable-position sample count, independent fallback. |
| `s_sr.started` | RTOS tick at successful motion start; move timeout reference. |
| `sample.position_raw`, `.velocity_raw`, `.torque_raw` | CW poll's local motor sample; not a persistent object. |
| `g_status.reference_zero_raw`, `.reference_ninety_raw` | Published endpoint mapping. |
| `g_status.calibration_stroke_counts` | Published span; R displays this as `span`. |
| `g_status.calibration_valid` | Becomes 1 only after successful G return/HOLD; X/Y gate. |
| `g_status.angle_deg_x100` | `(position-zero)*9000/stroke`; inferred angle, not a measured potentiometer angle. |
| `g_status.commanded_velocity_raw` | Requested signed velocity; zero after successful HOLD. |
| `g_status.velocity_actual_raw`, `.torque_actual_raw` | Refreshed on CW motion sampling; not necessarily fresh while idle or CCW. |
| `g_status.tmc_status`, `.sr_error` | Transport/motor result and SR reason; different enum spaces. |
| `g_status.position_sample_count`, `.position_error_count` | Position-read progress and errors. |
| `g_status.motor_stack_min_free_words` | Task stack high-water information, not bytes. |
| `ctx->mutex`, `ctx->done`, `ctx->error`, `ctx->stats` in SPI | Ownership handle, completion handle, ISR error flag and transfer counters. |
| `s_uart.rx_stream`, `.rx_restart`, `.stats` | Pending input, RX recovery flag, UART/drop counters. |
| `head`, `tail`, `dropped`, `last_ms` in trace module | Trace backlog, loss and per-event suppression timestamps. |

If X physically stops but stays state 4, inspect: position relative to `zero + stroke*0.995`; `previous` and delta stability; velocity threshold; both confirmation counters; RTOS elapsed move time; TMC status. Contact below the soft arm window is deliberately not accepted as the calibrated 90° endpoint. With the current soft-mode configuration, the move timeout is 30000 ms; hard mode uses 150000 ms. Do not change these conditions based only on missing rate-limited console lines.

## 11. State and command dictionaries

| SR number | State | Meaning |
|---|---|---|
| 0 | SR_BOOT | Waiting for initialization/profile selection |
| 1 | SR_CAL_CW | G outbound calibration |
| 2 | SR_CAL_CCW | G return using measured span |
| 3 | SR_READY | Idle/HOLD, eligibility also depends on validity/error flags |
| 4 | SR_RUN_CW | X active |
| 5 | SR_RUN_CCW | Y active |
| 6 | SR_FAULT | Fault; inspect R and correct cause |
| 7 | SR_MANUAL_CW | V active, calibration invalidated |
| 8 | SR_MANUAL_CCW | W active, requires operator stop at zero |

Queue command values: 0=select profile; 1=manual CW; 2=manual CCW; 3=stop/HOLD enum (SR console Z uses the direct path); 4=refresh diagnostics enum (SR R reads cached status directly); 5=G calibration; 6=X; 7=Y. These values are not SR states or queue occupancy.

SR error values: 0=none, 1=config, 2=TMC, 3=stop failure, 4=aborted calibration, 5=no travel, 6=direction, 7=timeout, 8=overshoot. Use the TMC enum definition separately when interpreting `tmc`.

## 12. UART output and trace safety

```text
drv_debug_write_text(text)
  DRV_DEBUG_Write(data, length, timeout)
    DRV_DEBUG_Lock → take s_uart.mutex
    clear error and stale tx_done tokens
    HAL_UART_Transmit_IT
    DRV_DEBUG_WaitCompletion → take s_uart.tx_done (blocks)

UART interrupt → HAL_UART_TxCpltCallback
  drv_debug_on_tx_complete → give tx_done FromISR → yield

writer resumes → check errors/stats → DRV_DEBUG_Unlock → give mutex
```

RX runs independently of that TX mutex. UART TX timeout aborts transmit without intentionally canceling RX. RX errors request receive recovery from the debug task; an error can also wake a waiting writer, which aborts TX before releasing its buffer.

`TRACE_ENTER`/`TRACE_EVENT`/`TRACE_VALUE` call `CallTrace_Record`. It validates the ID/context, saves PRIMASK, briefly disables interrupts, stores a bounded record and restores the previous PRIMASK. It performs no UART send, allocation or RTOS wait. `CallTrace_Drain`, in the debug task only, formats one short line and uses the ordinary UART TX path with a 10 ms timeout per relevant wait. This reduces interference but is not a guarantee of zero timing impact or absence of HardFault on all baseline integrations.

## 13. What is still needed for a truly complete measured trace

This document supplies the complete source-level path for the test, including branches omitted from the sampled log. The existing logger cannot retrospectively recover suppressed entries, missing variable values or transaction timings. Printing every SPI sample and register access over UART would substantially change execution timing.

For the next hardware capture, first verify the advancing HAL and RTOS clocks, enable terminal session logging, and record R after completed G, after X reports READY, and after Y completes. If X remains busy at the mechanical end, record the section 10 endpoint variables and the elapsed RTOS time before Z. A future bounded event snapshot can record the state, position, zero/span and counters on transitions without printing every polling call; that would be a separate firmware change. No untested logging additions or motion-policy changes are included here.

Related references: [all function IDs](TRACE_FUNCTION_MAP.md), [trace implementation guide](TRACE_GUIDE.md), [SR behavior](SR_SUPPORT.md). Source roots: `Portable/user_modifiable/Application/STM32C562RET6` and `Portable/user_modifiable/Device/TMC6460`, mirrored under `IAR_Reference/project/user_modifiable`.

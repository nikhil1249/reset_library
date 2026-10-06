# Optional call-flow tracing

The working, user-confirmed firmware is preserved unchanged in `../Benchmark/Actuator_Working_Benchmark.zip`. This active project adds optional tracing and uses `user_modifiable/Application/STM32C562RET6`. It has **not been compiled or hardware-tested**, at the user's request. Historical compiler/test results describe the benchmark only.

## Enable or remove tracing

In `Application/STM32C562RET6/Inc/app_config.h`:

```c
#ifndef TRACE_CALL_STACK
#define TRACE_CALL_STACK 1
#endif
```

Set the value to `0`, override it with `TRACE_CALL_STACK=0` in the project, or remove the definition. The trace header defaults an undefined switch to zero. With zero, all trace macros become no-ops, the logger has no code/data, and DebugCommand's original 1000 ms receive wait is restored. Existing operational messages remain. You do not need to remove trace statements individually. If physically removing trace support files, also remove their includes/call sites/project entries; the compile-time switch is the simpler option.

Add these three files when merging into a baseline: `Inc/call_trace.h`, `Inc/call_trace_ids.h`, and `Src/call_trace.c`. The IAR reference already includes them. Keep the baseline callback owners: `UserCode/uart_callbacks.c` and `drv_spi_iface.c` are reference-only examples, not extra callback owners to copy into an existing baseline.

## Reading the output

Example labels below illustrate the flow; they are not captured hardware output:

```text
1200 B cmd rx
1200 Q put 0
1205 Q get 0
... function-entry labels for profile configuration ...
2300 G cmd rx
2300 Q put 5
2305 Q get 5
2305 SR state 1
2305 S mx wait
2305 S mx take
2305 S dma start
2305 S sem wait
2306 S txrx done
2306 S sem give
2306 S sem take
2306 S mx rel
2306 pos 12345
```

The leading number is the **capture time in milliseconds**, not the later UART print time. A higher-priority task may run before a queue-send or mutex-release call returns: Q get can therefore precede the sender's Q put marker. Interpret timestamps and function labels as concurrent events, not strict nesting. `Fnnn` identifies a function entry; [TRACE_FUNCTION_MAP.md](TRACE_FUNCTION_MAP.md) maps each short label to its full function and file. This is an event log of function entry and synchronization, not a CPU backtrace or a function-return profiler.

| Label | Meaning |
|---|---|
| `G cmd rx` | DebugCommand has decoded G; it has not yet executed calibration |
| `Q put n` / `Q full n` | Enqueue succeeded/failed for command n |
| `Q get n` | MotorService dequeued command n |
| `S mx wait/take/busy/rel` | SPI mutex wait, acquired, acquisition failed, released |
| `S sem wait/give/take/to` | SPI completion wait, ISR give, task consumption, timeout |
| `S txrx done` | Full-duplex SPI DMA completion callback |
| `U mx ...`, `U sem ...` | UART TX mutex/completion synchronization |
| `U str put/get/full` | UART RX byte entered/left the stream buffer, or was dropped |
| `pos n` | Last successfully sampled raw motor position |
| `SR state n` | SR state transition, using existing state numbers |
| `T drop n` | Trace events were dropped because the trace ring or trace UART output could not keep up |

Command numbers: 0 profile selection, 1 manual CW, 2 manual CCW, 3 stop/hold, 4 diagnostics, 5 G calibration, 6 calibrated X, 7 calibrated Y. In SR, Z requests stop directly and clears the queue rather than enqueueing command 3.

## From a console character to motor motion

### 1. UART interrupt to DebugCommand

1. USART2 receives one byte. The existing HAL UART callback forwards to `drv_debug_on_rx_complete` in `Drivers/Src/drv_uart.c`.
2. `DRV_DEBUG_FindByHandle` selects the USART2 context. `xStreamBufferSendFromISR` copies the byte into the RX stream buffer. The callback rearms one-byte reception and yields if needed.
3. `task_debug_run` calls `DRV_DEBUG_ReadBuffered`, which calls `xStreamBufferReceive`. A stream buffer carries bytes; it does not own or operate the motor.
4. The task converts lowercase to uppercase and records the command marker, such as `G cmd rx`.
5. The existing parser checks profile initialization, busy/fault state and calibration eligibility. Rejected commands do not reach the motor queue.

The callback never prints a trace message. It only records a bounded event. UART's common USART1/USART3 functionality is not replaced by the USART2 trace path.

### 2. DebugCommand to MotorService

1. `svc_motor_control_queue_command` creates a command structure and calls `xQueueSend`. The queue copies that structure by value; a local stack pointer is not retained.
2. `task_tmc6460_run` first calls `svc_motor_control_sample`, then processes at most one queued command with `svc_motor_control_process_next`.
3. `xQueueReceive` copies the command into MotorService's local structure. The private `process_motor_command` dispatches it to `ActuatorSR_Command` in SR builds.
4. MotorService owns the motor context. DebugCommand does not run SPI motor operations directly.

`Q put` records admission to the queue, not successful motor movement. `Q get` shows when MotorService begins processing it.

### 3. B initializes the profile

`ActuatorSR_Command` -> `sr_select_profile` -> existing safe StopHold if initialized -> clear calibration -> copy profile -> `TMC6460_MotorConfigure`.

Configuration performs the existing register writes through `TMC6460_MotorWriteConfig` and `TMC6460_WriteRegister`. The SR-private velocity limit remains at least 3000000. NSR profile constants are unchanged. A position read follows successful configuration, and the existing profile-ready message is printed. B does not start calibration or movement.

### 4. G calibrates from physical zero

`ActuatorSR_Command` -> `sr_read_position` -> `sr_clear_calibration` -> capture zero -> `sr_begin(SR_CAL_CW)` -> `TMC6460_MotorSetVelocity`.

Each MotorService poll calls `ActuatorSR_Poll`. Its position/sample reads reach `TMC6460_MotorReadPosition` or `TMC6460_MotorReadMotionSample`. CW contact is decided by `actuator_90_end_reached`. The existing safe stop runs, span is recorded, and `sr_begin(SR_CAL_CCW)` starts the return. `actuator_zero_end_reached` compares the sampled position against captured zero. Successful StopHold at zero marks calibration valid and enables X/Y.

G still requires operator-confirmed physical zero. Tracing does not change calibration, torque, velocity or endpoint policy. The previously discussed count-only CW endpoint change has not been implemented: normal X still uses CW contact detection; Y uses calibrated zero counts.

### 5. Register call to SPI DMA completion

`TMC6460_ReadRegister/WriteRegister` -> `TMC6460_TransportReadRegister/WriteRegister` -> `spi_interface_transfer`.

1. The SPI driver validates the context and buffers.
2. `xSemaphoreTake(ctx->mutex, ticks)` obtains exclusive access to the transfer. A mutex serializes users and has task ownership/priority inheritance.
3. The driver clears any stale binary completion token, asserts CS, and starts `HAL_SPI_TransmitReceive_DMA` using the existing sequence.
4. The calling motor task waits on `ctx->done` with `xSemaphoreTake`. This is a binary completion semaphore, not the SPI ownership mutex.
5. The DMA/IRQ path invokes the existing shared `HAL_SPI_TxRxCpltCallback`, which forwards to `drv_tmc6460_on_complete`. It deasserts CS, gives the completion semaphore using `xSemaphoreGiveFromISR`, and may wake MotorService.
6. MotorService consumes the completion token, checks the existing transfer/error state, and releases the SPI mutex.
7. Transport code parses the response. The existing TMC read pipeline may require multiple datagrams; repeated low-level events are rate-limited.

Timeout and error paths retain their abort, CS release and mutex-release behavior. A mutex is released by task code; the ISR gives the binary completion semaphore, never the mutex.

### 6. Z remains a direct stop request

DebugCommand -> `svc_motor_control_request_abort` -> `ActuatorSR_RequestStop`, followed by `xQueueReset` in SR. The next motor poll processes the flag and calls the existing StopHold path. It does not wait for a queued Z behind other commands. `Q clr` indicates stale motor requests were purged.

## Five-second rate limit and transport isolation

Recurring function entries, position samples, register transfers, UART/SPI callbacks and synchronization events log at most once **per event ID per 5000 ms**, beginning with the first occurrence. Sparse command/state operations retain each call. Suppression means the output is not a lossless instruction-by-instruction history, and take/release samples need not represent every transfer.

Producers only store a timestamp, numeric ID and optional integer in a 128-entry static ring. They never allocate memory, print, block, take a FreeRTOS lock or dereference caller-owned string buffers. A short PRIMASK-protected section preserves the prior interrupt-mask state and works when called from an existing critical section. NMI/fault/system exceptions are excluded.

Only DebugCommand drains the ring, one line after an idle 20 ms UART receive wait. Pending command bytes are handled first. Trace output uses the existing UART writer with a 10 ms timeout for each of its mutex/completion waits. It therefore adds bounded serial-task latency, while MotorService retains higher priority. Ordinary operational messages retain their existing behavior.

UART trace events generated by the trace writer itself are suppressed, including its TX completion callback, to avoid an endless stream of self-generated traces. Concurrent SPI/motor events remain recordable. RX command markers are emitted when the debug task decodes the byte even if a UART RX event was suppressed during trace output.

There is no extra task, queue, semaphore, mutex or heap allocation for tracing. The ring, per-ID throttles and formatting buffer use approximately **2.5 KiB of additional static RAM on this 32-bit target**, plus constant labels/code in Flash. The formatting buffer is static and fixed at 64 bytes; integer formatting uses a 10-byte local scratch array and no printf or floating point. Exact linked memory usage has not been measured because compilation was explicitly skipped.

## Limits and verification

162 project-owned function definitions are instrumented, including the shared inline endpoint helper and reference UART/SPI callbacks. Stack-overflow hooks are deliberately excluded because invoking extra functions on an exhausted stack is unsafe. Trace internals, vendor HAL/FreeRTOS and generated startup/peripheral code are not recursively instrumented. Their relevant synchronization/transport boundaries are visible in the owned modules.

The benchmark preserves the last working version. This revision received source/path/coverage and bounded-buffer review only—no C compilation, executable tests, flashing or hardware test. These measures avoid known ISR-printing, recursive logging, unbounded-buffer and extra-heap hazards, but cannot guarantee absence of a HardFault or prove timing/stack margin on hardware. Confirm memory/stack margins with the existing M command and measure stop latency when you next build/test it. Disable TRACE_CALL_STACK to restore the original trace-free execution paths and receive wait.

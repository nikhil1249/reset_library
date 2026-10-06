# Trace function map

F labels identify function entry, not a captured CPU backtrace. Recurring entries appear at most once per five seconds.

| Label | Function | Source | Rate |
|---|---|---|---|
| `F001 tmc6460_select` | `tmc6460_select` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F002 tmc6460_deselect` | `tmc6460_deselect` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F003 S.get_context` | `spi_interface_get_context` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F004 S.find_by_handle` | `spi_interface_find_by_handle` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F005 S.clear_semaphore` | `spi_interface_clear_semaphore` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F006 S.init` | `spi_interface_init` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | each call |
| `F007 S.transfer` | `spi_interface_transfer` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F009 S.get_stats` | `spi_interface_get_stats` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F010 S.on_complete` | `drv_tmc6460_on_complete` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F011 S.on_error` | `drv_tmc6460_on_error` | `Application/STM32C562RET6/Drivers/Src/drv_tmc6460.c` | 5 s |
| `F012 U.FindByHandle` | `DRV_DEBUG_FindByHandle` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F013 U.ClearSemaphore` | `DRV_DEBUG_ClearSemaphore` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F014 U.Register` | `DRV_DEBUG_Register` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | each call |
| `F015 U.Init` | `DRV_DEBUG_Init` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | each call |
| `F016 U.Lock` | `DRV_DEBUG_Lock` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F017 U.Unlock` | `DRV_DEBUG_Unlock` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F018 U.WaitCompletion` | `DRV_DEBUG_WaitCompletion` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F019 U.Write` | `DRV_DEBUG_Write` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F020 U.ArmBufferedRx` | `DRV_DEBUG_ArmBufferedRx` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F021 U.StartBufferedRx` | `DRV_DEBUG_StartBufferedRx` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | each call |
| `F022 U.ReadBuffered` | `DRV_DEBUG_ReadBuffered` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F023 U.GetStats` | `DRV_DEBUG_GetStats` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F024 U.on_tx_complete` | `drv_debug_on_tx_complete` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F025 U.on_rx_complete` | `drv_debug_on_rx_complete` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F026 U.on_error` | `drv_debug_on_error` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F027 U.write_text` | `drv_debug_write_text` | `Application/STM32C562RET6/Drivers/Src/drv_uart.c` | 5 s |
| `F028 C.GetLastLegDiagnostic` | `ActuatorCalibration_GetLastLegDiagnostic` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F029 C.RecordTmcFailure` | `ActuatorCalibration_RecordTmcFailure` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F030 C.Abs32` | `ActuatorCalibration_Abs32` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F031 C.AbsDiff32` | `ActuatorCalibration_AbsDiff32` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F032 C.GetVelocity` | `ActuatorCalibration_GetVelocity` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F033 C.GetTorqueThreshold` | `ActuatorCalibration_GetTorqueThreshold` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F034 C.ReadMotionSampleRetry` | `ActuatorCalibration_ReadMotionSampleRetry` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F035 C.ReadPositionRetry` | `ActuatorCalibration_ReadPositionRetry` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F036 C.SetVelocityRetry` | `ActuatorCalibration_SetVelocityRetry` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F037 C.StopHoldRetry` | `ActuatorCalibration_StopHoldRetry` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F038 C.Notify` | `ActuatorCalibration_Notify` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F039 C.Median` | `ActuatorCalibration_Median` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F040 C.Average` | `ActuatorCalibration_Average` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F041 C.Spread` | `ActuatorCalibration_Spread` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F042 C.CaptureEndpoint` | `ActuatorCalibration_CaptureEndpoint` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F043 C.MoveToEnd` | `ActuatorCalibration_MoveToEnd` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F044 C.Analyze` | `ActuatorCalibration_Analyze` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F046 C.Run` | `ActuatorCalibration_Run` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | each call |
| `F047 C.GetLastTmcError` | `ActuatorCalibration_GetLastTmcError` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F048 C.GetTmcRetryCount` | `ActuatorCalibration_GetTmcRetryCount` | `Application/STM32C562RET6/Services/Src/actuator_calibration.c` | 5 s |
| `F049 P.Get` | `ActuatorProfile_Get` | `Application/STM32C562RET6/Services/Src/actuator_profile.c` | 5 s |
| `F050 P.FromCommand` | `ActuatorProfile_FromCommand` | `Application/STM32C562RET6/Services/Src/actuator_profile.c` | 5 s |
| `F051 R.SaturateInt64` | `ActuatorReference_SaturateInt64` | `Application/STM32C562RET6/Services/Src/actuator_reference.c` | 5 s |
| `F052 R.Clear` | `ActuatorReference_Clear` | `Application/STM32C562RET6/Services/Src/actuator_reference.c` | 5 s |
| `F053 R.LoadCompileTime` | `ActuatorReference_LoadCompileTime` | `Application/STM32C562RET6/Services/Src/actuator_reference.c` | 5 s |
| `F054 R.UpdateFromCalibration` | `ActuatorReference_UpdateFromCalibration` | `Application/STM32C562RET6/Services/Src/actuator_reference.c` | 5 s |
| `F055 R.AnchorAtEnd` | `ActuatorReference_AnchorAtEnd` | `Application/STM32C562RET6/Services/Src/actuator_reference.c` | 5 s |
| `F056 R.PositionToDegX100` | `ActuatorReference_PositionToDegX100` | `Application/STM32C562RET6/Services/Src/actuator_reference.c` | 5 s |
| `F057 SS.Abs32` | `ActuatorSoftStall_Abs32` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F058 SS.AbsDiff32` | `ActuatorSoftStall_AbsDiff32` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F059 SS.Clamp32` | `ActuatorSoftStall_Clamp32` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F060 SS.SaturateInt64` | `ActuatorSoftStall_SaturateInt64` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F061 SS.Reached` | `ActuatorSoftStall_Reached` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F062 SS.DistanceToEnd` | `ActuatorSoftStall_DistanceToEnd` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F063 SS.MsToSamples` | `ActuatorSoftStall_MsToSamples` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F064 SS.GetFixedCreepVelocity` | `ActuatorSoftStall_GetFixedCreepVelocity` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F065 SS.GetReferenceSearchVel` | `ActuatorSoftStall_GetReferenceSearchVelocity` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F066 SS.GetTorqueThreshold` | `ActuatorSoftStall_GetTorqueThreshold` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F067 SS.RecordTmcFailure` | `ActuatorSoftStall_RecordTmcFailure` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F068 SS.ReadMotionRetry` | `ActuatorSoftStall_ReadMotionRetry` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F069 SS.ReadPositionRetry` | `ActuatorSoftStall_ReadPositionRetry` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F070 SS.SetVelocityRetry` | `ActuatorSoftStall_SetVelocityRetry` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F071 SS.StopHoldRetry` | `ActuatorSoftStall_StopHoldRetry` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F072 SS.Notify` | `ActuatorSoftStall_Notify` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F073 SS.ComputeBoundaryX100` | `ActuatorSoftStall_ComputeBoundaryX100` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F074 SS.ComputeCreepVelocity` | `ActuatorSoftStall_ComputeCreepVelocity` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F075 SS.RunReferenceSearch` | `ActuatorSoftStall_RunReferenceSearch` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F076 SS.RunAnchored` | `ActuatorSoftStall_RunAnchored` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F077 SS.Run` | `ActuatorSoftStall_Run` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | each call |
| `F079 SS.GetTmcRetryCount` | `ActuatorSoftStall_GetTmcRetryCount` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F080 SS.GetLastTmcError` | `ActuatorSoftStall_GetLastTmcError` | `Application/STM32C562RET6/Services/Src/actuator_soft_stall.c` | 5 s |
| `F081 SR.calibrating` | `sr_calibrating` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F082 SR.moving_cw` | `sr_moving_cw` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F083 SR.moving` | `sr_moving` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F084 SR.set_state` | `sr_set_state` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F085 actuator_zero_end_reache` | `actuator_zero_end_reached` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F086 actuator_90_contact_arme` | `actuator_90_contact_armed` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F087 actuator_90_end_reached` | `actuator_90_end_reached` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F088 SR.update_position` | `sr_update_position` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F089 SR.stop` | `sr_stop` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F090 SR.read_position` | `sr_read_position` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F091 SR.begin` | `sr_begin` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F092 SR.Init` | `ActuatorSR_Init` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F093 SR.IsReady` | `ActuatorSR_IsReady` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F094 SR.RequestStop` | `ActuatorSR_RequestStop` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F095 SR.PrintStatus` | `ActuatorSR_PrintStatus` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F096 SR.clear_calibration` | `sr_clear_calibration` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F097 SR.select_profile` | `sr_select_profile` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F098 SR.Command` | `ActuatorSR_Command` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | each call |
| `F099 SR.Poll` | `ActuatorSR_Poll` | `Application/STM32C562RET6/Services/Src/actuator_sr.c` | 5 s |
| `F100 SV.sample_period` | `svc_motor_control_sample_period` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F101 update_motor_stack_water` | `update_motor_stack_watermark` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F102 update_angle_from_positi` | `update_angle_from_position` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F103 update_reference_status` | `update_reference_status` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F104 set_command_result` | `set_command_result` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F105 refresh_tmc_feedback` | `refresh_tmc_feedback` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F106 print_calibration_define` | `print_calibration_defines` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F107 calibration_progress` | `calibration_progress` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F110 get_soft_creep_velocity_` | `get_soft_creep_velocity_for_profile` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F111 SV.print_soft_stall_conf` | `svc_motor_control_print_soft_stall_config` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F112 soft_stall_progress` | `soft_stall_progress` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F113 process_motor_command` | `process_motor_command` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | each call |
| `F116 process_motor_command` | `process_motor_command` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | each call |
| `F117 SV.queue_command` | `svc_motor_control_queue_command` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | each call |
| `F118 SV.init` | `svc_motor_control_init` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | each call |
| `F119 motor_command_queue_spac` | `motor_command_queue_spaces` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F120 SV.request_abort` | `svc_motor_control_request_abort` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | each call |
| `F121 SV.process_next` | `svc_motor_control_process_next` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F122 SV.sample` | `svc_motor_control_sample` | `Application/STM32C562RET6/Services/Src/svc_motor_control.c` | 5 s |
| `F123 app_main_error_trap` | `app_main_error_trap` | `Application/STM32C562RET6/Src/app_main.c` | 5 s |
| `F124 app_main` | `app_main` | `Application/STM32C562RET6/Src/app_main.c` | each call |
| `F125 RAM.StackFree` | `RuntimeMemory_StackFree` | `Application/STM32C562RET6/Src/runtime_memory.c` | 5 s |
| `F126 RAM.Get` | `RuntimeMemory_Get` | `Application/STM32C562RET6/Src/runtime_memory.c` | 5 s |
| `F127 RAM.FormatDetailed` | `RuntimeMemory_FormatDetailed` | `Application/STM32C562RET6/Src/runtime_memory.c` | 5 s |
| `F128 TD.get_handle` | `task_debug_get_handle` | `Application/STM32C562RET6/Tasks/Src/task_debug.c` | 5 s |
| `F129 TD.get_state` | `task_debug_get_state` | `Application/STM32C562RET6/Tasks/Src/task_debug.c` | 5 s |
| `F130 debug_print_help` | `debug_print_help` | `Application/STM32C562RET6/Tasks/Src/task_debug.c` | 5 s |
| `F131 TD.run` | `task_debug_run` | `Application/STM32C562RET6/Tasks/Src/task_debug.c` | each call |
| `F144 TM.get_handle` | `task_tmc6460_get_handle` | `Application/STM32C562RET6/Tasks/Src/task_tmc6460.c` | 5 s |
| `F145 TM.get_state` | `task_tmc6460_get_state` | `Application/STM32C562RET6/Tasks/Src/task_tmc6460.c` | 5 s |
| `F146 TM.run` | `task_tmc6460_run` | `Application/STM32C562RET6/Tasks/Src/task_tmc6460.c` | each call |
| `F147 T.ConvertTransportStatus` | `TMC6460_ConvertTransportStatus` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F148 T.Init` | `TMC6460_Init` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F149 T.ReadRegister` | `TMC6460_ReadRegister` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F150 T.WriteRegister` | `TMC6460_WriteRegister` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F151 T.ReadField` | `TMC6460_ReadField` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F152 T.WriteField` | `TMC6460_WriteField` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F153 T.ReadChipID` | `TMC6460_ReadChipID` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F154 T.CheckCommunication` | `TMC6460_CheckCommunication` | `Device/TMC6460/Src/tmc6460.c` | 5 s |
| `F155 M.PackLimit` | `TMC6460_MotorPackLimit` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F157 M.Abs32` | `TMC6460_MotorAbs32` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F158 M.WriteConfig` | `TMC6460_MotorWriteConfig` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F159 M.VerifyDriverOn` | `TMC6460_MotorVerifyDriverOn` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F160 M.EnsureDriverEnabled` | `TMC6460_MotorEnsureDriverEnabled` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F161 M.VerifyRunState` | `TMC6460_MotorVerifyRunState` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F162 M.InitContext` | `TMC6460_MotorInitContext` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F163 M.Configure` | `TMC6460_MotorConfigure` | `Device/TMC6460/Src/tmc6460_motor.c` | each call |
| `F164 M.SetVelocity` | `TMC6460_MotorSetVelocity` | `Device/TMC6460/Src/tmc6460_motor.c` | each call |
| `F165 M.RunCW` | `TMC6460_MotorRunCW` | `Device/TMC6460/Src/tmc6460_motor.c` | each call |
| `F166 M.RunCCW` | `TMC6460_MotorRunCCW` | `Device/TMC6460/Src/tmc6460_motor.c` | each call |
| `F167 M.StopHold` | `TMC6460_MotorStopHold` | `Device/TMC6460/Src/tmc6460_motor.c` | each call |
| `F168 M.ReadPosition` | `TMC6460_MotorReadPosition` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F169 M.ReadMotionSample` | `TMC6460_MotorReadMotionSample` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F170 M.ReadFeedback` | `TMC6460_MotorReadFeedback` | `Device/TMC6460/Src/tmc6460_motor.c` | 5 s |
| `F171 T.FromSpiStatus` | `TMC6460_FromSpiStatus` | `Device/TMC6460/Src/tmc6460_transport.c` | 5 s |
| `F172 T.SPI_BuildFrame` | `TMC6460_SPI_BuildFrame` | `Device/TMC6460/Src/tmc6460_transport.c` | 5 s |
| `F173 T.SPI_ResponseAddress` | `TMC6460_SPI_ResponseAddress` | `Device/TMC6460/Src/tmc6460_transport.c` | 5 s |
| `F174 T.SPI_ResponseData` | `TMC6460_SPI_ResponseData` | `Device/TMC6460/Src/tmc6460_transport.c` | 5 s |
| `F175 T.ReadRegister` | `TMC6460_TransportReadRegister` | `Device/TMC6460/Src/tmc6460_transport.c` | 5 s |
| `F176 T.WriteRegister` | `TMC6460_TransportWriteRegister` | `Device/TMC6460/Src/tmc6460_transport.c` | 5 s |
| `F177 ActuatorEnd_Count` | `ActuatorEnd_Count` | `Application/STM32C562RET6/Services/Inc/actuator_end_detection.h` | 5 s |
| `F178 HAL_UART_TxCpltCallback` | `HAL_UART_TxCpltCallback` | `IAR_Reference/project/UserCode/uart_callbacks.c` | 5 s |
| `F179 HAL_UART_RxCpltCallback` | `HAL_UART_RxCpltCallback` | `IAR_Reference/project/UserCode/uart_callbacks.c` | 5 s |
| `F180 HAL_UART_ErrorCallback` | `HAL_UART_ErrorCallback` | `IAR_Reference/project/UserCode/uart_callbacks.c` | 5 s |
| `F181 HAL_SPI_TxRxCpltCallback` | `HAL_SPI_TxRxCpltCallback` | `IAR_Reference/project/UserCode/drv_spi_iface.c` | 5 s |
| `F182 HAL_SPI_ErrorCallback` | `HAL_SPI_ErrorCallback` | `IAR_Reference/project/UserCode/drv_spi_iface.c` | 5 s |

## Synchronization and command events

- `S mx wait`
- `S mx take`
- `S mx busy`
- `S mx rel`
- `S mx rel err`
- `S sem wait`
- `S sem take`
- `S sem to`
- `S sem give`
- `S sem full`
- `S sem clr`
- `S dma start`
- `S txrx done`
- `S err`
- `U mx wait`
- `U mx take`
- `U mx busy`
- `U mx rel`
- `U mx rel err`
- `U sem wait`
- `U sem take`
- `U sem to`
- `U sem give`
- `U sem full`
- `U sem clr`
- `U tx start`
- `U tx done`
- `U rx`
- `U str put`
- `U str full`
- `U str get`
- `Q put`
- `Q full`
- `Q get`
- `Q clr`
- `pos`
- `SR state`
- `A cmd rx`
- `B cmd rx`
- `C cmd rx`
- `D cmd rx`
- `E cmd rx`
- `F cmd rx`
- `G cmd rx`
- `H cmd rx`
- `I cmd rx`
- `J cmd rx`
- `K cmd rx`
- `L cmd rx`
- `M cmd rx`
- `N cmd rx`
- `O cmd rx`
- `P cmd rx`
- `Q cmd rx`
- `R cmd rx`
- `S cmd rx`
- `T cmd rx`
- `U cmd rx`
- `V cmd rx`
- `W cmd rx`
- `X cmd rx`
- `Y cmd rx`
- `Z cmd rx`

## Intentional exclusions

- `motor_stack_overflow_hook`: Fault hook: do not consume additional stack or call trace code.
- `vApplicationStackOverflowHook`: Fault hook forwards directly without tracing.

Vendor/generated functions and trace internals are excluded. Their transport and RTOS synchronization boundaries are traced in the owned drivers/services.

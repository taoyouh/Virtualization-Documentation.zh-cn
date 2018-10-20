# [概述](./index.md)
# [Windows 虚拟机监控程序平台 API](./hypervisor-platform/hypervisor-platform.md)
## [WHvCancelRunVirtualProcessor](./hypervisor-platform/funcs/WHvCancelRunVirtualProcessor.md)
## [WHvCreatePartition](./hypervisor-platform/funcs/WHvCreatePartition.md)
## [WHvCreateVirtualProcessor](./hypervisor-platform/funcs/WHvCreateVirtualProcessor.md)
## [WHvDeletePartition](./hypervisor-platform/funcs/WHvDeletePartition.md)
## [WHvDeleteVirtualProcessor](./hypervisor-platform/funcs/WHvDeleteVirtualProcessor.md)
## [WHvGetCapability](./hypervisor-platform/funcs/WHvGetCapability.md)
## [WHvGetPartitionProperty](./hypervisor-platform/funcs/WHvGetPartitionProperty.md)
### [数据类型](./hypervisor-platform/funcs/WHvPartitionPropertyDataTypes.md)
## [WHvGetVirtualProcessorRegisters](./hypervisor-platform/funcs/WHvGetVirtualProcessorRegisters.md)
### [数据类型](./hypervisor-platform/funcs/WHvVirtualProcessorDataTypes.md)
## [WHvMapGpaRange](./hypervisor-platform/funcs/WHvMapGpaRange.md)
## [WHvRunVirtualProcessor](./hypervisor-platform/funcs/WHvRunVirtualProcessor.md)
### [退出上下文](./hypervisor-platform/funcs/WHvExitContextDataTypes.md)
### [内存访问](./hypervisor-platform/funcs/MemoryAccess.md)
### [I/O 端口访问](./hypervisor-platform/funcs/IOPortAccess.md)
### [MSR 访问](./hypervisor-platform/funcs/MSRAccess.md)
### [CPUID 访问](./hypervisor-platform/funcs/CPUIDAccess.md)
### [虚拟处理器异常](./hypervisor-platform/funcs/VirtualProcessorException.md)
### [中断窗口](./hypervisor-platform/funcs/InterruptWindow.md)
### [不支持的功能](./hypervisor-platform/funcs/UnsupportableFeature.md)
### [已取消执行](./hypervisor-platform/funcs/ExecutionCancelled.md)
## [WHvSetPartitionProperty](./hypervisor-platform/funcs/WHvSetPartitionProperty.md)
### [数据类型](./hypervisor-platform/funcs/WHvPartitionPropertyDataTypes.md)
## [WHvSetupPartition](./hypervisor-platform/funcs/WHvSetupPartition.md)
## [WHvSetVirtualProcessorRegisters](./hypervisor-platform/funcs/WHvSetVirtualProcessorRegisters.md)
### [数据类型](./hypervisor-platform/funcs/WHvVirtualProcessorDataTypes.md)
## [WHvTranslateGva](./hypervisor-platform/funcs/WHvTranslateGva.md)
## [WHvUnmapGpaRange](./hypervisor-platform/funcs/WHvUnmapGpaRange.md)

# [虚拟机监控程序指令仿真器](./hypervisor-instruction-emulator/hypervisor-instruction-emulator.md)
## 指令仿真
### [I/O 端口访问](./hypervisor-instruction-emulator/funcs/IOPortAccessIE.md)
### [MMIO 访问](./hypervisor-instruction-emulator/funcs/MMIOAccessIE.md)
## 仿真器结构
### [WHV_EMULATOR_CALLBACKS](./hypervisor-instruction-emulator/funcs/WhvEmulatorCallbacks.md)
### [WHV_EMULATOR_IO_ACCESS_INFO](./hypervisor-instruction-emulator/funcs/WhvEmulatorIOAccessInfo.md)
### [WHV_EMULATOR_MEMORY_ACCESS_INFO](./hypervisor-instruction-emulator/funcs/WhvEmulatorMemoryAccessInfo.md)
### [WHV_EMULATOR_STATUS](./hypervisor-instruction-emulator/funcs/WhvEmulatorStatus.md)
## API 方法
### [WHvEmulatorCreateEmulator](./hypervisor-instruction-emulator/funcs/WHvEmulatorCreateEmulator.md)
### [WHvEmulatorDestoryEmulator](./hypervisor-instruction-emulator/funcs/WHvEmulatorDestoryEmulator.md)
### [WHvEmulatorTryIoEmulation](./hypervisor-instruction-emulator/funcs/WHvEmulatorTryEmulation.md)
### [WHvEmulatorTryMmioEmulation](./hypervisor-instruction-emulator/funcs/WHvEmulatorTryEmulation.md)
## 回调函数
### [WHV_EMULATOR_GET_VIRTUAL_PROCESSOR_REGISTERS_CALLBACK](./hypervisor-instruction-emulator/funcs/WHvEmulatorGetVirtualProcessorRegistersCallback.md)
### [WHV_EMULATOR_IO_PORT_CALLBACK](./hypervisor-instruction-emulator/funcs/WHvEmulatorIOPortCallback.md)
### [WHV_EMULATOR_MEMORY_CALLBACK](./hypervisor-instruction-emulator/funcs/WHvEmulatorMemoryCallback.md)
### [WHV_EMULATOR_SET_VIRTUAL_PROCESSOR_REGISTERS_CALLBACK](./hypervisor-instruction-emulator/funcs/WHvEmulatorSetVirtualProcessorRegistersCallback.md)
### [WHV_EMULATOR_TRANSLATE_GVA_PAGE_CALLBACK](./hypervisor-instruction-emulator/funcs/WHvEmulatorTranslateGVAPageCallback.md)

# [虚拟机已保存状态转储提供程序 API](./vm-dump-provider/vm-dump-provider.md)
## [ApplyPendingSavedStateFileReplay](./vm-dump-provider/funcs/ApplyPendingSavedStateFileReplayLog.md)
## [DataTypes](./vm-dump-provider/funcs/DataTypes.md)
## [GetArchitecture](./vm-dump-provider/funcs/GetArchitecture.md)
## [GetGuestPhysicalMemoryChunks](./vm-dump-provider/funcs/GetGuestPhysicalMemoryChunks.md)
## [GetGuestRawSavedMemorySize](./vm-dump-provider/funcs/GetGuestRawSavedMemorySize.md)
## [GetPagingMode](./vm-dump-provider/funcs/GetPagingMode.md)
## [GetRegisterValue](./vm-dump-provider/funcs/GetRegisterValue.md)
## [GetVpCount](./vm-dump-provider/funcs/GetVpCount.md)
## [GuestPhysicalAddressToRawSavedMemoryOffset](./vm-dump-provider/funcs/GuestPhysicalAddressToRawSavedMemoryOffset.md)
## [GuestVirtualAddressToPhysicalAddress](./vm-dump-provider/funcs/GuestVirtualAddressToPhysicalAddress.md)
## [LoadSavedStateFile](./vm-dump-provider/funcs/LoadSavedStateFile.md)
## [LoadSavedStateFiles](./vm-dump-provider/funcs/LoadSavedStateFiles.md)
## [LocateSavedStateFiles](./vm-dump-provider/funcs/LocateSavedStateFiles.md)
## [ReadGuestPhysicalAddress](./vm-dump-provider/funcs/ReadGuestPhysicalAddress.md)
## [ReadGuestRawSavedMemory](./vm-dump-provider/funcs/ReadGuestRawSavedMemory.md)
## [ReleaseSavedStateFiles](./vm-dump-provider/funcs/ReleaseSavedStateFiles.md)
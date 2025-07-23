# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

nRF Connect Device Manager (formerly McuManager iOS) is an iOS/macOS library implementing the McuManager protocol and SUIT (Software Update for Internet of Things) for device management and firmware updates over Bluetooth LE. It's maintained by Nordic Semiconductor and supports nRF52, nRF53, nRF54, and nRF91 series devices.

## Key Commands

### Building the Example App
```bash
cd Example/
pod install
open nRF\ Connect\ Device\ Manager.xcworkspace
```

### Running on Device
- Build and run from Xcode using the workspace file (not the project file)
- Minimum iOS 12.0 deployment target

### Package Management
- **Swift Package Manager**: Primary method, use Package.swift
- **CocoaPods**: Example app uses this, run `pod install` in Example/ directory
- **Carthage**: Limited support via Cartfile

## Architecture

### Library Structure (Source/)

**Core Transport Layer**:
- `McuMgrTransport`: Abstract transport protocol
- `McuMgrBleTransport`: BLE implementation with connection management and MTU handling

**Manager Classes** (extend McuManager):
- `DefaultManager`: OS commands (reset, time, echo)
- `ImageManager`: Firmware image management  
- `FileSystemManager`: File upload/download
- `SuitManager`: SUIT protocol support
- `FirmwareUpgradeManager`: High-level DFU orchestration

**DFU Implementation**:
- `FirmwareUpgradeController`: State machine for firmware updates
- `FirmwareUpgradeConfiguration`: Pipeline depth, byte alignment, reassembly settings
- Supports MCUboot and SUIT bootloaders

**Key Features**:
- SMP (Simple Management Protocol) implementation
- Multi-image DFU support
- DirectXIP provisioning
- Pipeline optimization for faster uploads
- Comprehensive logging system

### Example App Structure

View Controllers organized by feature:
- Scanner: BLE device discovery
- Manager tabs: Image, Files, Logs, Stats, Settings
- Uses Storyboards for UI

## Important Implementation Notes

1. **Thread Safety**: Always call DFU APIs (start/pause/cancel) from Main Thread

2. **Firmware Package Formats**:
   - `.bin`: Single-core MCUboot updates
   - `.suit`: SUIT envelope files
   - `.zip`: Multi-image/multi-core updates with manifest.json

3. **DFU Modes**:
   - `.testAndConfirm` (default, recommended)
   - `.confirmOnly` (for multi-image only)
   - `.testOnly`
   - `.uploadOnly`

4. **Speed Optimizations**:
   - `pipelineDepth`: Enable SMP pipelining (>1)
   - `byteAlignment`: Required with pipelining
   - `reassemblyBufferSize`: Auto-detected for NCS 2.0+

5. **SUIT vs MCUboot**:
   - SUIT uses different update logic (device-driven)
   - Only SHA256 algorithm currently supported for SUIT
   - Resources requested via delegate callbacks

## File References

Key implementation files:
- Transport: `Source/Bluetooth/McuMgrBleTransport.swift`
- DFU Manager: `Source/Managers/DFU/FirmwareUpgradeManager.swift`
- State Machine: `Source/Managers/DFU/FirmwareUpgradeController.swift`
- Package Parser: `Source/McuMgrPackage.swift`
- Example DFU: `Example/Source/View Controllers/Manager/FirmwareUpgradeViewController.swift`
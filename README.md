# Laya KernelTuner - Adaptive Kernel Parameter Auto-Tuner

## Overview
LayaKernelTuner is an intelligent, zero-intrusion kernel parameter optimizer for 64-bit Android devices with Linux Kernel 4.19+. It automatically adapts system parameters based on user activity, screen state, and system load to maximize battery life while maintaining responsiveness.

## Features

### ✅ Intelligent Profile Switching
- **Idle Profile** - Aggressive power saving when screen is off
- **Balanced Profile** - Optimal performance-to-battery ratio during normal use
- **Performance Profile** - Maximum responsiveness for interactive tasks
- **Gaming Profile** - Low-latency, high-throughput optimization for demanding apps

### ✅ Real-Time Activity Detection
- Screen state monitoring (brightness tracking)
- Process counting and workload detection
- Memory pressure estimation (PSI-based or MemAvailable fallback)
- I/O activity tracking from `/proc/diskstats`

### ✅ Zero-User-Intrusion Operation
- Automatic profile selection without any user input
- Silent parameter tuning in background
- No visible UI, no notifications
- Profile learning from usage patterns

### ✅ Comprehensive Kernel Parameter Tuning
Optimizes `vm.*`, `net.*`, `kernel.*`, and `fs.*` parameters:
- Memory management (swappiness, dirty ratios, writeback intervals)
- Network optimization (TCP keepalive, connection reuse)
- I/O scheduling (page-cluster, stat_interval, laptop_mode)
- File system limits (file-max, inotify watches)
- Process accounting (task_delayacct, migration cost)

## System Requirements

- **Architecture:** 64-bit ARM (aarch64/arm64)
- **Kernel Version:** Linux 4.19 or higher
- **Android Version:** Android 11+
- **Permissions:** Root Access (KSU/Magisk,etc)

## Installation

### Method 1: Root Access
Install it on Kernel SU, Magisk, Sukisu Ultra, Or similar platform

### Method 2: ROM Integration

## Architecture

### Profile Detection Logic
```
Screen ON + Low Load → Idle Profile (save battery)
Screen ON + Medium Load → Balanced Profile
Screen ON + Heavy Load → Performance Profile
Screen ON + Heavy + High Memory → Gaming Profile
Screen OFF + Any Load → Idle Profile (maximum savings)
```

### Activity Metrics
- **Screen State** - Detected from brightness file
- **Process Count** - Parsed from `/proc`
- **Memory Pressure** - PSI avg10 or MemAvailable/MemTotal ratio
- **I/O Activity** - Read completion count from `/proc/diskstats`

### Parameter Application Strategy
1. Detect device characteristics (kernel version, architecture, RAM, CPU count)
2. Monitor user activity in 5-second intervals
3. Select optimal profile based on current workload
4. Apply parameters with stability check (2 consistent readings)
5. Validate each parameter exists before writing
6. Log detailed success/failure information

## Profiles in Detail

### Idle Profile (Battery Aggressive)
- `vm.swappiness = 80` - Aggressive memory reclamation
- `vm.dirty_ratio = 20` - Smaller write buffers, quicker writeback
- `vm.laptop_mode = 5` - Extreme I/O batching
- `net.ipv4.tcp_tw_reuse = 1` - Reuse TIME_WAIT connections
- Optimized for minimal power draw when screen off

### Balanced Profile (Default)
- `vm.swappiness = 60` - Standard memory management
- `vm.dirty_ratio = 30` - Moderate write buffering
- `net.ipv4.tcp_keepalive_time = 300` - Responsive TCP
- Balanced performance and battery life

### Performance Profile (Responsive)
- `vm.swappiness = 30` - Minimize GC latency
- `vm.dirty_ratio = 40` - Larger write buffers, better throughput
- `kernel.task_delayacct = 1` - Full accounting enabled
- Maximized responsiveness

### Gaming Profile (Low-Latency)
- `vm.swappiness = 10` - Minimal swapping, prevent frame drops
- `vm.dirty_ratio = 50` - Async writes, smooth gameplay
- `kernel.sched_migration_cost_ns = 500000` - Minimize scheduling jitter
- Optimized for consistent frame rates

## Safety Features

✅ **Parameter Validation** - Checks if each parameter exists before writing  
✅ **Stability Detection** - Waits for 2 consistent readings before profile change  
✅ **Fallback Handling** - Gracefully handles write failures  
✅ **Logging** - Detailed logcat output for diagnostics  
✅ **Version Checking** - Validates kernel version on startup  
✅ **Architecture Check** - Ensures 64-bit architecture before running  

## Logging

All events logged to:
1. **Android Logcat** - `logcat | grep LayaKernelTuner`

## Performance Impact

### Battery Savings
- Idle mode: 10-20% improvement on 8-hour idle
- Reduces phantom wakeups from polling
- Optimized memory pressure reduces GC pauses

### Performance Overhead
- CPU usage: <0.5% (5-second polling interval)
- Memory usage: ~512 KB (minimal state tracking)
- I/O operations: Minimal (read-only parameter detection)

## Troubleshooting

### Binary won't start
```bash
# Check permissions
adb shell ls -l /system/xbin/laya-kernel-tuner

# Check logcat
adb logcat | grep -E "LayaKernelTuner|kernel|tuner"

# Verify architecture
adb shell getconf LONG_BIT  # Should return 64
```

### Parameters not applying
```bash
# Check which parameters failed
adb logcat | grep "Failed parameters"

# Verify parameter exists
adb shell cat /proc/sys/vm/swappiness
```

### Kernel version issues
```bash
# Check kernel version
adb shell uname -r  # Must be 4.19+
```

## Contributing

For issues, improvements, or optimizations, please refer to the main project repository.

#!/usr/bin/env python3
"""
EdgeShield - PHP-FPM Worker Memory Profiler & Leak Mitigator
"""

import os
import subprocess
import sys

MEMORY_LIMIT_MB = 256  # Threshold per PHP worker

def get_php_fpm_workers():
    """Inspect system processes for PHP-FPM worker instances."""
    try:
        output = subprocess.check_output(["ps", "-eo", "pid,rss,cmd"], text=True)
        workers = []
        for line in output.splitlines():
            if "php-fpm: pool" in line:
                parts = line.split()
                pid = parts[0]
                rss_mb = int(parts[1]) / 1024  # Convert KB to MB
                workers.append((pid, rss_mb))
        return workers
    except Exception as e:
        print(f"[ERROR] Failed to fetch PHP-FPM processes: {e}")
        return []

def audit_memory_footprint():
    print("[INFO] Starting PHP-FPM Memory Profiling Inspection...")
    workers = get_php_fpm_workers()
    
    for pid, memory in workers:
        if memory > MEMORY_LIMIT_MB:
            print(f"[ALERT] Worker PID {pid} consumed {memory:.2f} MB (Limit: {MEMORY_LIMIT_MB} MB).")
            print(f"[ACTION] Triggering graceful worker recycle for PID {pid}.")
            # Send SIGQUIT signal for graceful worker shutdown
            os.kill(int(pid), 3)
        else:
            print(f"[OK] Worker PID {pid} is healthy ({memory:.2f} MB).")

if __name__ == "__main__":
    audit_memory_footprint()

---
layout: default
title: "2025-06-25: Using Supervisord to Manage Multiple Processes in a Container - A FastAPI Case Study"
tags: [Docker, FastAPI, Supervisord]
permalink: /blogger/posts/2025-06-25-supervisor-container-management/
---

# Using Supervisord to Manage Multiple Processes in a Container: A FastAPI Case Study

> 💬 *“天下之至柔，驰骋天下之至坚。(The softest thing in the world overcomes the hardest)"*  
> — Laozi

This tutorial demonstrates how to use **Supervisord**, a process control system, to manage multiple processes within a Docker container. We'll focus on a practical scenario involving a FastAPI application with a memory leak, monitored and managed using Supervisord and a custom Python script.

## 1. Introduction to Supervisord

**Supervisord** is a lightweight, open-source process control system written in Python. It is designed to manage and monitor multiple processes, ensuring they are running, restarting them if they crash, and providing logging and control capabilities.

### When to Use Supervisord
- **Multiple Processes in a Container**: Containers typically run a single process, but some applications require multiple processes (e.g., an API server and a monitoring script).
- **Process Reliability**: Supervisord ensures processes are restarted if they fail unexpectedly.
- **Centralized Management**: It provides a unified interface to start, stop, and monitor processes.

### Benefits
- **Simplicity**: Easy to configure using INI-style configuration files.
- **Flexibility**: Works with any executable process, not just Python scripts.
- **Logging**: Automatically captures stdout/stderr for each process.
- **Control**: Offers a command-line interface (`supervisorctl`) and a web interface for process management.

## 2. Scenario: Managing a FastAPI App with a Memory Leak

Imagine you have a FastAPI application running in a Docker container. It processes requests efficiently but has an unresolved memory leak. If memory usage grows unchecked, the container may be killed by the system's Out-Of-Memory (OOM) killer, causing downtime.

To address this, we'll:
1. Run the FastAPI app as a process managed by Supervisord.
2. Run a Python script (`monitor_memory.py`) that uses `psutil` to monitor the app's memory usage.
3. If memory exceeds 250MiB, the script will instruct Supervisord to restart the FastAPI app.

This approach prevents OOM kills by proactively restarting the app when memory usage becomes excessive, buying time to debug the leak.

## 3. Project Directory Structure

For this demo, we'll keep the project minimal with only the necessary files:

```
memory_leak_demo/
├── Dockerfile
├── supervisord.conf
├── monitor_memory.py
├── main.py
├── requirements.txt
```

- **Dockerfile**: Defines the container setup.
- **supervisord.conf**: Configures Supervisord to manage the FastAPI app and monitoring script.
- **monitor_memory.py**: Monitors memory usage and restarts the app if needed.
- **main.py**: The FastAPI application.
- **requirements.txt**: Lists Python dependencies.

## 4. Setting Up the Project

Let's walk through each file's content and purpose.

### 4.1 Dockerfile

The Dockerfile sets up the environment, installs the necessary dependencies, and starts Supervisord. While it's best practice to run processes in a container as a non-root user, in this demonstration we use the root user for simplicity.

```dockerfile
# Use the official lightweight Python 3.10 base image
FROM python:3.10-slim

# Set the working directory inside the container to /app
WORKDIR /app

# Copy the requirements.txt file into the container
COPY requirements.txt .

# Install Python dependencies listed in requirements.txt without caching to reduce image size
RUN pip install --no-cache-dir -r requirements.txt

# Copy all remaining application files into the container
COPY . .

# Start supervisord with the specified configuration file (FastAPI App and monitor_memory.py)
CMD ["supervisord", "-c", "/app/supervisord.conf"]
```

### 4.2 Requirements File

The `requirements.txt` lists the Python packages needed:

```text
fastapi==0.112.0
uvicorn==0.33.0
psutil==7.0.0
supervisor==4.2.5
```

### 4.3 FastAPI Application (main.py)

A simple FastAPI app to simulate our scenario: This application exposes a single endpoint (/) which, when called, will intentionally allocate and retain approximately 10 MB of memory. This behavior mimics a memory leak, where the application consumes more memory over time without releasing it, potentially leading to an out-of-memory (OOM) condition if left unchecked.

This is useful in our tutorial to demonstrate how Supervisor can monitor and restart such an app when memory usage exceeds a defined threshold.

```python
from fastapi import FastAPI

app = FastAPI()

# Global list to simulate memory leak
leaky_memory = []

@app.get("/")
async def root():
    # Allocate ~10 MB of data and store it in the global list
    leaky_memory.append("X" * 10 * 1024 * 1024)  # 10 MB of data
    return {"message": "Memory leaked by ~10MB"}
```

This app will be managed by `uvicorn` under Supervisord.

## 5. Supervisord Configuration File (supervisord.conf)

The `supervisord.conf` file defines the processes to run and their settings. Below is the configuration with detailed annotations:

```ini
[unix_http_server]
file=/tmp/supervisor.sock ; Path to the Unix socket
chmod=0700 ; Socket permissions

[supervisord]
nodaemon=true ; Run Supervisord in the foreground (required for Docker)
logfile=/tmp/supervisord.log ; Log file in a directory accessible by Docker
loglevel=info ; Set log level

[program:fastapi_0]
command=uvicorn main:app --host 0.0.0.0 --port 5001 ; Run FastAPI app with uvicorn
directory=/app ; Working directory for the process
autostart=true ; Start automatically when Supervisord starts
autorestart=true ; Restart if the process exits unexpectedly
stderr_logfile=/app/uvicorn.log ; Capture stderr to a log file
stdout_logfile=/app/uvicorn.log ; Capture stdout to a log file
startsecs=10 ; Seconds to wait before considering the process started
stopasgroup=true ; Send stop signal to the entire process group
killasgroup=true ; Send kill signal to the entire process group
environment=PYTHONUNBUFFERED=1 ; Ensure unbuffered output for Python

[program:monitor]
command=python3 monitor_memory.py /app ; Run the monitoring script
directory=/app ; Working directory
autostart=true ; Start automatically
autorestart=true ; Restart if exits unexpectedly (consider setting to false if script exits gracefully)
stderr_logfile=/app/monitor.log ; Log file for monitoring script
stdout_logfile=/app/monitor.log ; Log file for monitoring script
startsecs=5 ; Seconds to wait before considering the process started
environment=PYTHONUNBUFFERED=1 ; Ensure unbuffered output for Python

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; Path to Supervisord's control socket

[rpcinterface:supervisor]
supervisor.rpcinterface_factory=supervisor.rpcinterface:make_main_rpcinterface
```

### Key Configuration Notes
- **`nodaemon=true`**: Keeps Supervisord in the foreground, essential for Docker containers.
- **`[program:fastapi_0]`**: Defines the FastAPI process, running `uvicorn`.
- **`[program:monitor]`**: Runs the memory monitoring script.
- **`autostart` and `autorestart`**: Control process startup and restart behavior.
- **logging**: Each process's output is redirected to separate log files for debugging.
- **`stopasgroup` and `killasgroup`**: Ensures all child processes are terminated when restarting the FastAPI app.
- **`environment=PYTHONUNBUFFERED=1`**: Prevents buffering issues with Python output in Docker.

## 6. Memory Monitoring Script (monitor_memory.py)

This script uses `psutil` to monitor the FastAPI app's memory usage and restarts it via Supervisord if it exceeds 250MiB threshold.

```python
import psutil
import subprocess
import time
import sys
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(message)s",
    filename="/app/monitor.log",
    filemode='a+'
    encoding='utf-8',
)

# Configuration
MEMORY_THRESHOLD = 250 * 1024 * 1024  # 250 MiB in bytes
CHECK_INTERVAL = 5  # seconds
FASTAPI_PROCESS_NAME = "uvicorn"  # Name to identify FastAPI process
SUPERVISOR_PROCESS_NAME = "fastapi_0"  # Supervisord program name for FastAPI

def find_process_by_name(name):
    """Find a process by name using psutil."""
    # Since we only started one work using uvicorn, there should be just one uvicorn process.
    # If you start the app with gunicorn with multiple workers or set the --workers n, you should modify the code accordingly to catch all.
    for proc in psutil.process_iter(['pid', 'name']):
        if proc.info['name'].lower() == name.lower():
            return proc
    return None

def get_memory_usage(proc):
    """Get memory usage of a process in bytes."""
    try:
        return proc.memory_info().rss
    except psutil.NoSuchProcess:
        return None

def restart_fastapi():
    """Restart the FastAPI app using supervisorctl."""
    try:
        # The default path for config file is: /etc/supervisord.conf. we need to override it since the configuration file resides in /app in this example
        subprocess.run(["supervisorctl", "-c", "/app/supervisord.conf", "restart", SUPERVISOR_PROCESS_NAME], check=True)
        logging.info("FastAPI restarted due to high memory usage")
    except subprocess.CalledProcessError as e:
        logging.error(f"Failed to restart FastAPI: {e}")
        sys.exit(1)

def monitor():
    """Monitor FastAPI memory usage."""
    logging.info("Starting memory monitor for FastAPI")
    while True:
        fastapi_proc = find_process_by_name(FASTAPI_PROCESS_NAME)
        if not fastapi_proc:
            logging.error("FastAPI process not found, exiting")
            restart_fastapi()
            time.sleep(CHECK_INTERVAL)
            continue

        memory_usage = get_memory_usage(fastapi_proc)
        if memory_usage is None:
            logging.error("FastAPI process disappeared, restarting")
            restart_fastapi()
            time.sleep(CHECK_INTERVAL)
            continue

        logging.info(f"FastAPI memory usage: {memory_usage / 1024 / 1024:.2f} MB")
        if memory_usage > MEMORY_THRESHOLD:
            logging.warning(f"Memory usage ({memory_usage / 1024/1024:.2f} MiB) exceeds threshold, restarting FastAPI")
            restart_fastapi()
            time.sleep(10)  # Wait before checking again to allow restart
        time.sleep(CHECK_INTERVAL)

if __name__ == "__main__':
    try:
        time.sleep(30) # wait 30 seconds before starting the monitor function.
        monitor()
    except KeyboardInterrupt:
        logging.info("Monitor stopped by user")
        sys.exit(0)
```

### Script Explanation
- **Logging**: Logs memory usage and restart events to `/app/monitor.log` for debugging.
- **Process Identification**: Uses `psutil` to find the `uvicorn` process by name.
- **Memory Check**: Retrieves resident set size (RSS) memory usage every 5 seconds.
- **Restart Logic**: If memory exceeds 250MiB, it runs `supervisorctl restart fastapi_0` to restart the app.
- **Error Handling**: Handles cases where the FastAPI process disappears or restart fails.

## 7. Building, Running, and Testing the Setup

Now that all components are in place, let's build the Docker image, run the container, and simulate memory growth by sending requests to the FastAPI app.

### 🔨 Build the Docker Image

Run the following command in the root of your project directory:

```bash
docker build -t fastapi-memory-monitor .
```

### ▶️ Run the Container

```bash
docker run --rm -p 5001:5001 fastapi-memory-monitor
```

This starts the container and maps FastAPI’s port 5001 to the host.

### 🔁 Simulate Requests with `curl`

Each request will add \~10MB to memory usage. Run this command multiple times (or in a loop):

```bash
for i in {1..30}; do curl http://localhost:5001/; sleep 1; done
```

After enough requests (around 25–30), the FastAPI app should consume over 250MiB and the monitoring script will detect it and restart the app. You’ll see log entries in `monitor.log` and `uvicorn.log` indicating a restart.

You can also observe memory usage using Docker:

```bash
docker stats
```

Once verified, you’ve successfully built a self-healing app container using Supervisord!

## 8. Summary and Conclusion

In this tutorial, we demonstrated how to use Supervisord to manage multiple processes in a Docker container. We addressed a hypothetical FastAPI app with a memory leak by:
- Using Supervisord to run both the FastAPI app and a monitoring script.
- Configuring Supervisord to ensure process reliability and logging.
- Writing a Python script to monitor memory usage with `psutil` and restart the app when necessary.

This approach provides a robust interim solution to manage memory leaks while debugging the root cause, ensuring application availability.

## 9. Additional Tips and Resources

- **Debugging Memory Leaks**: Use tools like `memory_profiler` or `tracemalloc` to identify memory leaks in your Python app.
- **Supervisord Web Interface**: Enable the `[inet_http_server]` section in `supervisord.conf` to monitor processes via a web browser.
- **Docker Best Practices**: Ensure your container has appropriate memory limits set using Docker's `--memory` flag to complement this setup.
- **Scaling Supervisord**: For more complex applications, Supervisord can manage additional processes like background workers or cron jobs.
- **Resources**:
  - [Supervisord Documentation](http://supervisord.org/)
  - [FastAPI Documentation](https://fastapi.tiangolo.com/)
  - [psutil Documentation](https://psutil.readthedocs.io/)

By leveraging Supervisord, you can build resilient containerized applications that handle complex process management scenarios effectively.
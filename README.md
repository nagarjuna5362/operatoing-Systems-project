# 🖥️ Hardware Introspection Agent

**A specialized AI-powered Linux hardware inspection agent for the Agentic Shell**

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Overview

The Hardware Introspection Agent accepts natural-language questions about computer hardware and answers them by collecting **real hardware information** from the Linux operating system. It is one of 35 specialized agents in a next-generation Agentic Shell.

### Key Features

- 🔍 **Real hardware data** — Never guesses or invents specifications
- 🛡️ **Read-only & safe** — No system modifications, no arbitrary commands
- 🧠 **LLM-powered** — Uses function calling for intelligent tool selection
- 🔌 **Modular design** — Easy to integrate into the master Agentic Shell
- 📊 **Comprehensive** — CPU, RAM, GPU, storage, USB, PCI, motherboard, BIOS

---

## Table of Contents

1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Usage](#usage)
4. [Architecture](#architecture)
5. [Available Tools](#available-tools)
6. [Integration with Master Shell](#integration-with-master-shell)
7. [Testing](#testing)
8. [Example Conversations](#example-conversations)
9. [Sample Hardware Report](#sample-hardware-report)
10. [Known Limitations](#known-limitations)
11. [Troubleshooting](#troubleshooting)

---

## Installation

### Prerequisites

**Operating System:** Ubuntu Linux (20.04+ recommended)

**Required Linux packages:**

```bash
# Install hardware inspection utilities
sudo apt update
sudo apt install -y dmidecode lshw pciutils usbutils util-linux procps coreutils
```

- `dmidecode` — BIOS/motherboard/memory slot details (requires root)
- `pciutils` — Provides `lspci` for PCI device listing
- `usbutils` — Provides `lsusb` for USB device listing
- `util-linux` — Provides `lsblk` and `lscpu`

**Python:** 3.10 or higher

### Setup

```bash
# 1. Navigate to the project directory
cd "operating Systems project"

# 2. Create a virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate

# 3. Install Python dependencies
pip install -r requirements.txt

# 4. Configure your LLM API key
cp .env.example .env
nano .env   # Add your API key
```

---

## Configuration

Copy `.env.example` to `.env` and set your API key:

```bash
# Google Gemini (default provider)
LLM_PROVIDER=google
GOOGLE_API_KEY=your_api_key_here
GEMINI_MODEL=gemini-3.8-flash
```

**Get a free Gemini API key:** https://aistudio.google.com/apikey

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_PROVIDER` | `google` | LLM provider (`google` or `openai`) |
| `GOOGLE_API_KEY` | — | Google Gemini API key |
| `OPENAI_API_KEY` | — | OpenAI API key |
| `GEMINI_MODEL` | `gemini-3.8-flash` | Gemini model name |
| `OPENAI_MODEL` | `gpt-4o-mini` | OpenAI model name |
| `COMMAND_TIMEOUT` | `10` | Max seconds for system commands |
| `MASK_SERIALS` | `false` | Mask serial numbers in output |

---

## Usage

### Interactive CLI

```bash
# Standard mode (with LLM)
python main.py

# For full hardware details (requires root for dmidecode)
sudo python main.py
```

### Example Session

```
╔══════════════════════════════════════════════════════════════╗
║          🖥️  Hardware Introspection Agent v1.0.0           ║
║                                                              ║
║   Ask any question about your hardware in plain English.     ║
║   Type 'exit' or 'quit' to leave.                            ║
╚══════════════════════════════════════════════════════════════╝

Hardware Introspection Agent > What RAM do I have installed?

🔍 Processing: "What RAM do I have installed?"

  🔧 Calling tool: get_memory_info({})

📋 Answer:
────────────────────────────────────────────────────────────────
Your system has **16 GB** of **DDR4** RAM running at **3200 MT/s**,
manufactured by Kingston. You have 2 out of 4 DIMM slots populated
(2 slots are empty and available for expansion).
────────────────────────────────────────────────────────────────

Hardware Introspection Agent > exit
```

### Offline Fallback

If no API key is configured, the agent remains conversational and uses the
validated local keyword router. It still executes only the predefined
read-only hardware tools and explains results in plain English; it never
prints raw JSON as the normal response:

```bash
python main.py
# Uses the safe local conversational fallback if no API key is set
```

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Master Agentic Shell               │
│            handle_hardware_query(query)              │
└──────────────────────┬──────────────────────────────┘
                       │
         ┌─────────────▼─────────────┐
         │    agent.py (LLM Agent)   │
         │  • Receives NL question   │
         │  • Selects tools via LLM  │
         │  • Returns NL answer      │
         └─────────────┬─────────────┘
                       │ function calling
         ┌─────────────▼─────────────┐
         │   tools.py (11 Tools)     │
         │  • get_memory_info()      │
         │  • get_cpu_info()         │
         │  • get_gpu_info()         │
         │  • get_motherboard_info() │
         │  • get_usb_devices()      │
         │  • get_pci_devices()      │
         │  • get_storage_info()     │
         │  • get_system_summary()   │
         │  • get_system_health()    │
         │  • get_hardware_report()  │
         │  • search_hardware_info() │
         └─────────────┬─────────────┘
                       │
         ┌─────────────▼─────────────┐
         │  system_utils.py          │
         │  • run_command() (safe)   │
         │  • read_proc_file()       │
         │  • Parsing helpers        │
         └─────────────┬─────────────┘
                       │
         ┌─────────────▼─────────────┐
         │   Linux OS / Hardware     │
         │  dmidecode, lscpu, lspci  │
         │  lsusb, lsblk, /proc     │
         └───────────────────────────┘
```

### Module Responsibilities

| Module | Responsibility |
|--------|---------------|
| `main.py` | Interactive CLI, user input loop |
| `hardware_agent/agent.py` | LLM integration, tool calling, response generation |
| `hardware_agent/tools.py` | 10 hardware inspection functions + tool registry |
| `hardware_agent/system_utils.py` | Safe command execution, output parsing |
| `hardware_agent/config.py` | Environment/configuration management |
| `tests/test_tools.py` | Unit tests with mocked system outputs |

---

## Available Tools

| # | Tool | Description |
|---|------|-------------|
| 1 | `get_memory_info()` | RAM capacity, type, speed, slots, manufacturer (dmidecode, free, /proc/meminfo) |
| 2 | `get_cpu_info()` | Processor model, cores, threads, frequency (lscpu, /proc/cpuinfo) |
| 3 | `get_gpu_info()` | Graphics adapters and model information (lspci -nn) |
| 4 | `get_motherboard_info()` | Motherboard manufacturer, BIOS version (dmidecode) |
| 5 | `get_usb_devices()` | Connected USB devices listing (lsusb) |
| 6 | `get_pci_devices()` | PCI devices (network, graphics, etc.) (lspci) |
| 7 | `get_storage_info()` | Storage devices, capacities, mount points (lsblk, df -h) |
| 8 | `get_system_summary()` | OS info, hostname, uptime, architecture (platform, /etc/os-release, /proc/uptime) |
| 9 | `get_system_health()` | CPU load averages, available memory, disk usage (for "Why is my computer running slowly?") |
| 10 | `get_hardware_report()` | Complete combined hardware report |
| 11 | `search_hardware_info(query)` | Query router — selects the right tool |

---

## Integration with Master Shell

The agent exposes a simple interface for the master Agentic Shell:

```python
from hardware_agent.agent import handle_hardware_query, AGENT_METADATA

# Process any hardware question
answer = handle_hardware_query("What RAM do I have?")
print(answer)

# Discover agent capabilities
print(AGENT_METADATA["id"])           # "hardware_introspection_agent"
print(AGENT_METADATA["capabilities"]) # List of capability strings
print(AGENT_METADATA["tools"])        # List of available tool names
```

### Integration Pattern

```python
# In the master shell's agent router:
AGENTS = {
    "hardware_introspection_agent": {
        "handler": handle_hardware_query,
        "metadata": AGENT_METADATA,
        "keywords": ["hardware", "cpu", "ram", "gpu", "storage", "usb", ...],
    },
    # ... other 34 agents ...
}

def route_query(user_query):
    # The master LLM or keyword router selects the right agent
    agent_id = select_agent(user_query)
    return AGENTS[agent_id]["handler"](user_query)
```

---

## Testing

### Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ -v --tb=short

# Run a specific test class
pytest tests/test_tools.py::TestGetMemoryInfo -v
```

### Test Coverage

Tests cover:
- ✅ Correct RAM information extraction
- ✅ Correct empty RAM slot calculation
- ✅ CPU and GPU information retrieval
- ✅ Missing Linux utilities handling
- ✅ Permission-denied errors
- ✅ Malformed command output
- ✅ LLM tool selection for different queries
- ✅ Unknown hardware information handling
- ✅ Safe handling of arbitrary/malicious input
- ✅ Master shell integration interface
- ✅ System utils parsing helpers

All tests use **mocked system command outputs** — no physical hardware or live LLM API required.

---

## Example Conversations

### 1. RAM Query
```
User:  What kind of RAM do I have installed?
Agent: Your system has 16 GB of DDR4 RAM running at 3200 MT/s. Two 8 GB
       DIMMs from Kingston are installed in slots A1 and A2. You have
       2 empty slots (B1, B2) available for expansion.
```

### 2. CPU Query
```
User:  What is my processor model and speed?
Agent: You have an Intel Core i7-10700K running at 3.80 GHz base frequency
       with a max turbo of 5.10 GHz. It has 4 cores and 8 threads on the
       x86_64 architecture.
```

### 3. GPU Query
```
User:  Which graphics card is installed?
Agent: Your system has two graphics adapters:
       1. Intel CometLake-S GT2 UHD Graphics 630 (integrated)
       2. NVIDIA GeForce RTX 3060 (discrete, PCI slot 02:00.0)
```

### 4. Storage Query
```
User:  What storage devices are connected?
Agent: Your system has two storage devices:
       • Samsung SSD 870 (500 GB SATA) — mounted at /
       • Samsung SSD 980 PRO (1 TB NVMe) — with /boot/efi and /home partitions
```

### 5. BIOS Query
```
User:  Show my BIOS version.
Agent: Your BIOS is from American Megatrends Inc., version 2401,
       released on 03/15/2024, with a 16 MB ROM.
```

---

## Sample Hardware Report

When you ask for a "complete hardware report", the agent returns:

```
HARDWARE REPORT
===============

SYSTEM
  Hostname:      my-workstation
  OS:            Ubuntu 22.04.3 LTS
  Kernel:        5.15.0-91-generic
  Architecture:  x86_64
  Uptime:        3d 14h 22m

CPU
  Model:     Intel Core i7-10700K @ 3.80GHz
  Cores:     4 physical, 8 threads
  Max Freq:  5100 MHz

MEMORY
  Total:     16 GB DDR4 @ 3200 MT/s
  Slots:     2/4 occupied (2 empty)
  Modules:   2× 8 GB Kingston KHX3200C16/8G

GPU
  1. Intel UHD Graphics 630 (integrated)
  2. NVIDIA GeForce RTX 3060 (discrete)

MOTHERBOARD
  Manufacturer:  ASUSTeK COMPUTER INC.
  Model:         ROG STRIX Z490-E GAMING

BIOS
  Vendor:   American Megatrends Inc.
  Version:  2401 (03/15/2024)

STORAGE
  • sda   — 500 GB (Samsung SSD 870)     → /
  • nvme0 — 1 TB   (Samsung SSD 980 PRO) → /boot/efi, /home

USB DEVICES (4)
  • Linux Foundation 2.0 root hub
  • Logitech Unifying Receiver
  • Linux Foundation 3.0 root hub
  • Kingston Technology HyperX Alloy
```

*Note: The above is illustrative. Actual output depends on your hardware.*

---

## Known Limitations

1. **Root privileges required** — `dmidecode` needs root/sudo for BIOS, motherboard, and detailed RAM slot information. Without root, the agent falls back to `/proc/meminfo` for basic memory data.

2. **Linux only** — The agent uses Linux-specific utilities (`dmidecode`, `lscpu`, `lspci`, `lsusb`, `lsblk`) and `/proc` filesystem. It will not work on Windows or macOS.

3. **Virtual machines** — Some hardware details may be unavailable or show hypervisor information in VMs.

4. **LLM fallback** — Without an API key, the safe local router supports common
   hardware questions and maintains the interactive session, but cannot resolve
   nuanced references as reliably as the LLM.

5. **SMBIOS/firmware coverage** — RAM slot details depend on the motherboard firmware exposing them via SMBIOS. If the firmware doesn't report slot data, the agent returns "Unknown" rather than guessing.

6. **GPU details** — Detailed GPU information (VRAM, driver version) requires additional tools like `nvidia-smi` which are not included in the base toolset.

---

## Troubleshooting

### "Command 'dmidecode' is not installed"
```bash
sudo apt install dmidecode
```

### "Permission denied" errors
Run the agent with sudo for full hardware details:
```bash
sudo python main.py
```

### "No API key configured"
1. Get a free API key from https://aistudio.google.com/apikey
2. Copy `.env.example` to `.env`
3. Add your key: `GOOGLE_API_KEY=your-key-here`

### lsusb / lspci not found
```bash
sudo apt install usbutils pciutils
```

### Empty or incomplete hardware data
- Check if you're running on bare metal vs. a virtual machine
- Some fields require SMBIOS support from the motherboard firmware
- Run with `sudo` for maximum hardware visibility

---

## Security Notes

- ✅ **Read-only** — No system modifications are ever made
- ✅ **No shell=True** — All commands use safe argument lists
- ✅ **No arbitrary execution** — Only predefined tool functions can run
- ✅ **Timeouts enforced** — Commands that hang are killed
- ✅ **Input validation** — Tool inputs are validated before execution
- ✅ **Serial masking** — Optional serial number masking for privacy

---

## Project Structure

```
operating Systems project/
├── main.py                    # Interactive CLI entry point
├── requirements.txt           # Python dependencies
├── .env.example               # Example environment configuration
├── README.md                  # This file
├── hardware_agent/
│   ├── __init__.py
│   ├── config.py              # Environment & configuration
│   ├── system_utils.py        # Safe command execution & parsing
│   ├── tools.py               # 10 hardware inspection functions
│   └── agent.py               # LLM integration & tool calling
└── tests/
    ├── __init__.py
    └── test_tools.py           # Unit tests (mocked outputs)
```

---

## License

This project is developed as a B.Tech Operating Systems project.
#   o p e r a t o i n g - S y s t e m s - p r o j e c t  
 
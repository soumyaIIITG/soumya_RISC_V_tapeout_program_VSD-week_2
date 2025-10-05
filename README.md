# 🧩 Fundamentals of System-on-Chip (SoC) Design

## 📌 What is a System-on-Chip (SoC)?

A **System-on-Chip (SoC)** is like squeezing an entire computer onto a single chip. Instead of having the CPU, memory, and peripherals on separate chips, everything is integrated into one piece of silicon.

✅ **Why it matters:**

* Smaller and more compact devices.
* Lower power consumption.
* Faster communication between components.
* Cost-effective when mass-produced.

This is why our smartphone can run complex apps, games, and AI features while fitting in our pocket — all powered by its SoC.

---

## 🧩 Components of a Typical SoC

1. **CPU (Processor Core)** – The “brain” of the system, runs programs and makes decisions.
2. **Memory** – On-chip SRAM/ROM for speed + connections to off-chip DRAM/Flash.
3. **Peripherals** – Interfaces to the outside world: UART, SPI, I²C, USB, Ethernet, GPIO, plus accelerators like GPUs or AI engines.
4. **Interconnect** – The “highways” inside the chip that connect everything. Simple SoCs use **buses**, advanced ones use **Network-on-Chip (NoC)**.


<img width="918" height="648" alt="image" src="https://github.com/user-attachments/assets/07fb2968-1a80-4853-81d8-bd33247f2c8b" />

---



## 🍼 Why BabySoC?

**BabySoC** is a simplified learning model — like a training bicycle for SoC design. It typically includes:

* A tiny CPU core (often RISC-V).
* Minimal memory.
* Simple peripherals (UART, GPIO).
* A basic bus interconnect.

🎯 **Purpose:**

* Focus on concepts without overwhelming complexity.
* Learn how CPUs talk to memory and peripherals.
* Understand how interconnects glue everything together.

Once we grasp BabySoC, it’s much easier to move on to industrial-grade SoCs with multiple CPUs, GPUs, and advanced interconnects.

---

## 🛠️ Role of Functional Modelling

SoC design isn’t done in one go — it’s a journey:

1. **Functional Modelling (High-Level)** – Use C, SystemC, or Python to describe what the system should do. Answer: *“Does this idea work?”*
2. **RTL Design (Register-Transfer Level)** – Translate the concept into hardware descriptions (Verilog, VHDL, SystemVerilog).
3. **Physical Design** – Turn RTL into an actual silicon layout that can be fabricated.

💡 **Why functional modelling first?**

* Catch mistakes early (before costly silicon).
* Explore trade-offs quickly.
* Provide a reference model for RTL verification.

It’s like sketching a blueprint before building a house.

---

# 🧩 Part 2 — Simulation & Verification of VSDBabySoC

## 🔹 Overview — What is VSDBabySoC?

**VSDBabySoC** is a compact System-on-Chip (SoC) that integrates both **digital** and **analog** IP blocks.
It demonstrates SoC-level integration, mixed-signal interaction, and analog output verification.

### Components

* **RVMYTH:** Simple RISC-V CPU core
* **PLL:** 8× Phase-Locked Loop for clock stability
* **DAC:** 10-bit Digital-to-Analog Converter

### Objectives

* Integrate multiple IPs into a unified SoC
* Calibrate and verify analog output from digital inputs

📸 <img width="1262" height="700" alt="image" src="https://github.com/user-attachments/assets/4f70720f-63cb-4712-b4ec-5684e4e323fa" />


---

## 🔹 Project Structure

```
VSDBabySoC/
├── src/
│   ├── include/        # Header files (*.vh)
│   ├── module/         # Verilog + TLV modules
│   │   ├── vsdbabysoc.v   # Top-level SoC
│   │   ├── rvmyth.v       # CPU core
│   │   ├── avsdpll.v      # PLL
│   │   ├── avsddac.v      # DAC
│   │   └── testbench.v    # Testbench
└── output/             # Simulation results
```

---

## 🛠️ Environment Setup & TLV → Verilog Conversion

### Step 1: Install Python dependencies

```bash
sudo apt update
sudo apt install python3-venv python3-pip
```

### Step 2: Create and activate a virtual environment

```bash
python3 -m venv sp_env
source sp_env/bin/activate
```

### Step 3: Install SandPiper-SaaS

```bash
pip install pyyaml click sandpiper-saas
```

### Step 4: Convert TLV to Verilog

```bash
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```

✅ **Output:** `rvmyth.v` generated successfully in `src/module/`.

---

## 🧪 Simulation Flow

### ▶️ Pre-Synthesis Simulation

```bash
mkdir -p output/pre_synth_sim

iverilog -o output/pre_synth_sim/pre_synth_sim.out \
  -DPRE_SYNTH_SIM \
  -I src/include -I src/module \
  src/module/testbench.v

cd output/pre_synth_sim
./pre_synth_sim.out
```

### ▶️ View Waveforms in GTKWave

```bash
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```

---

## 🔍 Key Signals to Observe

| Signal              | Description                                              |
| ------------------- | -------------------------------------------------------- |
| ⏱️ `CLK`            | Input clock signal (from PLL)                            |
| 🔄 `reset`          | Reset input                                              |
| 🎚 `OUT`            | DAC analog output (digital representation in simulation) |
| 🔢 `RV_TO_DAC[9:0]` | 10-bit CPU output driving DAC input                      |

---

## 🧠 Instruction Program Executed by BabySoC

| #  | Instruction         | Description                 |
| -- | ------------------- | --------------------------- |
| 0  | `ADDI r9, r0, 1`    | Set decrement step (r9 = 1) |
| 1  | `ADDI r10, r0, 43`  | Set loop limit              |
| 2  | `ADDI r11, r0, 0`   | Initialize counter          |
| 3  | `ADDI r17, r0, 0`   | Clear DAC input             |
| 4  | `ADD r17, r17, r11` | Accumulate into r17         |
| 5  | `ADDI r11, r11, 1`  | Increment counter           |
| 6  | `BNE r11, r10, -4`  | Loop until r11 = 43         |
| 7  | `ADD r17, r17, r11` | Accumulate again            |
| 8  | `SUB r17, r17, r11` | Adjust r17                  |
| 9  | `SUB r11, r11, r9`  | Decrement counter           |
| 10 | `BNE r11, r9, -4`   | Repeat loop until r11 = 1   |
| 11 | `SUB r17, r17, r11` | Final adjust                |
| 12 | `BEQ r0, r0, ...`   | Infinite loop (halt)        |

---

## 🔄 Execution Timeline

| Phase                | Register Behavior | r17 Value      | Description       |
| -------------------- | ----------------- | -------------- | ----------------- |
| Ramp (Loop 1)        | r11 = 0→42        | Σ(0..42) = 903 | Gradual increase  |
| Peak                 | r11 = 43          | 946            | Transient maximum |
| Oscillation (Loop 2) | r11 = 43→1        | 903 ± r11      | Oscillatory decay |
| Final                | r11 = 1           | Adjusted       | Stable state      |

**Data Flow:**
`Instruction Memory → CPU Pipeline → Register r17 → DAC → Analog Output`

---

## ⚖️ DAC Conversion Formula

[
V_{OUT} = \frac{r17}{1023} \times V_{REFSPAN} \quad \text{where } V_{REFSPAN} = 1.0V
]

| r17        | DAC Output Voltage |
| ---------- | ------------------ |
| 903        | 0.882 V            |
| 946 (Peak) | 0.925 V            |

💡 *Tip:* In GTKWave, switch the `OUT` signal to **Analog Step** view to visualize the DAC waveform.

---

## 📈 Waveforms & Observations

### 🖼️ <img width="1270" height="713" alt="image" src="https://github.com/user-attachments/assets/7a651cdf-1066-4e49-ac71-c16c518b1c61" />


* The `OUT` output from DAC (declared as `real reg`) provides an analog voltage ≈ **0.4848 V**.
* The SoC-level `OUT` (declared as `wire`) gives a **digital rounded value (0)**.

### 🖼️ <img width="1251" height="674" alt="image" src="https://github.com/user-attachments/assets/0a69a921-7757-49d8-835d-8d8492efa107" />


* The DAC’s `OUT` (real reg) again provides a correct analog voltage ≈ **0.924 V**.
* The `OUT` from the RVMYTH core (declared as register) shows the **digital equivalent (0)**.

---

## 🔍 Summary

* The **DAC** correctly converts digital values from `r17` into proportional analog voltages.
* The **SoC/Core output signals** depend on the data type declaration — `wire` or `reg` — displaying either digital or analog representations.

---

## ⚙️ Troubleshooting Tips

* ⚠️ Avoid module redefinition — ensure each TLV/Verilog file is included only once.
* Verify all **signal names** and **file paths** before compilation.

---

📘 *End of Part 2 — Simulation & Verification of VSDBabySoC*

---




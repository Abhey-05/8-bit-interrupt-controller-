# 🛠️ 8-bit Interrupt Controller (Verilog HDL)

This project implements a simple **8-input Interrupt Controller** using Verilog HDL.

It manages 8 interrupt lines and allows the processor to handle them either using:
- ✅ **Polling Mode** (check each one in order)
- 🎯 **Custom Priority Mode** (check in custom order set by processor)

Useful for small CPUs, embedded systems, or custom SoC designs.

---

## 🔧 How It Works

### 🌀 After Reset:
- The controller **waits for a command** from the processor.
- The command is sent over a shared 8-bit data bus.
- The **lowest 2 bits** in the data bus determine the **mode**:

| Command  | Mode           |
|----------|----------------|
| `xx_xx01`| Polling Mode   |
| `xx_xx10`| Priority Mode  |

---

## 🔁 Polling Mode

In this mode:
- Controller checks all 8 interrupt lines **in a loop**.
- If any interrupt is active:
  - It sets `interrupt_out = 1`.
  - Waits for `acknowledgement` signal from processor.
  - Sends interrupt info: `01011_ID` on `data_bus`, where `ID` is interrupt number.
  - Waits for final confirmation: `10100_ID` from processor.
  - If confirmation matches, it moves to next interrupt.
  - If not, it **resets**.

### 🔄 Steps:
1. Check interrupt 0 to 7 in order.
2. If active, raise interrupt and wait.
3. Processor acknowledges, gets ID.
4. Processor confirms service done.

---

## 🥇 Custom Priority Mode

Similar to Polling, but order is **set by processor** in beginning.

### 📥 Initialization:
- Processor sends 4 cycles of 8-bit data.
- Each contains **two 3-bit IDs** → total 8 priorities.

For example:
data_bus = xxxyyy10
Means:
- `xxx` = highest priority ID
- `yyy` = second-highest priority ID

After receiving all 4 cycles:
- Controller polls using this custom order.
- Remaining process (acknowledgement, condition codes) is **same as Polling**.

---

## 🔢 Condition Codes

| Mode       | Sender       | Code   | Meaning                            |
|------------|--------------|--------|------------------------------------|
| Polling    | Controller   | 01011  | Interrupt info sent to processor  |
| Polling    | Processor    | 10100  | Confirmation of service complete  |
| Priority   | Controller   | 10011  | Interrupt info (based on priority)|
| Priority   | Processor    | 01100  | Service confirmation               |

These are **5-bit prefixes** attached to the 3-bit interrupt ID.

---


## 🔍 Key Signals

| Signal           | Type    | Description                                 |
|------------------|---------|---------------------------------------------|
| `clock`          | Input   | System clock                                |
| `reset`          | Input   | Resets the controller                       |
| `mask`           | Input   | Enable/disable masking                      |
| `intr_rq_1`      | Input   | 8-bit input (interrupt requests)            |
| `data_bus`       | InOut   | Shared data bus with processor              |
| `acknowledgement`| Input   | Handshake signal from processor             |
| `interrupt_out`  | Output  | Goes high when interrupt is active          |
| `enable`         | Output  | Enables writing to data bus                 |

---


## ✅ TODO

- [ ] Check for duplicate entries in priority table.
- [ ] Allow some interrupt lines to be disabled.
- [ ] Add global masking register per interrupt.

---


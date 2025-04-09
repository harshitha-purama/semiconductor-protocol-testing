# JTAG / Boundary Scan (IEEE 1149.1)

## 1. Purpose & Common Uses

* **JTAG Name:** Joint Test Action Group (the committee that created the standard).
* **Standard Name:** IEEE Standard 1149.1 - Standard Test Access Port and Boundary-Scan Architecture.
* **Core Problem Solved:** Provides a standardized digital interface to access and control the pins of compliant chips on a Printed Circuit Board (PCB) without requiring direct physical probing, which is difficult with modern dense packages (like BGAs).
* **Key Uses:**
    * **Board-Level Interconnect Testing (Boundary Scan):** Detecting manufacturing defects like shorts and open circuits between chip pins on a PCB.
    * **In-System Programming (ISP):** Programming programmable devices (FPGAs, CPLDs) and flash memories while they are installed on the board.
    * **Processor Debugging:** Providing low-level hardware access for controlling CPU execution (run, halt, step), inspecting registers, and accessing memory.

## 2. The JTAG Interface (TAP - Test Access Port)

The standard defines a set of pins for the Test Access Port:

* **`TCK` (Test Clock):** Input clock provided by the external JTAG controller/debugger. All JTAG operations are synchronous to this clock.
* **`TMS` (Test Mode Select):** Input signal used to control the internal state machine (TAP Controller). The sequence applied to TMS determines the JTAG operation.
* **`TDI` (Test Data In):** Serial data input *to* the chip's JTAG logic (used to load instructions or test data).
* **`TDO` (Test Data Out):** Serial data output *from* the chip's JTAG logic (used to read out instructions, test results, or device IDs).
* **`TRST` (Test Reset - Optional):** Optional asynchronous reset pin for the JTAG logic.

* **JTAG Chain:** Multiple JTAG devices on a board are often daisy-chained: `TDO` of one chip connects to `TDI` of the next. `TCK`, `TMS`, and `TRST` (if used) are typically connected in parallel.

## 3. Key Concepts

* **TAP Controller:** An internal 16-state Finite State Machine (FSM) within each JTAG-compliant chip. It interprets the `TMS` signal on `TCK` rising edges to navigate through states that manage instruction loading, data capture, data shifting, and updating outputs.
* **Instruction Register (IR):** A shift register holding the currently active JTAG instruction code. The instruction tells the JTAG logic what operation to perform (e.g., which Data Register to access).
* **Data Registers (DR):** Various shift registers used for data operations, selected by the current instruction in the IR. Key DRs include:
    * **Boundary Scan Register (BSR):** A core component for testing. It consists of a chain of individual memory cells ('boundary scan cells') connected logically between the chip's internal core logic and its physical pins. Allows capturing pin states or overriding pin outputs.
    * **BYPASS Register:** A mandatory single-bit register providing a minimal path from `TDI` to `TDO`, used to shorten the scan chain when accessing other devices.
    * **IDCODE Register:** A register holding a standard 32-bit device identification code (manufacturer, part number, version).

## 4. Common JTAG Instructions (Mandatory by IEEE 1149.1)

* **`EXTEST` (External Test):** Selects the BSR. Used to test board interconnects by allowing control of output pin states and observation of input pin states via the BSR. Isolates chip core logic from pins.
* **`SAMPLE/PRELOAD`:** Selects the BSR. `SAMPLE` captures pin values into the BSR without affecting chip operation. `PRELOAD` loads data into the BSR output cells, often before an `EXTEST`. Does not isolate core logic.
* **`BYPASS`:** Selects the BYPASS register, shortening the scan path through the chip.

*(Other common instructions include `IDCODE` to read the device ID, `INTEST` for internal testing, `RUNBIST` to trigger self-tests.)*

## 5. How Boundary Scan Works (Interconnect Test Example)

1.  **Chain Setup & Instruction Load:** Connect JTAG controller. Shift the `EXTEST` instruction into the IR of all relevant chips.
2.  **Shift Test Pattern:** Shift a test pattern through the BSR chain via `TDI` to set desired output pin values.
3.  **Apply & Capture:** Update outputs to drive the pattern onto PCB traces. After signals settle, capture the values seen at input pins into the BSRs.
4.  **Shift Out Results:** Shift the captured BSR data out through the chain via `TDO`.
5.  **Compare & Analyze:** Compare the shifted-out results with expected values. Mismatches indicate connectivity faults (shorts, opens).

## 6. BSDL (Boundary Scan Description Language)

* A standard file format from the chip vendor describing the chip's JTAG implementation (BSR structure, instructions, IDCODE, etc.).
* Essential input for JTAG test generation and execution software.

## 7. Pros and Cons

* **Pros:**
    * Standardized (IEEE 1149.1).
    * Reduces need for physical test probes, enabling testing of complex boards/packages (BGAs).
    * Combines testing, programming, and debugging functions.
    * Helps isolate faults.
* **Cons:**
    * Requires JTAG-compliant ICs (adds cost/complexity).
    * Adds pins (TAP interface).
    * Test speed limited by TCK frequency.
    * Requires specialized software and BSDL files.
    * Primarily tests static connectivity, not timing or analog aspects.

## 8. Relevance to Testing & Debug

* **Manufacturing Test:** Core technology for detecting PCB assembly defects.
* **Board Bring-up & Debug:** Verifying connections, diagnosing faults.
* **In-System Programming:** Loading firmware/bitstreams into FPGAs, CPLDs, MCUs, Flash.
* **CPU Debug:** Low-level hardware debugging access.


## 9. Deeper Dive: The TAP Controller State Machine

The heart of JTAG is the 16-state **TAP (Test Access Port) Controller** Finite State Machine (FSM). Its transitions are controlled solely by the value of `TMS` sampled on the rising edge of `TCK`. Understanding its flow is key to understanding JTAG operations.

Here's a conceptual breakdown:

* **Reset Path:**
    * `Test-Logic-Reset`: The starting state (and reset state). Holding `TMS` HIGH for 5+ TCK cycles forces the controller here.
* **Idle Path:**
    * `Run-Test/Idle`: A stable state where the chip's normal logic can operate (or just idle). JTAG logic is inactive. From here, `TMS=1` starts a sequence to access registers.
* **Data Register (DR) Scan Path:** (Used for BSR, IDCODE, BYPASS etc.)
    * `Select-DR-Scan`: Decision point (`TMS=0` -> Capture-DR, `TMS=1` -> Select-IR-Scan).
    * `Capture-DR`: Parallel loads data into the selected DR (e.g., captures pin states into BSR if SAMPLE/EXTEST is active). Moves to Shift-DR or Exit1-DR.
    * `Shift-DR`: Shifts data serially one bit per TCK cycle from `TDI` through the selected DR to `TDO`. `TMS=0` keeps it here; `TMS=1` moves to Exit1-DR. **This is where data is actually shifted in/out.**
    * `Pause-DR`: Can temporarily pause shifting (`TMS=0` stays here, `TMS=1` moves to Exit2-DR).
    * `Exit1-DR` & `Exit2-DR`: Intermediate states for navigating out of the shift/pause loop.
    * `Update-DR`: Latches the data shifted into the DR onto parallel outputs (e.g., makes the BSR drive the pins if EXTEST is active). Moves back to Run-Test/Idle or Select-DR-Scan.
* **Instruction Register (IR) Scan Path:** (Used to load JTAG instructions)
    * `Select-IR-Scan`: Decision point (`TMS=0` -> Capture-IR, `TMS=1` -> Test-Logic-Reset). Reached from Select-DR-Scan via `TMS=1`.
    * `Capture-IR`: Parallel loads a fixed pattern (usually `...001`) into the IR shift register. Moves to Shift-IR or Exit1-IR.
    * `Shift-IR`: Shifts instruction data serially from `TDI` through the IR to `TDO`. `TMS=0` keeps it here; `TMS=1` moves to Exit1-IR. **This is where new instructions are loaded.**
    * `Pause-IR`: Can temporarily pause shifting.
    * `Exit1-IR` & `Exit2-IR`: Intermediate states.
    * `Update-IR`: Latches the new instruction from the IR shift register into the instruction holding register, making it the active instruction. Selects the corresponding DR. Moves back to Run-Test/Idle or Select-DR-Scan.

*Navigation Strategy:* The external JTAG controller drives `TMS` and `TCK` precisely to navigate this state machine to perform the desired operations (select IR/DR, shift data, update outputs).

## 10. More Common JTAG Instructions

Besides the mandatory `EXTEST`, `SAMPLE/PRELOAD`, and `BYPASS`, other frequently encountered (often optional but standard) instructions include:

* **`IDCODE`:** Selects the device Identification Register (IDCODE Register) as the DR. When data is shifted out (`Shift-DR` state), the chip's 32-bit unique ID (Manufacturer ID, Part Number, Version) is output on `TDO`. Allows tools to automatically identify devices in the chain.
* **`USERCODE`:** Similar to `IDCODE`, but selects a user-defined code register. Often used to read back programming status or other custom information.
* **`INTEST` (Internal Test):** Selects the BSR. Allows driving test data *from* the BSR *into* the chip's core logic and capturing the core logic's response back *into* the BSR. Used for testing the chip's internal functionality via the boundary scan cells.
* **`RUNBIST` (Run Built-In Self-Test):** Selects a DR (often just BYPASS). This instruction triggers the execution of the chip's internal self-test sequences (like Memory BIST). The TAP controller usually goes to `Run-Test/Idle` while BIST runs. Results are typically read out later via another JTAG access or status pins.
* **`CLAMP`:** Selects the BYPASS register. Drives the values previously loaded into the BSR output cells onto the chip's pins (similar to `EXTEST`) but disconnects the BSR from `TDI`/`TDO`. Useful for holding the outputs of one chip stable (quieting the bus) while testing another chip in the chain.

## 11. BSDL (Boundary Scan Description Language) File Details

BSDL is crucial for any tool that needs to control a device via JTAG. Key information found in a BSDL file (`.bsd` or `.bsdl` extension):

* **Entity Declaration:** Names the device the BSDL file describes.
* **Generic Parameter:** Often specifies the package type.
* **Port Description:** Lists all the physical pins of the device.
* **Pin Mapping:** Connects the physical pins to their logical signals and port types (input, output, bidirectional, clock, JTAG TAP pins etc.).
* **Scan Port Identification:** Identifies which pins are `TDI`, `TDO`, `TMS`, `TCK`, `TRST`.
* **Instruction Register Description:** Defines the JTAG instructions supported by the chip, their binary opcodes, and their lengths.
* **Register Access Description:** Specifies which Data Register (e.g., `BOUNDARY`, `BYPASS`, `DEVICE_ID`) is connected between `TDI` and `TDO` for each instruction.
* **Boundary Register Description:** This is often the largest part. It lists every cell in the Boundary Scan Register (BSR) chain, specifies the type of cell (e.g., input buffer observation, output buffer control), which pin(s) it relates to, and any control cells that enable/disable output buffers.
* **Device IDCODE:** Specifies the expected 32-bit value for the IDCODE instruction.

*Tools use this information to know exactly how long the scan chains are, which bits correspond to which pins, and what instruction codes to shift in.*

## 12. Basic JTAG Chain Debugging Tips

If a JTAG chain isn't working (e.g., tools report errors, can't find devices):

* **Check Physical Connections:** Ensure the JTAG adapter is correctly wired to the board's TAP signals (`TCK`, `TMS`, `TDI`, `TDO`, GND). Check for shorts or opens.
* **Check Power:** Ensure the target board and JTAG adapter have the correct power and voltage levels referenced.
* **Verify TCK Signal:** Use an oscilloscope to check if `TCK` is present and clean.
* **Check Reset:** Ensure `TRST` (if used) is in the correct state, or that the TAP controller isn't stuck in reset.
* **Read IDCODEs:** Attempting to read IDCODEs is often the first step. If it fails, it might indicate:
    * A break in the chain (TDO of one chip not reaching TDI of the next).
    * A faulty device holding `TDO` low or high.
    * Incorrect BSDL files being used by the software (leading it to expect the wrong chain length or ID).
    * Incorrect configuration in the JTAG tool software.
* **Isolate the Problem:** If possible, try accessing shorter segments of the chain or individual devices (if the board design allows).


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

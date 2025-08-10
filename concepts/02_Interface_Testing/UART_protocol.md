# UART (Universal Asynchronous Receiver/Transmitter) Protocol

## 1. Purpose & Common Uses

* **Goal:** Enables simple, bi-directional, serial communication between devices—especially microcontrollers, computers, and peripherals—over short to medium distances using asynchronous signaling.
* **Invented By:** The UART concept dates to early serial communication in teletypes and modems, standardized in electronics by manufacturers like National Semiconductor.
* **Common Uses:**
    * Communication between microcontrollers and PCs (debugging, programming).
    * Connectivity with GPS modules, Bluetooth/Wi-Fi modules, and GSM modems.
    * Serial consoles for embedded Linux systems.
    * Configuring/configuring sensors or user interfaces in embedded systems.
    * Industrial machine interfaces (legacy/fieldbus connectivity).

## 2. Key Characteristics

* **Asynchronous:** No clock line; devices agree on data rate (baud rate) in advance and exchange data timed by internal clocks.
* **Serial:** Data is transmitted one bit at a time (over a single wire in each direction).
* **Full-Duplex:** TX (Transmit) and RX (Receive) lines allow simultaneous two-way communication.
* **Point-to-Point:** Classic UART is intended for direct device-to-device links (not shared bus), but RS-485 variants allow multidrop.
* **No Device Addressing:** Line-level signals connect a specific transmitter to a specific receiver (unlike I2C/SPI which are multi-device buses).

## 3. The Signal Lines

* **TX (Transmit):**
    * The wire on which a device sends data to another device’s RX line.
* **RX (Receive):**
    * The wire that receives data from the counterpart’s TX line.
* **Ground:** Must be shared between devices.
* **Optional Control Lines:**
   * **RTS (Request To Send), CTS (Clear To Send):** Used for hardware flow control (optional, more common in RS-232 serial ports).
   * **DSR, DTR, DCD, RI:** Other signals for legacy modem/telecom usage (rare in embedded systems).
* **Voltage Levels:**
    * **TTL/CMOS UART:** Logic LOW usually 0V, HIGH 3.3V or 5V.
    * **RS-232 level:** Uses ±12V or ±5V, inverted logic; requires level conversion to connect to MCU logic.

## 4. Fundamental Rules & Bus States

* **IDLE State:** TX line sits HIGH (logic '1').
* **Data Framing:** Each data transfer ("frame") consists of:
    - **Start Bit:** Single LOW (0) bit signaling start of frame.
    - **Data Bits:** Usually 7, 8, or 9 bits, LSB first.
    - **(Optional) Parity Bit:** For basic error detection.
    - **Stop Bit(s):** One or more HIGH (1) bits marking end of frame.
* **Baud Rate:** Sender and receiver must agree (e.g., 9600, 115200bps). Mismatched rates lead to data errors.
* **No Clock Line:** Timing is self-contained based on baud rate.

## 5. Basic Transaction Flow

### Data Transmission

1. **Idle:** TX/RX lines HIGH.
2. **Transmit:** Sender pulls TX LOW (start bit).
3. **Data:** Sender sends data bits (LSB first), optionally appends parity bit.
4. **Stop:** Sender sends stop bit(s) (TX HIGH).
5. **Idle:** Line returns to HIGH until next start bit.
6. **Receiver:** Detects start bit, samples subsequent bits based on expected baud rate, reconstructs byte.

### Optional Flow Control

- **Hardware (RTS/CTS):** Used to signal readiness to send/receive (common in high-speed/PC UARTs).
- **Software (XON/XOFF):** Special in-band characters to manage flow (less common in microcontrollers).

## 6. Key Concepts Recap

* **Asynchronous Framing:** Each byte is sent independently; the receiver must synchronize to each frame’s start bit.
* **No Bus Arbitration:** Only one device drives each TX line; electrical contention not possible in point-to-point links.
* **Simple Protocol:** No device addressing or transaction structure—each byte sent is delivered directly.
* **No Explicit Error Correction:** Parity can detect (not correct) single-bit errors, but most implementations ignore parity.

## 7. Pros and Cons

* **Pros:**
    * Extremely simple protocol and hardware (breadboard-friendly).
    * Requires minimal wiring (2 data lines plus ground).
    * Supported in most microcontrollers and PCs.
    * Easy for debugging, bootloaders, and development.
    * Widely understood and supported by tools (serial terminals, analyzers).

* **Cons:**
    * No native support for multiple devices on one link.
    * **No clock line:** Requires tight baud rate tolerance; mismatch causes errors.
    * Lower speeds compared to modern serial buses.
    * No built-in addressing, error checking, or correction (beyond parity).
    * Longer cables require careful signal integrity planning (especially at higher speeds).
    * Not hot-swappable—glitches if connections made/disconnected while powered.

## 8. Relevance to Testing & Debug

Understanding UART is critical for:

* **Firmware Debugging:** Easy real-time monitoring and logging from embedded devices via serial port.
* **Board Bring-Up:** Early hardware diagnostics often use UART for basic communication when more complex peripherals are not yet functional.
* **Programming Microcontrollers:** Many devices accept firmware via UART bootloader.
* **Protocol Testing:** Verifying baud rate, frame format, data integrity, and flow control.
* **Interfacing Lab/Test Equipment:** Many analyzers and test jigs communicate or log data using UART.
* **Production Testing:** Basic functional test result streaming, PCBA programming, or configuration via serial port.

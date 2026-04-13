# Reflow Oven Controller

![Setup](setup.jpg)

## Overview

An 8051-assembly reflow oven controller that drives a 1500 W toaster oven through a solid-state relay to execute a full reflow soldering profile: ramp to soak, soak, ramp to reflow, reflow, and cool. Built on the CV-8052 soft processor running on a DE10-Lite FPGA board, with a K-type thermocouple and cold-junction compensation for temperature sensing.

The system runs a Moore finite-state machine, sends live temperature data over UART for a real-time strip chart, and provides user feedback through an LCD, 7-segment displays, and a speaker. Safety abort conditions (no temperature rise in the first 60 seconds, or temperature exceeding 240 °C) shut down the oven automatically.

The controller was validated against a Fluke 45 multimeter across 25–240 °C and held within the required ±3 °C tolerance over the full range. It was then used to reflow-solder three EFM8 PCBs for subsequent labs, all of which powered up correctly.

## Hardware

- **Processor:** CV-8052 (8051-compatible soft core) on Intel DE10-Lite FPGA
- **Temperature sensing:** K-type thermocouple (~41 µV/°C) through a difference amplifier with gain ≈ 300
- **Cold-junction compensation:** LM335 precision temperature sensor
- **Oven control:** Solid-state relay (SSR) driven by PWM from the CV-8052
- **User interface:** 16×2 LCD, DE10-Lite 7-segment displays, DIP switches for parameter selection, keypad and pushbuttons for start/stop
- **Audio feedback:** Piezo speaker

## How It Works

### Temperature sensing

The thermocouple outputs roughly 10 mV at the top of the operating range. A difference amplifier with a gain of ~300 maps that to 0–3 V so the CV-8052's ADC can use its full dynamic range. A difference topology was picked over a single-ended amplifier specifically because the SSR switching a 1500 W load right next to the sensing electronics generates a lot of common-mode noise, and the extra rejection kept the readings stable. The LM335 reads room temperature at startup and is added in software to give the true oven temperature.

### State machine

The control logic is a Moore FSM with eight states: `IDLE`, `PREHEAT`, `SOAK`, `RAMP`, `REFLOW`, `COOLING`, `DONE`, and `ERROR`. Transitions are driven by temperature thresholds, elapsed-time counters, user input, or safety violations. Each state has a fixed output (SSR on/off, PWM duty cycle, beeper behaviour), which makes the logic easy to reason about and kept the safety integration simple — the abort conditions are just additional transition edges into `ERROR`.

### PWM oven control

The SSR is driven by PWM rather than bang-bang on/off control. Ramp stages run at 100% duty cycle to heat quickly; soak and reflow stages drop the duty cycle to hold the setpoint without overshooting; cooling and error states go to 0%. The PWM is generated from a timer interrupt so its timing stays consistent regardless of what the main loop is doing.

### Serial strip chart

A separate timer interrupt fires once per second, triggering an ADC read, temperature conversion, LCD update, and UART transmission of the current temperature. A Python script on the host reads the serial port and plots the temperature in real time.

### Safety

Two abort conditions are wired directly into the FSM:
- **No rise:** if the oven doesn't reach 50 °C within 60 seconds of starting, the controller assumes the thermocouple isn't actually in the oven and aborts.
- **Over-temperature:** if the reading exceeds 240 °C at any point, the controller aborts.

Both transitions go to `ERROR`, which disables the SSR, holds the output at 0 %, and beeps ten times.

## Validation

Temperature accuracy was validated against a Fluke 45 digital multimeter with the thermocouple placed in the oven and readings recorded at ~10 °C intervals from 25 °C to 250 °C. Absolute error stayed within ±3 °C across the full operating range, meeting the project specification. Remaining error was attributable to ADC quantization, thermocouple noise, thermal lag between the multimeter probe and the thermocouple junction, and small timing differences between when each reading was recorded.

Ramp rate was measured from the serial strip chart and fell within the required 1–2 °C/s during heating stages. Safety abort conditions were tested intentionally (leaving the thermocouple outside the oven, and forcing an over-temperature condition); the controller responded correctly in every case.

Three EFM8 PCBs were assembled using the controller and all powered up correctly on first boot.

## Challenges

**Serial communication.** PuTTY wouldn't connect to the CV-8052 for three days. Ended up being two independent problems stacked on top of each other — a Bluetooth virtual COM port was grabbing the wrong device, and a damaged IC on one group member's board was preventing the UART from transmitting at all. Swapping hardware eventually isolated both issues.

**Op-amp saturation.** The temperature reading suddenly jumped to 370 °C partway through testing. Traced it back to a loose resistor in the difference amplifier feedback path — the gain had effectively gone to infinity, saturating the output. Re-seating the resistor fixed it, but the bigger lesson was to be more suspicious of breadboard wiring when readings go nonsensical.

**Debug discipline.** Iterative, piece-by-piece testing (verifying thermocouple readings with a multimeter before hooking up the ADC, testing UART before sending live data, validating each FSM state independently) saved time overall even though it felt slower in the moment. Most of the debugging time later in the project came from issues in sections we hadn't tested carefully enough early on.

## Future Improvements

- **Closed-loop PID control** instead of fixed-duty-cycle PWM would hold the soak and reflow temperatures tighter. Writing a full PID in 8051 assembly wasn't realistic in the time we had, but it's the natural next step.

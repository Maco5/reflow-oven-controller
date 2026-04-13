# Reflow Oven Controller

UBC ELEC 291 Project 1 — a reflow oven controller built on the CV-8052 (8051-compatible) soft processor running on a DE10-Lite FPGA. The system drives a 1500 W toaster oven through a solid-state relay to execute a full reflow soldering profile (ramp to soak, soak, ramp to reflow, reflow, cool), with K-type thermocouple sensing, cold-junction compensation, LCD and 7-segment display output, a live serial strip chart, and automatic safety abort conditions.

All firmware is written in 8051 assembly. The controller was validated against a Fluke 45 multimeter across 25–240 °C within the required ±3 °C tolerance, and was used to reflow-solder three EFM8 PCBs.

## Demo

📺 **[Video demonstration](https://www.youtube.com/watch?v=HULL_BBDajs)**

## Team

Group project completed with **James Kerr, Retter Ma, Helia Ahmadzadeh, and Chloe Lam**.

## Acknowledgements

Completed as Project 1 for **ELEC 291/292** at the University of British Columbia, taught by **Dr. Jesús Calviño-Fraga**. The project specification, CV-8052 soft processor, and supporting course materials are his work (© 2007–2026) and all rights remain with Dr. Calviño-Fraga and UBC.

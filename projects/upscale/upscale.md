---
title: Upscaling Scales.
---

In this project we write some python code for a standard 4 wire scale.

Below is a typical wiring scheme to get a 4‑wire load cell working with an HX711 board on a Raspberry Pi. The key points are:

Hook the load cell’s four leads to the HX711 amplifier (E+, E‑, A+, A‑).
Connect the HX711 amplifier’s digital pins (GND, DT, SCK, VCC) to the Raspberry Pi.
Ensure the HX711 data lines are at 3.3 V logic when talking to the Pi (either by powering from 3.3 V or using level shifting).

1. Identify the load cell wires
Most 4‑wire load cells use the following color convention (confirm yours, as it can vary):

Red = Excitation+ (often labeled E+)
Black = Excitation− (often labeled E−)
Green = Signal+ (often labeled S+ or A+)
White = Signal− (often labeled S− or A−)
(Some load cells reverse green/white for S+ and S–, but the red and black for E+ and E– are usually consistent.)

Why 3.3 V?
The Pi’s GPIO pins are 3.3 V logic.
If the HX711 module you have does not include an onboard regulator/level shifter and you feed it 5 V on VCC, then its data/clock lines could output 5 V back to the Pi’s GPIO—which can damage the Pi.
Many HX711 breakout boards do run fine at 3.3 V and will thus keep the DT and SCK lines safe at 3.3 V.
If your HX711 board truly requires 5 V (check documentation or measure it), then use your 5 → 3.3 V level shifters on DT and SCK to ensure the Raspberry Pi does not see 5 V logic.


              WIRING SUMMARY (LOAD CELL + HX711 + RASPBERRY PI)
┌───────────────────────────────────────────────────────────────────────────────┐
│ LOAD CELL (4 wires) → HX711 AMPLIFIER                                       │
├───────────────────────────────────────────────────────────────────────────────┤
│  Load Cell Red   →  HX711 E+                                                │
│  Load Cell Black →  HX711 E–                                                │
│  Load Cell White →  HX711 A–                                                │
│  Load Cell Green →  HX711 A+                                                │
└───────────────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────────────┐
│ HX711 AMPLIFIER → RASPBERRY PI (B+ model)                                    │
├───────────────────────────────────────────────────────────────────────────────┤
│  HX711 GND  →  Pi GND        (Black wire)                                    │
│  HX711 DT   →  Pi GPIO17     (Pin 11)  (White wire)                          │
│  HX711 SCK  →  Pi GPIO27     (Pin 13)  (Orange wire)                         │
│  HX711 VCC  →  Pi 3.3 V or 5 V (commonly 3.3 V)  (Red wire)                   │
└───────────────────────────────────────────────────────────────────────────────┘

NOTES:
• You specified the Pi side wire colors as:
    - Red for VCC
    - Black for GND
    - White for DT
    - Orange for SCK
• If using 5 V, ensure the HX711 module output signals are level-shifted down to
  3.3 V for the Pi’s GPIO. Many HX711 boards run fine at 3.3 V.
• After wiring, run your Python code with the HX711 library. Calibrate as needed.


# 1. Create a virtual environment named .venv (or pick your own name)
python3 -m venv .venv

# 2. Activate the virtual environment
source .venv/bin/activate

# (Optional) Upgrade pip within the venv
pip install --upgrade pip

sudo apt-get remove python3-rpi.gpio

pip install RPi.GPIO

# 1. Clone the repo
git clone https://github.com/tatobari/hx711py.git

# 2. Enter the directory
cd hx711py

# 3. Install into your virtual environment
#    This compiles & installs the library so you can "import hx711"
pip install .
or
python setup.py install
cd ..

python scale.py

## Patch for old Pi's.
/usr/local/lib/python3.??/dist-packages/RPi/GPIO/__init__.py
or
/usr/lib/python3/dist-packages/RPi/GPIO/__init__.py


5. Patch the Old-Style Revision Code Check (If Needed)
If you’re encountering the NotImplementedError: This module does not understand old-style revision codes with your Pi B+ (revision 0x10), locate the function that raises this error (often in something named _get_rpi_info() within certain forks of RPi.GPIO or a custom library file). You’ll see a check like:

if not (revision >> 23 & 0x1):
    raise NotImplementedError(
        'This module does not understand old-style revision codes'
    )

Example Patch
Remove or modify that check, and add a block to handle the old-style revision. For example:

# Check if bit 23 is set: indicates new-style code
is_new_style = bool(revision >> 23 & 0x1)

if not is_new_style:
    # old-style revision code path
    # e.g. Pi B+ with revision code 0x10
    return {
        'P1_REVISION': 3,  # Pi B+ typically "3" for 40-pin layout
        'REVISION': hex(revision)[2:],
        'TYPE': 'Model B+ (Old-Style Rev)',
        'MANUFACTURER': 'Sony UK',     # or "Unknown" if unsure
        'PROCESSOR': 'BCM2835',
        'RAM': '512M',
    }
else:
    # Proceed with the new-style revision logic
    ...


(1)   3.3V Power (3V3)        -> . . <-  5V Power          (2)
(3)   GPIO2  (SDA1)           -> . . <-  5V Power          (4)
(5)   GPIO3  (SCL1)           -> . . <-  GND               (6)
(7)   GPIO4                   -> . . <-  GPIO14 (TXD0)     (8)
(9)   GND                     -> . . <-  GPIO15 (RXD0)     (10)
(11)  GPIO17                  -> . . <-  GPIO18            (12)
(13)  GPIO27                  -> . . <-  GND               (14)
(15)  GPIO22                  -> . . <-  GPIO23            (16)
(17)  3.3V Power (3V3)        -> . . <-  GPIO24            (18)
(19)  GPIO10 (SPI0 MOSI)      -> . . <-  GND               (20)
(21)  GPIO9  (SPI0 MISO)      -> . . <-  GPIO25            (22)
(23)  GPIO11 (SPI0 SCLK)      -> . . <-  GPIO8  (SPI0 CE0) (24)
(25)  GND                     -> . . <-  GPIO7  (SPI0 CE1) (26)
(27)  ID_SD (Reserved EEPROM) -> . . <-  ID_SC (Reserved)  (28)
(29)  GPIO5                   -> . . <-  GND               (30)
(31)  GPIO6                   -> . . <-  GPIO12            (32)
(33)  GPIO13                  -> . . <-  GND               (34)
(35)  GPIO19                  -> . . <-  GPIO16            (36)
(37)  GPIO26                  -> . . <-  GPIO20            (38)
(39)  GND                     -> . . <-  GPIO21            (40)
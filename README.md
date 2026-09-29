![picokit-14-dht11-leds](https://raw.githubusercontent.com/mytechnotalent/picokit-14-dht11-leds/main/picokit-14-dht11-leds.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-14 DHT11 LEDS

### DHT11 Comfort LEDs and Authenticated Heartbeat
#### Lesson 14 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The fourteenth Picokit lesson. The node reads a DHT11 temperature and
humidity sensor on GP4 every two seconds and drives the red, yellow, and
green comfort LEDs from the temperature band, and every five seconds it
transmits an authenticated heartbeat over LoRa to a Python gateway that
logs and displays it. It reuses the standard node shape and adds a small
comfort annunciator on top of the sensor.

<br>

## What it teaches

- Mapping a temperature in tenths to a comfort band.
- Driving several GPIO outputs from a single annunciator state.
- Sealing a tiny JSON body with Argon2id and XChaCha20-Poly1305 and sending it
  with an AT+SEND over the RYLR998.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| DHT11 data | GP4 | temperature and humidity input |
| Red / Yellow / Green | GP16 / GP18 / GP17 | comfort LEDs |
| Onboard LED | GP25 | heartbeat, one blink per transmit |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. Every two seconds it samples the
DHT11 on GP4 and maps the temperature to a comfort state: green for a
comfortable reading, yellow when it warms past -5.0 C, and red at or above
0.0 C. Every 5 seconds it seals `{"n":14,"s":<seq>,"c":<code>}` with the
field key and sends it over LoRa. The gateway authenticates each frame and
only then parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_14_dht11_leds.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
=== PICOKIT-14 DHT11 LEDS // COMFORT LEDS + AUTHENTICATED HEARTBEAT ===
DHT t=230 h=610 code=3
DHT t=229 h=608 code=3
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=14 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-15-ir-decode](https://github.com/mytechnotalent/picokit-15-ir-decode)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-14-dht11-leds/blob/main/LICENSE)

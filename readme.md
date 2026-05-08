# WSU Indoor Air Quality (IAQ) Project

## Wireless Sensor Network for Residential IAQ Measurement

Raspberry Pi-based sensor nodes for continuous indoor CO2, temperature, and barometric pressure measurement in residential homes. Developed at Washington State University's Laboratory for Atmospheric Research (LAR) as part of the EPA STAR-funded Pacific Northwest IAQ field campaign (2015-2017). CO2 measurements served as a trace gas for calculating air change rates used across all project publications.

## Overview

- CO2 measured via SenseAir K30 (NDIR, UART); temperature and pressure via Bosch BMP180 (I2C); HTU21DF (relative humidity and temperature) deployed but not logged
- Sensor data published in real-time over MQTT and written to daily-rotating TSV log files
- Each node runs as a pair of systemd services that auto-start at boot and recover from failures automatically
- WiFi (802.11bgn) connectivity; log files accessible over the local network via Samba
- Configurable sampling interval (default: 20 seconds)
- Deployed in residential homes across the Pacific Northwest alongside VOC and formaldehyde instruments

## Hardware

| Component | Model | Measured Variables | Interface |
|-----------|-------|--------------------|-----------|
| CO2 sensor | SenseAir K30 | CO2 (ppm) | UART (9600 baud) |
| Pressure/temperature | Bosch BMP180 | Pressure (mbar), temperature (°C) | I2C |
| Humidity/temperature | Measurement Specialties HTU21DF | RH (%), temperature (°C) | I2C |
| Host | Raspberry Pi | — | 802.11bgn WiFi |

## Repository Structure

```
rpi-python-iaq-sensor/
├── scripts/
│   ├── k30-logger.py           # CO2 data logger (K30, UART)
│   └── bmp180-logger.py        # Pressure/temperature data logger (BMP180, I2C)
├── python/
│   └── daq.py                  # Original proof-of-concept script (v0.1; House 2)
├── etc/
│   ├── wsn/
│   │   ├── k30-logger.conf     # K30 logger configuration
│   │   └── bmp180-logger.conf  # BMP180 logger configuration
│   ├── samba/
│   │   └── smb.conf            # Samba share configuration
│   └── systemd/system/
│       ├── k30-logger.service
│       └── bmp180-logger.service
├── third-party/                # Third-party documentation sources and fetch script
├── resources/
│   └── Using-the-RPi.md        # Reference links for Raspberry Pi setup
├── install.sh                  # Installation script
└── changelog.md
```

## Setup

Install system dependencies and the Adafruit BMP180 library:

```bash
sudo apt-get update
sudo apt-get install git build-essential python-dev python-smbus samba samba-common
git clone https://github.com/adafruit/Adafruit_Python_BMP.git
cd Adafruit_Python_BMP
sudo python setup.py install
```

Install Python dependencies:

```bash
pip install paho-mqtt pyserial RPi.GPIO
```

Run the installation script to deploy logger executables, configuration files, and systemd services:

```bash
sudo ./install.sh
```

The services start automatically at boot. To check status or restart manually:

```bash
systemctl status k30-logger.service
systemctl status bmp180-logger.service
```

## Configuration

Configuration files are installed to `/etc/wsn/` by `install.sh`. Edit before or after installation and restart the relevant service.

**`/etc/wsn/k30-logger.conf`**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `serial_port` | `/dev/ttyAMA0` | UART device for K30 |
| `serial_baud` | `9600` | Baud rate |
| `interval` | `20` | Sampling interval (seconds) |
| `log_dir` | `/var/log/wsn/k30` | Directory for TSV log files |
| `broker_addr` | `10.1.1.4` | MQTT broker IP address |
| `broker_port` | `1883` | MQTT broker port |
| `report_topic` | `home/{hostname}/k30/state` | MQTT topic (`{hostname}` substituted at runtime) |

**`/etc/wsn/bmp180-logger.conf`**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `interval` | `20` | Sampling interval (seconds) |
| `log_dir` | `/var/log/wsn/bmp180` | Directory for TSV log files |
| `broker_addr` | `10.1.1.4` | MQTT broker IP address |
| `broker_port` | `1883` | MQTT broker port |
| `report_topic` | `home/{hostname}/bmp180/state` | MQTT topic |

## Data Format

Log files are tab-separated with daily rotation. File names follow the pattern `co2.YYYY-MM-DD.tsv` and `pressure.YYYY-MM-DD.tsv`.

**K30 log columns:** `timestamp` (ISO 8601), `CO2` (ppm)

**BMP180 log columns:** `timestamp` (ISO 8601), `temperature` (°C), `pressure` (mbar)

## Dependencies

The logger scripts were written in Python 2 and developed for Raspbian Linux. Key packages:

- `paho-mqtt` — MQTT client for real-time data publishing
- `pyserial` — UART communication with the K30
- `RPi.GPIO` — Raspberry Pi GPIO access
- `Adafruit_Python_BMP` — BMP180 driver (install separately; see Setup)

## Authors

**Code authored by:**
Patrick T. O'Keeffe, Washington State University, Laboratory for Atmospheric Research

**Framework developed by:**
Von P. Walden, Washington State University, Laboratory for Atmospheric Research (v.walden@wsu.edu)

**Field deployment and data analysis:**
Nathan M. Lima, Yunha Huangfu, B. Tommy Jobson, and Patrick T. O'Keeffe, Washington State University

**Questions or comments:**
Open an issue or reach out via [GitHub](https://github.com/NathanL-CodeBase).

## Acknowledgments

This work was funded by the U.S. Environmental Protection Agency (EPA) Science to Achieve Results (STAR) grant program. This sensor system was one component of a broader indoor air quality measurement campaign. Contributions from Brian K. Lamb, William M. Kirk, Stephanie N. Pressley, Bonnie Lin, and David J. Cook are gratefully acknowledged.

## Citation

### Dissertation

Lima, N. M. (2022). *An Examination of Indoor Air Quality in Residential Homes Using Fine-Scale Temporal Measurements and Future Climate Model Simulations* [Dissertation, Washington State University]. ProQuest. https://www.proquest.com/openview/975cbdf830bc026b3190b2a201cd17c0/

### Journal Articles

Huangfu, Y., Lima, N. M., O'Keeffe, P. T., Kirk, W. M., Lamb, B. K., Pressley, S. N., Lin, B., Cook, D. J., Walden, V. P., & Jobson, B. T. (2019). Diel variation of formaldehyde levels and other VOCs in homes driven by temperature dependent infiltration and emission rates. *Building and Environment, 159*(106153), 1-10. https://doi.org/10.1016/j.buildenv.2019.05.031

Huangfu, Y., Lima, N., O'Keeffe, P., Kirk, W., Lamb, B., Walden, V., & Jobson, B. (2020). Whole house emission rates and loss coefficients of formaldehyde and other VOCs as a function of air change rate. *Environmental Science & Technology, 54*(4), 2143-2151. https://doi.org/10.1021/acs.est.9b05594

Kirk, W. M., Fuchs, M., Huangfu, Y., Lima, N. M., O'Keeffe, P. T., Lin, B., Jobson, B. T., Pressley, S. N., Walden, V. P., Cook, D. J., & Lamb, B. K. (2018). Indoor air quality and wildfire smoke impacts in the Pacific Northwest. *Science and Technology for the Built Environment, 24*(2), 149-159. https://doi.org/10.1080/23744731.2017.1393256

Lin, B., Huangfu, Y., Lima, N. M., Jobson, B. T., Kirk, W. M., O'Keeffe, P. T., Pressley, S. N., Walden, V. P., Lamb, B. K., & Cook, D. J. (2017). Analyzing the relationship between human behavior and indoor air quality. *Journal of Sensor and Actuator Networks, 6*, 13. https://doi.org/10.3390/jsan6030013

### Conference Papers

Huangfu, Y., Lima, N. M., O'Keeffe, P. T., Lin, B., Cook, D. J., Walden, V. P., Kirk, W. M., Pressley, S. N., Lamb, B. K., & Jobson, B. T. (2018a, July 23). Indoor air toxics in a net zero energy house under multiple ventilation settings. *The 15th Conference of the International Society of Indoor Air Quality.*

Huangfu, Y., Lima, N. M., O'Keeffe, P. T., Lin, B., Cook, D. J., Walden, V. P., Kirk, W. M., Pressley, S. N., Lamb, B. K., & Jobson, B. T. (2018b, July 23). The role of temperature on indoor concentrations of air toxic VOCs. *The 15th Conference of the International Society of Indoor Air Quality.*

Huangfu, Y., O'Keeffe, P. T., Walden, V. P., Kirk, W. M., Lamb, B. K., Jobson, B. T., Lima, N. M., Lin, B., & Cook, D. J. (2017, December 11). Indoor levels of formaldehyde and other pollutants and relationship to air exchange rates and human activities. *The American Geophysical Union Annual Fall Conference.*

Kirk, W. M., Lamb, B. K., Pressley, S. N., Jobson, B. T., Walden, V. P., Cook, D. J., Fuchs, M., O'Keeffe, P. T., Huangfu, Y., Lima, N. M., & Lingard, B. (2016, September 12). Climate change and indoor air quality in the Pacific Northwest. *Defining Indoor Air Quality: Policy, Standards and Best Practices. ASHRAE and AIVC IAQ 2016 Conference.*

Lamb, B. K., Huangfu, Y., Lima, N. M., O'Keeffe, P. T., Cook, D. J., Kirk, W. M., Lin, B., Pressley, S. N., Walden, V. P., & Jobson, B. T. (2017, October 5). Integrated measurements and model of indoor air quality. *Northwest Regional Modeling Consortium,* Richland, WA.

Lima, N. M., Huangfu, Y., Walden, V. P., Kirk, W. M., Lamb, B. K., Jobson, B. T., Pressley, S. N., O'Keeffe, P. T., Musser, A., Nolte, C. G., Spero, T. L., & Toombs, K. (2018, July 23). Simulations of indoor air quality based on future climate conditions. *The 15th Conference of the International Society of Indoor Air Quality.*

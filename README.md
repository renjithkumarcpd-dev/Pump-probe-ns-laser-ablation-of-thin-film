# Pump-Probe Ablation DAQ

Automated data acquisition and control software for nanosecond pump-probe laser ablation experiments on thin metallic films. The system synchronizes Q-switch triggering, a motorized XZ sample stage, a shutter, and oscilloscope acquisition to run unattended multi-position, multi-pulse reflectometry/transmissometry scans, and includes post-processing scripts to normalize and plot the resulting time-resolved signals.

Developed for the Laser Diagnostics Section, Institute for Plasma Research (Homi Bhabha National Institute), Gandhinagar, India, for studies of laser ablation and melt dynamics of Al and Ag thin films on glass/quartz substrates.

## What it does

- **Firmware** (`firmware/`) — Arduino sketch that drives the XZ stepper stage, the beam shutter, and the Q-switch handshake, and reports pulse/motion status back to the PC over serial.
- **Control GUI** (`control_gui/`) — a Tkinter application that talks to the Arduino and to an Agilent/Keysight oscilloscope over VISA. It runs the full experiment sequence: move to a position, arm the scope, open the shutter, wait for the Q-switch pulse, capture and save the waveform, then advance — for a specified number of positions and pulses-per-position, with manual shutter/stage jog controls and an emergency-stop.
- **Analysis** (`analysis/`) — scripts to generate normalized, averaged "nav" CSV files from the raw per-pulse waveform captures (with an outlier/similarity check across repeated positions), and to produce publication-style reflection/transmission plots across pressure, fluence, and pulse-number scans.

## Repository structure

```
pump-probe-ablation-daq/
├── firmware/
│   └── shutter_qswitch_controller.ino   # Arduino: stage motion, shutter, Q-switch handshake
├── control_gui/
│   ├── stage_controller_gui.py          # Tkinter GUI — main entry point
│   └── oscilloscope_interface.py        # VISA oscilloscope acquisition module (imported by the GUI)
├── analysis/
│   ├── nav_file_generator.py            # Raw waveform -> normalized/averaged "nav" CSV files
│   └── pump_probe_plotter.py            # Plots R/T/absorption vs. pressure, fluence, pulse number
├── requirements.txt
├── CITATION.cff
└── LICENSE
```

## Requirements

**Hardware**
- Arduino (Uno/Nano-class) driving two stepper-motor axes (X, Z), a shutter actuator, and a Q-switch handshake line
- Agilent/Keysight oscilloscope with VISA/LAN (SCPI) access
- Serial (USB) connection between PC and Arduino

**Software**
- Python 3.9+ (see `requirements.txt`)
- Arduino IDE to flash `firmware/shutter_qswitch_controller.ino`
- NI-VISA or Keysight IO Libraries Suite (provides the VISA runtime `pyvisa` connects to)

Install Python dependencies:
```bash
pip install -r requirements.txt
```

## Usage

1. **Flash the firmware** — open `firmware/shutter_qswitch_controller.ino` in the Arduino IDE, adjust pin definitions if your wiring differs, and upload it to the Arduino.
2. **Configure defaults** — in `control_gui/oscilloscope_interface.py`, set `DEFAULT_SCOPE_IP` (or VISA resource string) and `BASE_DIRECTORY` (where waveform files are saved) for your setup. In `control_gui/stage_controller_gui.py`, set `PORT` to the Arduino's serial port.
3. **Run the control GUI**:
   ```bash
   cd control_gui
   python stage_controller_gui.py
   ```
   Connect to the Arduino, enter the scan geometry (positions, pulses per position, step sizes) and shot metadata (prefix, wavelength, pressure, energy, timescale), then start the automated sequence. Waveforms are saved per pulse with filenames encoding the shot metadata.
4. **Generate normalized nav files**:
   ```bash
   python analysis/nav_file_generator.py
   ```
   Edit the `PATH` and `FILE_PREFIXES` parameters at the top of the script to point at your raw data directory.
5. **Plot results**:
   ```bash
   python analysis/pump_probe_plotter.py
   ```
   Edit the `FIXED`/`PLOT_MODE`/`LAYOUT` parameters at the top of the script to select which reflection/transmission comparison to generate.

> The analysis scripts currently use hardcoded, machine-specific paths and parameters at the top of each file (by design, for quick iteration during data collection) — edit these before running on your own data.

## Citing this software

If this code is useful in your own experiments or you build on it, please cite it — see [`CITATION.cff`](CITATION.cff), or use the "Cite this repository" button on the GitHub repo page. A BibTeX entry:

```bibtex
@software{kumar_pump_probe_ablation_daq,
  author  = {Kumar R, Renjith},
  title   = {Pump-Probe Ablation DAQ: Automated Control and Acquisition Software
             for Nanosecond Pump-Probe Laser Ablation Experiments},
  year    = {2026},
  url     = {https://github.com/<your-username>/pump-probe-ablation-daq},
  version = {1.0.0}
}
```

If you are citing this from the associated instrumentation paper, please also check that paper for the citation it uses, once published.

## License

MIT — see [`LICENSE`](LICENSE).

## Author

Renjith Kumar R — PhD Research Scholar, Laser Diagnostics Section, Institute for Plasma Research / Homi Bhabha National Institute, Gandhinagar, India.


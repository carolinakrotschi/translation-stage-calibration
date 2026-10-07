# Translation Stage Calibration

Interferometric calibration of motorized translation stages, built for my bachelor thesis at the
Max Planck Institute of Quantum Optics.

A Michelson interferometer is mounted on the stage. Every time the stage moves by half a laser
wavelength (λ/2 ≈ 394 nm at 787.3 nm), the interference signal passes through one full fringe.
Counting fringes while the stage moves therefore gives an independent, wavelength-referenced
measurement of the real displacement, which can be compared against the position the stage
controller reports.

The software controls the stage, reads out the detector in real time, counts fringes and shows
everything live in a desktop GUI.

## What it can do

- **Four detection schemes**, each as its own program:
  - **Camera**: mean intensity in a 40 × 40 px region of interest on a Thorlabs camera
  - **Single photodiode**: fast analog readout via NI-DAQmx
  - **Photodiode + reference**: a second diode normalizes out laser intensity drift
  - **Quadrature homodyne**: two photodiodes 90° out of phase; the phase φ = atan2(S₂, S₁) gives
    the direction of motion and sub-fringe resolution instead of a plain fringe count
- **Position lock** (`thor_main_homodyne_lock.py`, experimental): uses the homodyne phase as
  feedback to correct stage drift, with a configurable deadband that separates real drift from
  noise. In the thesis, the loop was limited by the mechanical response of the stage.
- **Calibration routines** for signal levels and thresholds before each measurement
- **Two stage platforms**: PI stages via the GCS protocol (`pipython`) and Thorlabs Kinesis
  long-travel stages via .NET (`pythonnet`)
- **Live plots and CSV export** of fringe count, stage position and detector signals
- **Simulation mode** for the camera, so the GUI can be tried without hardware

## Project structure

Every program exists twice: files starting with `thor_` drive the Thorlabs Kinesis stage, all
others drive the PI stage. The structure is identical.

| File | Purpose |
|---|---|
| `main_camera.py` / `thor_main_camera.py` | Fringe counting with the camera |
| `main_diode.py` / `thor_main_diode.py` | Fringe counting with one photodiode |
| `main_diode_reference.py` / `thor_main_diode_reference.py` | Measurement diode + reference diode |
| `main_homodyne.py` / `thor_main_homodyne.py` | Direction-aware quadrature homodyne detection |
| `thor_main_homodyne_lock.py` | Homodyne detection with active position lock |
| `handler_camera.py` | Camera connection, ROI, frame acquisition, simulation mode |
| `handler_diode.py` | Photodiode readout, calibration and fringe counters |
| `handler_stage.py` / `thor_handler_stage.py` | Stage connection, referencing and motion |
| `Camera/` | Thorlabs camera runtime DLLs (Windows) |
| `thorlabs_tsi_sdk-0.0.8/` | Thorlabs camera Python SDK |

Each file starts with a table of contents and a step-by-step walkthrough of how a command
travels through the code.

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install ./thorlabs_tsi_sdk-0.0.8   # only needed for the camera
python thor_main_homodyne.py
```

Controlling real hardware needs the vendor drivers (ThorCam, Kinesis, PIMikroMove, NI-DAQmx).
The full lab setup, including alignment checks and troubleshooting, is described in
[docs/SETUP.md](docs/SETUP.md).

## Tech

Python · customtkinter · matplotlib · NumPy · NI-DAQmx · PI GCS (`pipython`) · Thorlabs Kinesis
via `pythonnet` · multithreaded acquisition so hardware I/O never blocks the GUI

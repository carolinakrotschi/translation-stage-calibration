# Lab Setup Guide

This guide assumes the optical setup is already built and you want to connect the software to
it. Details on the optical components are in my bachelor thesis.

## 1. Install the software

The vendor programs below are hardware drivers. They must be installed on the lab computer for
Python to talk to the devices:

| Device | Software |
|---|---|
| Thorlabs camera | [ThorCam](https://www.thorlabs.com/software-pages/thorcam/) incl. the Windows SDK (*Programming Interfaces*) |
| Thorlabs long-travel stage | Thorlabs Kinesis (64-bit); DLLs are loaded from `C:\Program Files\Thorlabs\Kinesis\` |
| PI stage | PIMikroMove |
| Photodiodes | [NI-DAQmx](https://www.ni.com/en/support/downloads/drivers/download.ni-daq-mx.html) |

Then set up Python (3.10 or newer):

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install .\thorlabs_tsi_sdk-0.0.8    # camera only
```

On macOS/Linux the same works with `python3 -m venv .venv` and `source .venv/bin/activate`
(the GUI and the camera simulation run there too; the hardware drivers are Windows-only).

## 2. Connect the hardware

1. **Check the interference.** Open ThorCam and confirm that fringes are visible and centered.
   Camera detection uses a fixed 40 × 40 px region of interest in the middle of the sensor.
2. **PI stage.** In PIMikroMove, select the stage, run *Axes → Reference position* and enable
   the servo. Set the travel limits (`self.min_position`, `self.max_position`) in
   `handler_stage.py` to match your stage.
3. **Thorlabs Kinesis stage** (e.g. LTS150 / LTS300):
   - Set the controller's serial number in `thor_handler_stage.py` (`serial_no`). It is printed on
     the controller and shown in Kinesis.
   - Set the travel limits in `thor_handler_stage.py` (default: 0–300 mm).
   - **Close Kinesis and any simulator before starting Python.** Kinesis locks the USB connection
     and the script will fail to connect otherwise.
4. Close ThorCam and Kinesis, then start the program you need (see the table in the README).

## 3. Photodiode measurements

1. Make sure the beam is well coupled into the photodiode(s). The NI test panel that opens when
   the DAQ is plugged in shows the live signal.
2. For two-diode setups, block each beam in turn and check that both channels carry a similar
   signal.

## 4. Troubleshooting: camera misses fringes

1. Restart the program.
2. Try a different stage velocity.
3. Use a longer travel distance.
4. Adjust the dark/bright detection thresholds in the camera program. The defaults were found
   experimentally on this setup:

   ```python
   self.dark_threshold   = min_val + value_range * 0.125
   self.bright_threshold = max_val - value_range * 0.60
   ```
5. Don't touch the optical table while measuring.

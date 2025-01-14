# Setup guide

As mentioned in the [readme](./README.md), the project is intended as a proof-of-concept (not a "plug-and-play" DIY project), so I can't guarantee that all steps will be easy or perfectly documented. If that's not enough to discourage you, follow the steps below to build and run D-POINT on your own hardware.

Before continuing, [clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) this repository using git:

```bash
git clone https://github.com/jcparkyn/dpoint.git
```

## Building the stylus

<details>
<summary><strong>Tools required</strong></summary>

- A 3D printer, for printing the stylus body. I used a Creality Ender 3, but most types of 3D printer should work.
- Hot air rework station, for soldering the force sensor. You might be able to make do with a soldering iron, but I wouldn't recommend it.
- A soldering iron, for soldering the other electronics.
- An inkjet or laser printer, for printing the ArUco markers.

</details>

<details>
<summary><strong>Parts required</strong></summary>

- Development board: [Seeed Studio XIAO nRF52840 Sense](https://www.seeedstudio.com/Seeed-XIAO-BLE-Sense-nRF52840-p-5253.html).
- Force sensor: Alps Alpine [HSFPAR003A](https://tech.alpsalpine.com/e/products/detail/HSFPAR003A/) or [HSFPAR004A](https://tech.alpsalpine.com/e/products/detail/HSFPAR004A/) (preferred). These can be a little hard to find, but unfortunately, there aren't many other off-the-shelf options aside from FSRs. The [HSFPAR007A](https://tech.alpsalpine.com/e/products/detail/HSFPAR007A/) will also work, but you'll have to modify the PCB footprint.
- Custom PCB: Order this from your PCB manufacturer of choice, using the provided [gerber files](./electronics/gerbers/). Use a PCB thickness of 1.6mm.
- Printer filament. I used PLA, but most hard plastics should work.
- A 6.5x10mm compression spring. I used one from a spring kit (like [this one](https://www.ebay.com.au/itm/175482659706)), but he spring tension does not need to match exactly.
- A 10440 lithium-ion battery (e.g., [this one](https://www.ebay.com/itm/194025159718)).
- Wire, 22-24 AWG for soldering.
- Something to use as a nib for the stylus. I used a [Wacom replacement nib](https://estore.wacom.com/en-au/wacom-standard-replacement-nibs-previous-gen.html), but you might be able to substitute this with something else (e.g., 3D printer filament). If you do, make sure to adjust the dimensions on the 3D model to match.

</details>

### Micro-controller code

To upload the code to the development board:
1. Follow [these instructions](https://wiki.seeedstudio.com/XIAO_BLE/#getting-started) to download the necessary software.
1. Open [microcontroller/dpoint-arduino](./microcontroller/dpoint-arduino/) in the Arduino IDE.
1. Connect the board to your computer using a USB-C cable.
1. Click **Tools > Board > Seeed nRF52 Boards > Seeed XIAO nRF52840 Sense**.
1. Upload the code.

### Build steps

1. Print all three STL files from [print/export/](./print/export/) on your 3D printer.
1. Solder the force sensor to the custom PCB. Make sure to align the top-left of the force sensor (marked with a very small circle) with the circle on the PCB.
1. Solder the force sensor PCB, battery, and development board together using wires, following the schematic below. Make sure to cut wires the right length to fit the stylus body.
1. Place the force sensor PCB, battery, and microcontroller into the bottom half of the 3D printed stylus. The stylus body also has a hole for a switch to disconnect the battery during development, but you can ignore this.
1. Find a small, flat piece of metal or hard plastic, and glue it to the back of the 3D printed nib base where it contacts the force sensor.
1. Insert the nib into the 3D printed nib base, then fit it into the front of the stylus using the spring. You may need to hold it in place until the stylus is fully assembled.
1. Add the top half of the stylus, and secure it in place using tape or glue.
1. Print out the ArUco markers from [markers/stylus-markers.pdf](./markers/stylus-markers.pdf) at 100% scale. This has two copies of each marker (and some spares), but you only need one.
1. Cut carefully along each of the lines around the markers. The white borders are important, so don't cut them off.
1. Glue the markers to the stylus using a glue stick (or your glue of choice). Place them according to the diagram below.
1. Finally, you'll also need a ChArUco board for camera calibration, which you can print using the pattern from [markers/](./markers/). You can either attach this to a flat board, or leave it on your desk.


Wiring diagram:

<img src="https://github.com/Jcparkyn/dpoint/assets/51850908/f026f9f5-f5c4-458f-8883-b0071133f5e3" width="400px" />

Marker placement:

<img src="https://github.com/Jcparkyn/dpoint/assets/51850908/ae7184bb-005e-4dba-aeb7-f23acaa67785" width="400px" />

## Software setup

The python code requires Python 3.11, and should work on any platform.

1. Open the [python/](./python/) directory (`cd python`).
2. Create and activate a Python [virtual environment](https://docs.python.org/3/tutorial/venv.html#creating-virtual-environments) (optional, but recommended).
3. Install dependencies:
```bash
python -m pip install -r requirements.txt
```
4. Run the camera calibration script (see steps [below](#camera-calibration)). If you're using a Logitech C922 webcam like mine, you may be able to skip this step at the cost of some accuracy.
5. Optional, for better accuracy: Run the [marker calibration script](./python/calibrate_markers.py). Take at least 15 photos of the stylus from different angles (using your calibrated webcam), put them in the `IMAGE_PATH` directory for the script, then run the script.

## Running the application

Run [main.py](./python/main.py). If you have the `python` directory open in VSCode, you can do this by pressing <kbd>F5</kbd>, or running:
```bash
python -m main
```

Auto-exposure is disabled, so you might need to adjust the exposure level in `marker_tracker.py`. Make it as low as you can without the markers failing to detect. Adding a light source will help.

Put the ChArUco board on your desk in view of the webcam, then press <kbd>C</kbd> (with the camera window focused) to calibrate the camera position.

Make sure the stylus is on

### Camera calibration
To calibrate your camera:
1. Take 5-10 photos of the ChArUco board from different angles using your webcam.
2. Put the images into `calibration_pics/f30/`.
3. Run `calibrate_camera.py`.

This calibrates the camera intrinsic parameters, but not the camera position or rotation. These are calibrated at run-time.

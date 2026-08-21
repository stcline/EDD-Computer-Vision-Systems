# 🤖 Day 1: BASH Review, SSH & System Setup

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to review and sharpen your BASH and SSH skills before diving into computer vision work. A solid command-line foundation will make every debugging session, file transfer, and script execution faster and less frustrating throughout this unit. 🚀

By the end of this session you and your partner should be able to:
- Navigate the Raspberry Pi filesystem confidently using BASH commands
- Connect to your Pi via SSH from your workstation
- Organize a project directory structure for the unit
- Update the OS and set up a Python virtual environment ready for CV libraries

---

## 🖥️ Part 1: SSH & Navigation Review

### Connecting to Your Pi

Use SSH to connect headlessly from your workstation. Your instructor will provide the Pi's IP address.

```bash
ssh pi@<YOUR_PI_IP_ADDRESS>
```

Once connected, confirm you are on the right machine:

```bash
hostname
uname -a
```

### Filesystem Navigation

Practice these core navigation commands — you will use all of them every day this unit:

| Command | What it does |
|---|---|
| `pwd` | Print current working directory |
| `ls -la` | List all files with permissions and sizes |
| `cd ~/Documents` | Change to a directory |
| `cd ..` | Move up one directory level |
| `mkdir cv_project` | Create a new directory |
| `cp file.py backup.py` | Copy a file |
| `mv old.py new.py` | Rename or move a file |
| `rm file.py` | Delete a file (no recycle bin — be careful!) |
| `cat file.py` | Print a file's contents to the terminal |
| `nano file.py` | Open a file in the nano text editor |

**Partner Activity:** Partner A creates the directory structure below using only BASH commands. Partner B verifies it using `ls -R ~/cv_project`. Then switch roles and tear down and rebuild it.

```
~/cv_project/
├── notebooks/
├── scripts/
├── images/
└── models/
```

---

## ⚙️ Part 2: File Permissions & Process Control

### Permissions

Every file on Linux has read (`r`), write (`w`), and execute (`x`) permissions for the owner, group, and others. You will need to make Python scripts executable.

```bash
# View permissions
ls -l myscript.py

# Make a script executable
chmod +x myscript.py

# Run it
./myscript.py
```

The permission string `-rwxr-xr--` breaks down as:
- `-` → regular file
- `rwx` → owner can read, write, execute
- `r-x` → group can read and execute
- `r--` → others can only read

### Process Control

```bash
# See all running processes
ps aux

# Find a specific process
ps aux | grep python

# Kill a runaway script by PID
kill 1234

# Kill by name
pkill python3

# Run a script and push it to the background
python3 myscript.py &

# Bring background job back to foreground
fg
```

---

## 🐍 Part 3: Virtual Environment Setup

Starting with Raspberry Pi OS Bookworm, plain `pip install` is blocked at the system level to protect OS packages. The correct solution — and professional best practice — is to use a **Python virtual environment**.

```bash
# Create the virtual environment (do this once)
python3 -m venv ~/cv_env

# Activate it (do this every session)
source ~/cv_env/bin/activate

# Your prompt will change to show (cv_env) — you are now inside the venv
# Install a test package
pip install numpy

# Confirm it installed
python3 -c "import numpy; print(numpy.__version__)"

# Deactivate when done
deactivate
```

### Make Activation Automatic

So you never forget to activate the venv, add it to your shell startup file:

```bash
echo "source ~/cv_env/bin/activate" >> ~/.bashrc
source ~/.bashrc
```

---

## 🔄 Part 4: OS Update & Camera Enable

```bash
# Update package lists and upgrade installed packages
sudo apt update && sudo apt upgrade -y

# Install system-level dependencies OpenCV needs
sudo apt install -y python3-full libopenblas-dev libblas-dev liblapack-dev libhdf5-dev libgtk-3-0 libcap-dev

# Enable the camera interface
sudo raspi-config
# Navigate to: Interface Options → Legacy Camera (or Camera) → Enable
# Reboot when prompted
sudo reboot
```

After reboot, reconnect via SSH and verify the camera is recognized:

```bash
libcamera-still --list-cameras
```

---

## 💻 Terms to Know

- **SSH (Secure Shell):** A protocol for securely accessing another computer over a network using an encrypted terminal connection
- **BASH:** The default command-line shell on Linux/Raspberry Pi OS; stands for Bourne Again SHell
- **Virtual environment (venv):** An isolated Python environment that keeps a project's libraries separate from the system Python installation
- **PEP 668:** A Python standard adopted in 2023 that prevents pip from installing packages into the system Python — this is why bare `pip install` fails on Bookworm
- **`apt`:** The system package manager for Debian/Raspberry Pi OS; used for OS-level packages
- **`pip`:** Python's package installer; used inside a virtual environment for Python libraries
- **File permissions:** A Linux system controlling who can read, write, or execute each file (owner / group / others)
- **Process:** A running program; each has a unique Process ID (PID)
- **`~`:** Shorthand for the current user's home directory (e.g., `/home/pi`)

## 📝 Next Steps

- Confirm your `cv_env` virtual environment activates automatically on login
- Verify `libcamera-still --list-cameras` shows your Pi Camera V2
- Complete **Engineering Notebook Entry #1:** Sketch the directory structure you built; document every command used in Parts 3 and 4; note any errors encountered and how you resolved them
- Tomorrow: you will learn about the camera hardware itself and capture your first images with Python

## 📚 Resources

- [Raspberry Pi Official BASH Documentation](https://www.raspberrypi.com/documentation/)
- [PEP 668 Explanation & venv Fix — Raspberry Pi Forums](https://github.com/raspberrypi/bookworm-feedback/issues/4)
- [Python venv Official Docs](https://docs.python.org/3/library/venv.html)
- [Linux File Permissions Explained — linuxcommand.org](https://linuxcommand.org/lc3_lts0090.php)
- [libcamera Documentation](https://libcamera.org/getting-started.html)

---
---

# 🤖 Day 2: Camera Optics & the Raspberry Pi Camera V2

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to understand the physical and optical properties of the Raspberry Pi Camera V2 before writing a single line of vision code. Knowing *why* images look the way they do — why objects blur at close range, why the frame cuts off at a certain distance, why brightness changes with exposure settings — will make you a far better debugger and system designer. 🚀

By the end of this session you and your partner should be able to:
- Explain focal length, aperture, field of view, and sensor size in plain language
- Calculate the area the camera "sees" at a given working distance using the FOV formula
- Capture stills and video at multiple resolution modes and explain the tradeoffs
- Predict where to mount the camera for a given project based on your calculations

---

## 📷 Part 1: The Pi Camera V2 — Hardware Specs

### Sensor Overview

The Raspberry Pi Camera Module V2 uses the **Sony IMX219** image sensor. Key specifications:

| Property | Value |
|---|---|
| Sensor | Sony IMX219 |
| Resolution | 8 MP (3280 × 2464 pixels) |
| Sensor size | 1/4 inch |
| Pixel size | 1.12 µm |
| Aperture | f/2.0 (fixed) |
| Focal length | 3.04 mm (fixed focus) |
| Horizontal FOV | 62.2° |
| Vertical FOV | 48.8° |
| Diagonal FOV | ~73° |

The lens is **fixed focus** — there is no autofocus motor. The camera is factory-focused for objects roughly 1 meter and beyond.

### Video Modes

Different video modes use different portions of the sensor:

| Mode | Resolution | Frame Rate | Notes |
|---|---|---|---|
| Full stills | 3280 × 2464 | — | Full sensor, max detail |
| 1080p video | 1920 × 1080 | 30 fps | Center crop of sensor |
| 720p video | 1280 × 720 | 60 fps | Center crop, higher frame rate |
| 480p video | 640 × 480 | 90 fps | Small crop, fastest for CV |

**Important for CV work:** 1080p and lower video modes crop the center of the sensor, which *narrows* your actual field of view compared to the full-resolution specs above.

---

## 🔭 Part 2: Focal Length & Field of View

### What Is Focal Length?

Focal length (measured in mm) describes how strongly the lens bends light. A **shorter focal length = wider angle of view**. The V2's 3.04 mm lens is very wide for its tiny sensor — this is why it can see ~62° horizontally from such a small board.

### What Is Aperture?

Aperture (the f-number) controls how much light enters the lens. The V2's fixed f/2.0 aperture is relatively wide (lets in a lot of light), making it good for indoor environments without extra lighting.

### Calculating Your Field of View at Distance

You can calculate the width and height of what the camera sees at any distance using the following formulas:

For a camera with horizontal FOV angle θ_h and vertical FOV angle θ_v, the visible width W and height H at distance D are:

$$W = 2 \times D \times \tan\!\left(\frac{\theta_h}{2}\right)$$

$$H = 2 \times D \times \tan\!\left(\frac{\theta_v}{2}\right)$$

**Example — Bird Feeder Mount:**
If you mount the camera 40 cm (0.4 m) away from a bird feeder:

- W = 2 × 0.4 × tan(31.1°) ≈ **0.48 m (48 cm)**
- H = 2 × 0.4 × tan(24.4°) ≈ **0.36 m (36 cm)**

So your camera would see a 48 cm × 36 cm window — enough to frame a typical feeder.

### Hands-On FOV Calculation Activity

Fill in this table for your own project idea. Measure the actual area the camera sees using a ruler and compare to the calculated values:

| Distance (D) | Calculated Width | Calculated Height | Measured Width | Measured Height |
|---|---|---|---|---|
| 20 cm | | | | |
| 40 cm | | | | |
| 100 cm | | | | |
| 200 cm | | | | |

---

## 📸 Part 3: Capturing Images & Video

### Capturing a Still Image

```bash
# Make sure the venv is active
source ~/cv_env/bin/activate

# Capture a still at full resolution
libcamera-still -o ~/cv_project/images/test_full.jpg

# Capture at reduced resolution
libcamera-still --width 1280 --height 720 -o ~/cv_project/images/test_720.jpg
```

### Capturing Video

```bash
# Record 10 seconds of 1080p30 video
libcamera-vid -t 10000 --width 1920 --height 1080 -o ~/cv_project/images/test_1080.h264

# Record 10 seconds of 480p90 (better for CV — higher frame rate)
libcamera-vid -t 10000 --width 640 --height 480 --framerate 90 -o ~/cv_project/images/test_480.h264
```

### Viewing Images via SCP

Transfer images to your workstation to view them:

```bash
# Run this on your workstation (not the Pi)
scp pi@<PI_IP>:~/cv_project/images/test_full.jpg .
```

### Partner Activity: Resolution Comparison

Capture the same scene at full resolution and 640×480. Compare:
- File size (use `ls -lh`)
- Visible detail
- Time to capture

Document findings in your notebook — which mode would you choose for real-time computer vision and why?

---

## 💡 Part 4: Lighting & Exposure Basics

The V2's fixed aperture means your primary exposure controls in CV scripts will be:
- **Shutter speed** — longer = brighter but more motion blur
- **ISO/gain** — higher = brighter but more noise (grain)
- **Scene lighting** — the most impactful variable you can actually control

For CV to work reliably, consistent lighting matters more than resolution. A diffused LED ring light or a window facing north (no direct sun) dramatically improves detection accuracy. Keep this in mind when designing your final project's mounting system.

---

## 💻 Terms to Know

- **Focal length:** The distance (in mm) between the lens and the image sensor when the lens is focused at infinity; determines angle of view
- **Aperture (f-number):** A ratio describing how wide the lens opening is; lower f-number = more light admitted
- **Field of View (FOV):** The angular extent of the scene the camera can capture; expressed in degrees horizontally, vertically, or diagonally
- **Sensor size:** The physical dimensions of the image sensor; larger sensors capture more light and detail
- **Fixed focus:** A lens with no autofocus motor; depth of field is determined by the focal length and aperture
- **Resolution:** The number of pixels in an image (width × height); higher resolution = more detail but larger files and slower processing
- **Frame rate (fps):** Frames captured per second; higher fps = smoother motion detection but more CPU load
- **Depth of field:** The range of distances that appear acceptably sharp in an image
- **Exposure:** The total amount of light reaching the sensor; controlled by aperture, shutter speed, and ISO

## 📝 Next Steps

- Complete the FOV calculation table for at least 3 distances relevant to your project idea
- Transfer at least one image to your workstation via SCP
- Complete **Engineering Notebook Entry #1** (continued): Add a sketch of the camera sensor diagram with FOV angles labeled; include your completed FOV table with both calculated and measured values; write a short paragraph predicting the best mounting distance for your project concept
- Tomorrow: you will open a live camera feed in Python using OpenCV and start processing frames in real time

## 📚 Resources

- [Official Raspberry Pi Camera Documentation](https://www.raspberrypi.com/documentation/accessories/camera.html)
- [Raspberry Pi Camera V2 Specs — Waveshare](https://www.waveshare.com/rpi-camera-v2.htm)
- [Lens FOV Calculator — Commonlands Optics](https://commonlands.com/pages/camera-fov-calculator)
- [libcamera Documentation & CLI Reference](https://libcamera.org/getting-started.html)
- [Understanding Camera Exposure — Cambridge in Colour](https://www.cambridgeincolour.com/tutorials/camera-exposure.htm)

---
---

# 🤖 Day 3: Intro to OpenCV — Images, Frames & Basic Operations

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to get hands-on with OpenCV — the open-source computer vision library that powers most of the CV techniques in this unit. You will open a live camera feed, manipulate frames in real time, and build the basic pipeline that every future lab will extend. 🚀

By the end of this session you and your partner should be able to:
- Open and display a live camera feed using Python and OpenCV
- Perform core image operations: resize, color space conversion, blur, and annotation
- Understand why the HSV color space is preferred over RGB for CV tasks
- Write a complete Python script that captures, processes, and saves an annotated image

---

## 🐍 Part 1: OpenCV Install & First Image

### Install OpenCV in Your venv

```bash
source ~/cv_env/bin/activate
pip install opencv-python
python3 -c "import cv2; print(cv2.__version__)"
```

### Open a Static Image

Create `~/cv_project/scripts/day3_image.py`:

```python
import cv2

# Load an image from disk
img = cv2.imread('/home/pi/cv_project/images/test_full.jpg')

# Print shape: (height, width, channels)
print(f"Image shape: {img.shape}")
print(f"Data type: {img.dtype}")

# Display it (requires a monitor or VNC — if headless, skip to saving)
cv2.imshow('My Image', img)
cv2.waitKey(0)         # Wait for any key press
cv2.destroyAllWindows()
```

> **Headless Pi tip:** If you are connected via SSH without a display, skip `imshow` for now and save processed images to disk with `cv2.imwrite()`, then use SCP to view them.

---

## 🎥 Part 2: Live Camera Feed

### The Capture Loop

This is the fundamental pattern for all real-time CV — every future script builds on this loop:

```python
import cv2

cap = cv2.VideoCapture(0)   # 0 = first camera device

# Set resolution
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

while True:
    ret, frame = cap.read()     # Read one frame
    if not ret:
        print("Failed to grab frame")
        break

    # --- All processing happens here ---
    cv2.imshow('Live Feed', frame)

    # Press 'q' to quit
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

Save as `~/cv_project/scripts/day3_live.py` and run:

```bash
python3 ~/cv_project/scripts/day3_live.py
```

---

## 🎨 Part 3: Core Image Operations

Add each of these operations *inside* the while loop, one at a time. Observe the effect before adding the next one.

### Resize

```python
small = cv2.resize(frame, (320, 240))
```

### Grayscale Conversion

```python
gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
```

### Gaussian Blur (noise reduction before detection)

```python
blurred = cv2.GaussianBlur(gray, (15, 15), 0)
# Kernel size (15,15) must be odd numbers; larger = more blur
```

### Drawing Overlays

```python
# Rectangle: (image, top-left corner, bottom-right corner, color BGR, thickness)
cv2.rectangle(frame, (50, 50), (200, 200), (0, 255, 0), 2)

# Circle: (image, center, radius, color BGR, thickness)
cv2.circle(frame, (320, 240), 30, (255, 0, 0), 3)

# Text: (image, text, origin, font, scale, color BGR, thickness)
cv2.putText(frame, 'Hello CV!', (10, 30),
            cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
```

---

## 🌈 Part 4: RGB vs. HSV Color Space

### Why Not RGB?

In RGB, the color of a pixel is split across three channels (Red, Green, Blue) in a way that makes it hard to isolate a specific color under changing lighting. A red apple in bright light and a red apple in shadow have very different RGB values.

### How HSV Works

HSV separates color information into three independent channels:

| Channel | Meaning | Range (OpenCV) |
|---|---|---|
| **H** (Hue) | The actual color (red, green, blue...) | 0–179 |
| **S** (Saturation) | How vivid/pure the color is | 0–255 |
| **V** (Value) | Brightness | 0–255 |

This makes it easy to define a color range that is robust to lighting changes — you only tune the Hue range and leave S/V broad.

### Convert to HSV

```python
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

# Example: create a mask for blue objects
lower_blue = (100, 100, 50)
upper_blue = (130, 255, 255)
mask = cv2.inRange(hsv, lower_blue, upper_blue)

# Apply mask to original frame — only blue pixels survive
result = cv2.bitwise_and(frame, frame, mask=mask)
```

---

## 🧪 Part 5: Lab Challenge

Build a complete script `~/cv_project/scripts/day3_annotated.py` that:

1. Opens the live camera feed at 640×480
2. On each frame, displays:
   - Your team name in the top-left corner
   - Current frame dimensions (width × height) in the bottom-left corner
   - A timestamp using Python's `datetime` module in the top-right corner
3. Converts the frame to HSV and displays the HSV version side-by-side with the original using `cv2.hconcat([frame, cv2.cvtColor(hsv, cv2.COLOR_HSV2BGR)])`
4. When the user presses `s`, saves the current frame to `~/cv_project/images/snapshot.jpg`
5. When the user presses `q`, quits cleanly

**Hint for timestamp:**
```python
from datetime import datetime
timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
```

---

## 💻 Terms to Know

- **OpenCV:** Open Source Computer Vision Library; a Python/C++ library for real-time image and video processing
- **Frame:** A single image captured from a video stream; video is a sequence of frames displayed at a set frame rate
- **BGR:** Blue-Green-Red — OpenCV's default channel order (note: *not* RGB like most other tools)
- **HSV:** Hue-Saturation-Value color model; preferred for color-based CV because it separates color (hue) from brightness (value)
- **Mask:** A binary image (pixels are either 0 or 255) used to isolate regions of interest in another image
- **Kernel:** In image processing, a small matrix applied across an image to perform operations like blur or edge detection
- **Contour:** A curve joining all continuous points along a boundary with the same color or intensity
- **`cv2.VideoCapture`:** The OpenCV class for reading video from a camera or file
- **`cv2.waitKey(1)`:** Waits 1 ms for a key press; essential in the capture loop to allow the display to refresh
- **Headless:** Running a Raspberry Pi without a monitor, accessed only via SSH

## 📝 Next Steps

- Push your Day 3 scripts to your GitHub repository
- Complete **Engineering Notebook Entry #1** (continued): Paste your annotated script, include a screenshot of the side-by-side HSV/RGB output (transfer via SCP), and answer in writing: *Why is HSV better than BGR for detecting a specific color under changing light conditions?*
- Tomorrow: you will formally launch the EDD design process — concept sketches, problem statements, and project selection

## 📚 Resources

- [OpenCV Official Python Documentation](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)
- [OpenCV Python Tutorials — PyImageSearch](https://pyimagesearch.com/start-here/)
- [Intro to OpenCV with Raspberry Pi — Circuit Rocks](https://learn.circuit.rocks/introduction-to-opencv-using-the-raspberry-pi)
- [HSV Color Space Explained — LearnOpenCV](https://learnopencv.com/color-spaces-in-opencv-cpp-python/)
- [Install OpenCV on Raspberry Pi — Random Nerd Tutorials](https://randomnerdtutorials.com/install-opencv-raspberry-pi/)

---
---

# 🤖 Day 4: Design Process Launch — Problem Identification & Concept Sketches

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to formally launch the EDD design process for your Computer Vision System project. You and your partner will define a real problem worth solving, generate multiple design concepts, and begin the documentation trail that will follow your project all the way to Demo Day. 🚀

By the end of this session you and your partner should be able to:
- Write a clear, specific problem statement that identifies a user, a need, and measurable criteria
- Independently generate at least three annotated concept sketches
- Evaluate and discuss tradeoffs between concepts using engineering reasoning
- Complete the first formal entry in your Engineering Notebook

---

## 🔍 Part 1: The EDD Design Process

### Where We Are

Engineering Design and Development (EDD) follows a structured design process. In this unit we are working through the full cycle:

```
Problem Identification → Research → Concept Generation →
Decision Matrix → Design Brief → Plan → Build → Test → Iterate → Present
```

Today covers **Problem Identification** and **Concept Generation**. This is not a shortcut phase — the quality of your problem definition directly determines the quality of your final system.

### What Makes a Good Problem Statement?

A strong problem statement answers three questions:
1. **Who** has the problem? (the user/stakeholder)
2. **What** is the unmet need or inefficiency?
3. **How will we know** when the problem is solved? (measurable criteria)

**Weak example:** "I want to build a camera system."

**Strong example:** "Backyard birders often miss seeing which species visit their feeder because they cannot watch it constantly. A visual AI system that automatically identifies and logs bird species by photo, with timestamps, would allow users to review visits without being present."

---

## 💡 Part 2: Computer Vision Application Gallery

Before sketching, review the landscape of what is possible with the hardware and skills you are building. Your final project must use at least one of the CV techniques from this unit.

| CV Technique | Example Applications |
|---|---|
| **Color/Shape Tracking** | Sorting objects on a conveyor, tracking a sports ball, QC inspection |
| **Object Detection (TFLite)** | Wildlife camera, vehicle counter, inventory scanner, delivery detection |
| **Pose Estimation** | Rep counter for exercise, posture coach, gesture-controlled interface |
| **Face Detection/Recognition** | Security door system, classroom attendance, personalized greetings |
| **Multi-technique combined** | Smart birdfeeder (detect bird → identify species → log + notify) |

**Class Discussion Questions (10 minutes):**
- What problem in your school, home, or community could a camera system help with?
- Who would benefit from your system and how would they use it?
- What CV technique(s) from the list above would your solution require?

---

## ✏️ Part 3: Individual Concept Sketching

### Rules for Concept Sketching

- **Work independently** — do not show your partner your sketches until both are done
- Sketches must be **hand-drawn** — no computer diagrams at this stage
- Each sketch must show: the physical setup, camera placement, what the camera sees, and what the system does with that information
- Label every major component
- Add brief annotations explaining key design choices

### Each Partner Must Sketch at Least 3 Concepts

For each concept, include:

1. **System sketch** — physical drawing of the complete setup
2. **Name** — give the concept a short memorable name
3. **Problem it solves** — one sentence
4. **CV technique used** — which module(s) from this unit apply
5. **Output/action** — what does the system *do* with what it sees? (log to file, sound alarm, send notification, trigger LED, etc.)
6. **Biggest unknown** — what would you need to figure out to make this work?

---

## 🤝 Part 4: Partner Concept Discussion

After independent sketching, share your concepts with your partner and discuss:

- Which concepts use skills you will actually practice in this unit?
- Which concepts are physically feasible with a Raspberry Pi 4 and Camera V2?
- Which concepts would be most useful or impactful for a real user?
- Which concept excites both of you the most?

**Do not make a final decision today.** You will complete a formal Decision Matrix on Day 13 after you have practiced all the CV techniques. However, note your top 1–2 ideas because they will inform how you approach the skill-building labs.

---

## 📓 Engineering Notebook Requirements

Every notebook entry must include the following at the top:
```
Date: ___________
Partner Names: ___________
Entry #: ___________
Objective: (one sentence — what were you trying to accomplish today?)
```

And must include at the bottom:
```
Results/Observations: (what happened? include data, screenshots, or sketches)
Next Steps: (what will you do next session based on what you learned?)
```

### Entry #2 — Required Contents Today

- [ ] Problem statement draft (use the who/what/how structure from Part 1)
- [ ] 3 annotated concept sketches (Partner A's)
- [ ] 3 annotated concept sketches (Partner B's)
- [ ] 1 paragraph: compare and contrast your two sets of concepts — where did your ideas overlap? Where were they different?
- [ ] 1 paragraph: which concept are you most interested in pursuing and why?

---

## 💻 Terms to Know

- **Design process:** A structured, iterative approach to solving engineering problems; in EDD it moves from problem identification through prototyping and testing
- **Problem statement:** A clear, concise description of an unmet user need that guides the entire design effort
- **Concept sketch:** A hand-drawn ideation drawing that communicates a design idea quickly, without committing to exact dimensions or specifications
- **Stakeholder:** Anyone who is affected by or has an interest in the outcome of a design project (users, operators, bystanders)
- **Design criteria:** Measurable standards a design must meet to be considered successful (e.g., "detects species with ≥80% accuracy")
- **Design constraints:** Limitations the design must work within (e.g., "must run on a Raspberry Pi 4," "must cost under $50")
- **Iteration:** Repeating the design process loop — build, test, observe, revise — to improve a design over time
- **Decision matrix:** A table that scores multiple design concepts against weighted criteria to make a defensible design selection
- **Design brief:** A formal document summarizing the problem, stakeholders, criteria, and constraints for a design project

## 📝 Next Steps

- Photograph or scan your concept sketches and upload them to your GitHub repo under `notebooks/`
- Begin thinking about which CV techniques you are most excited to explore in the upcoming labs — those interests can guide your final project choice
- Complete **Engineering Notebook Entry #2** (all items in the checklist above) before next class
- Tomorrow: you begin the CV skill-building labs — starting with color tracking and shape detection

## 📚 Resources

- [PLTW EDD Course Framework](https://www.pltw.org/our-programs/pltw-engineering)
- [How to Write a Problem Statement — Engineering Design Process](https://www.sciencebuddies.org/science-fair-projects/engineering-design-process/engineering-design-process-steps)
- [Concept Sketching Tips for Engineers — Autodesk](https://www.autodesk.com/products/fusion-360/blog/what-is-a-concept-sketch/)
- [Computer Vision Real-World Applications Overview — OpenCV](https://opencv.org/applications-of-computer-vision/)
- [Smart Birdfeeder Project (Unit Inspiration) — GitHub](https://github.com/stcline/smart-birdfeeder)

# 🤖 EDD Computer Vision Systems — Partner Project

**PLTW Engineering Design & Development | Lynwood High School**
**Unit Duration:** 4 Weeks (20 class periods × 1 hour)
**Hardware:** Raspberry Pi 4 · Raspberry Pi Camera Module V2 (8MP)
**Core Libraries:** OpenCV · MediaPipe · TensorFlow Lite

***

## 📋 Table of Contents

- [Project Description](#-project-description)
- [Unit Schedule](#-unit-schedule)
- [Repository Structure](#-repository-structure)
- [Partner Expectations](#-partner-expectations)
- [Engineering Notebook Requirements](#-engineering-notebook-requirements)
- [Presentation Goals](#-presentation-goals)
- [Setup & Installation](#-setup--installation)
- [Resources](#-resources)

***

## 🔭 Project Description

In this project you and your partner will **design, build, program, and test a computer vision system** that solves a real-world problem of your choosing. Inspired by the [Smart Birdfeeder Project](https://github.com/stcline/smart-birdfeeder), your system will use a Raspberry Pi 4 and Camera Module V2 to perform visual AI tasks — identifying objects, tracking motion, estimating human pose, or recognizing faces — and take a meaningful action based on what it sees.

### What This Project Is

- A full engineering design cycle from problem identification through testing and presentation
- A team software and hardware project built on a Raspberry Pi 4
- A demonstration that you can use real computer vision tools to solve a genuine problem

### Possible Final System Examples

| Concept | CV Technique | Output |
|---|---|---|
| Smart bird/wildlife feeder | TFLite object detection | Species log with timestamps |
| Security door system | Face recognition | Alert or log for unknown visitors |
| Fitness rep counter | Pose estimation | Audio/visual feedback on reps |
| Inventory parts counter | Object detection + counting | CSV count log |
| Color-sort QC station | HSV color tracking | Pass/fail indicator |
| Gesture-controlled device | Hand/pose landmarks | Trigger external hardware |
| Hallway traffic counter | Object detection | Occupancy log by time of day |

Your final project does **not** need to be one of these — use the decision matrix process to find the right fit for your team.

***

## 📅 Unit Schedule

All dates are approximate. Your instructor will confirm exact dates on Day 1.

> **Key:** 📓 = Notebook entry due | 🔧 = Hands-on lab | 🎨 = Design deliverable | 🧪 = Testing

### Phase 1 — Foundations (Days 1–4)

| Day | Topic | Deliverables |
|---|---|---|
| **Day 1** | BASH review, SSH, virtual environment setup, OS update | Repo created, venv activated, camera tested 🔧 |
| **Day 2** | Camera optics: focal length, FOV, sensor specs, capture modes | FOV calculation table, image captures at multiple resolutions 🔧 📓 Entry #1 |
| **Day 3** | Intro to OpenCV: live feed, image ops, HSV color space | `day3_annotated.py` pushed to repo 🔧 📓 Entry #1 (cont.) |
| **Day 4** | EDD design process: problem identification, concept sketches | 3 concept sketches per partner 🎨 📓 Entry #2 |

### Phase 2 — CV Skill Building (Days 5–12)

| Day | Topic | Deliverables |
|---|---|---|
| **Day 5** | Module A: Color tracking with HSV masking and trackbars | `color_tracking.py` 🔧 |
| **Day 6** | Module A: Shape detection with contours and Hough transforms | `shape_detection.py` 🔧 📓 Entry #3 |
| **Day 7** | Module B: TFLite object detection — model setup and inference | `object_detect.py` 🔧 |
| **Day 8** | Module B: Filtering detections and logging to CSV | `detection_logger.py` + CSV output 🔧 📓 Entry #4 |
| **Day 9** | Module C: Pose estimation with MediaPipe — 33 keypoints | `pose_basic.py` 🔧 |
| **Day 10** | Module C: Joint angle calculation and gesture logic | `pose_angles.py` 🔧 📓 Entry #5 |
| **Day 11** | Module D: Face detection with Haar cascades + Face Mesh | `face_detect.py` 🔧 |
| **Day 12** | Module D: Face recognition — known vs. unknown classifier | `face_recognize.py` 🔧 📓 Entry #6 |

### Phase 3 — Design & Planning (Days 13–15)

| Day | Topic | Deliverables |
|---|---|---|
| **Day 13** | Design brief + decision matrix | Design brief + scored decision matrix 🎨 📓 Entry #7 |
| **Day 14** | Gantt chart + risk planning | Gantt chart (Google Sheets) shared with instructor 🎨 📓 Entry #8 |
| **Day 15** | System architecture + FOV/mounting design | Architecture diagram + camera placement calc 🎨 📓 Entry #9 |

### Phase 4 — Build, Code & Test (Days 16–19)

| Day | Topic | Deliverables |
|---|---|---|
| **Day 16** | Build Sprint 1: core CV pipeline | Working proof-of-concept, code pushed 🔧 |
| **Day 17** | Build Sprint 2: integration and output | Detection logging working, 10-trial test table 🧪 |
| **Day 18** | Build Sprint 3: refinement and reliability | Iteration log documenting all changes 🧪 |
| **Day 19** | Final testing + documentation polish | 20-trial test protocol, metrics table, repo polished 🧪 📓 Entry #10 |

### Phase 5 — Presentation (Day 20)

| Day | Topic | Deliverables |
|---|---|---|
| **Day 20** | Demo Day — live presentation | 5-min presentation + live demo + Q&A |

***

## 📁 Repository Structure

Create your own repository on GitHub (do not clone this one) and clone it on your raspberry pi.

`git clone [your repo url]`

Your repository must follow this structure. Create all folders on Day 1.

```
your-project-name/
│
├── README.md                  ← This file (keep it updated!)
│
├── scripts/                   ← All Python scripts
│   ├── day3_annotated.py
│   ├── color_tracking.py
│   ├── shape_detection.py
│   ├── object_detect.py
│   ├── detection_logger.py
│   ├── pose_basic.py
│   ├── pose_angles.py
│   ├── face_detect.py
│   ├── face_recognize.py
│   └── final_project.py       ← Your final project script
│
├── models/                    ← Downloaded TFLite model files (.tflite, labels.txt)
│
├── images/                    ← Test images, sample captures, screenshots
│   └── known_faces/           ← Reference photos for face recognition (if used)
│
├── data/                      ← Output CSV logs from detection/logging scripts
│
├── notebooks/                 ← Scanned/photographed engineering notebook pages
│   ├── entry_01_YYYY-MM-DD.jpg
│   ├── entry_02_YYYY-MM-DD.jpg
│   └── ...
│
├── design/                    ← Design deliverables
│   ├── concept_sketches/      ← Photos of hand-drawn sketches (both partners)
│   ├── design_brief.md
│   ├── decision_matrix.md (or .xlsx)
│   ├── gantt_chart.md (or link to Google Sheet)
│   └── system_architecture.md
│
└── docs/
    └── test_results.md        ← Formal test protocol and results table
```

Set this up on Day 1:

```bash
cd ~/cv_project
mkdir -p scripts models images/known_faces data notebooks design/concept_sketches docs
touch README.md
git add .
git commit -m "Initial repo structure"
git push
```

### AT THE END OF EVERY DAY - Push your changes on your local repo to GitHub

***

## 👥 Partner Expectations

This is a **partner project**. Both partners are equally responsible for the work, documentation, and final presentation. The repository must clearly show that **both people are actively contributing** — not just one person doing all the work.

### Contribution Evidence Requirements

The simplest and most transparent way to show both partners are contributing is through **co-authored commits**. Every commit pushed to the repo should credit both partners using GitHub's co-author trailer:

```bash
git commit -m "Add color tracking script with HSV masking

Co-authored-by: Partner Name <partner_github_email@example.com>"
```

GitHub automatically recognizes this trailer and shows **both names** on the commit in the repo, giving both partners credit in the contribution graph.

**To make this easier**, create a helper alias on your Pi on Day 1 (replace with your actual partner's info):

```bash
# Add to ~/.bashrc
alias gcp='git commit --template ~/.git_coauthor_template'

# Create the template file
echo "
Co-authored-by: Partner Full Name <partner@email.com>" > ~/.git_coauthor_template
```

Or simply paste the `Co-authored-by:` line manually at the bottom of every commit message.

### What the Instructor Will Check

At the end of the unit, the commit history will be reviewed for:

- [ ] **Both names appear as co-authors** on the majority of commits
- [ ] **Commits are spread across the unit** — not all pushed on the last day
- [ ] **Commit messages are descriptive** — not just "update" or "fix" (see examples below)
- [ ] **Both partners' notebook entries are scanned** and uploaded to `notebooks/`
- [ ] **Both partners speak** during the Day 20 presentation

### Commit Message Examples

| ❌ Not Acceptable | ✅ Acceptable |
|---|---|
| `update` | `Add HSV mask for blue object tracking` |
| `fix stuff` | `Fix contour detection failing on dark backgrounds` |
| `working` | `Complete pose angle calculation for elbow joint` |
| `final` | `Add final test results table to docs/test_results.md` |

### Division of Work

There is no required split of tasks — partners may divide work however makes sense. However, the expectation is:

- Each partner writes and commits at least some code
- Each partner maintains their **own** engineering notebook (see below)
- Each partner contributes to the design deliverables (concept sketches are done individually by design)
- Each partner is prepared to answer technical questions during the presentation

***

## 📓 Engineering Notebook Requirements

**Both partners maintain separate, individual engineering notebooks.** These are not shared — each person keeps their own.

### Format

Notebooks are physical in your PLTW Engineering Notebook. Entries must be **dated, signed by both partners, and non-editable after the session ends** (i.e., no going back and changing old entries).

### Required Header for Every Entry

```
Title: CV Project - [entry number and title from below]
Date: ___________
Entry #: ___________
Partner Names: ___________
Objective: (one sentence — what were you trying to accomplish today?)
```

### Required Footer for Every Entry

```
Results / Observations:
(What happened? Include data tables, sketches, or screenshots as appropriate.)

Next Steps:
(What will you do next session, and what question do you still need to answer?)

Bottom of page signatures, dates and other boxes complete.
```

### The 10 Required Entries

| Entry | Due | Content |
|---|---|---|
| #1 | Day 3 | BASH setup log + camera pipeline sketch + HSV reflection paragraph |
| #2 | Day 4 | Problem statement + 3 annotated concept sketches + partner comparison paragraph |
| #3 | Day 6 | Color tracking + shape detection lab results (screenshots, conditions, failures) |
| #4 | Day 8 | TFLite detection accuracy table across 10 trials with varying conditions |
| #5 | Day 10 | Keypoint diagram + angle calculation method + rep counter test results |
| #6 | Day 12 | Face recognition accuracy table + ethics reflection paragraph |
| #7 | Day 13 | Completed design brief + scored decision matrix with justification |
| #8 | Day 14 | Risk log (5+ risks with mitigation plans) + Gantt chart summary |
| #9 | Day 15 | System architecture diagram + FOV/mounting calculation |
| #10 | Day 19 | Final test protocol results (20+ trials) + metrics table + design process reflection |

### Uploading Notebook Pages to the Repo

After each session, photograph or scan your notebook entry and upload it to the `notebooks/` folder:

```bash
# Example filename convention
notebooks/entry_03_2026-09-15.jpg
```

Then commit and push:

```bash
git add notebooks/
git commit -m "Add notebook entries 3 and 4

Co-authored-by: Partner Name <partner@email.com>"
git push
```

***

## 🎤 Presentation Goals

Presentations take place on **Day 20** and run **5 minutes + 2-minute Q&A**.

### Presentation Structure

1. **Problem (30 sec)** — What problem does your system solve? Who is the user?
2. **Design Decisions (1 min)** — Walk through your decision matrix: what concepts did you consider, and why did you choose this one? Show a concept sketch.
3. **Live Demo (2 min)** — Run your system live in front of the class. Narrate what is happening.
4. **Results (1 min)** — Share your test metrics: detection rate, false positive rate, performance under different conditions. What worked? What didn't?
5. **Future Improvements (30 sec)** — If you had two more weeks, what would you improve or add?

### Presentation Expectations

- **Both partners must speak** for roughly equal time
- The demo must be **live** — not a pre-recorded video
- Slides are optional but not required; the demo is the main event
- Be prepared to answer technical questions such as:
  - "How did you choose your confidence threshold?"
  - "What does your system do if it gets a false positive?"
  - "Why did you choose TFLite over OpenCV for detection?"
  - "How did you calculate the right mounting distance for your camera?"

### Grading Rubric Overview

| Category | Weight | Description |
|---|---|---|
| Engineering Notebook (×2, individual) | 30% | 10 required entries, dated, complete, uploaded to repo |
| Repository & Commit History | 20% | Structure followed, both partners co-authoring, descriptive messages |
| Design Deliverables | 20% | Design brief, decision matrix, Gantt chart, architecture diagram |
| Final System Performance | 15% | Meets stated design criteria; accuracy/reliability data presented |
| Presentation & Demo | 15% | Both partners speak, live demo runs, technical questions answered |

***

## ⚙️ Setup & Installation

### First-Time Setup (Day 1)

```bash
# SSH into your Pi
ssh pi@<YOUR_PI_IP>

# Create and activate virtual environment
python3 -m venv ~/cv_env
echo "source ~/cv_env/bin/activate" >> ~/.bashrc
source ~/.bashrc

# Install system dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-full libatlas-base-dev libhdf5-dev libgtk-3-0 libcap-dev

# Install Python libraries
pip install opencv-python mediapipe tflite-runtime face_recognition

# Enable the camera
sudo raspi-config   # Interface Options → Camera → Enable
sudo reboot
```

### Cloning the Repo on Your Pi

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git ~/cv_project
cd ~/cv_project
```

### Running Scripts

```bash
source ~/cv_env/bin/activate
python3 scripts/final_project.py
```

***

## 📚 Resources

### Documentation
- [Raspberry Pi Official Documentation](https://www.raspberrypi.com/documentation/)
- [OpenCV Python Docs](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)
- [MediaPipe Solutions Guide](https://developers.google.com/mediapipe/solutions/guide)
- [TensorFlow Lite Object Detection on Raspberry Pi](https://github.com/tensorflow/examples/tree/master/lite/examples/object_detection/raspberry_pi)

### Tutorials & Guides
- [Install OpenCV on Raspberry Pi — Random Nerd Tutorials](https://randomnerdtutorials.com/install-opencv-raspberry-pi/)
- [Pose Estimation & Face Landmark — Core Electronics](https://core-electronics.com.au/guides/pose-and-face-landmark-raspberry-pi/)
- [Color Object Tracking — Instructables](https://www.instructables.com/Color-Detection-Based-Object-Tracking/)
- [Shape Detection with OpenCV — Instructables](https://www.instructables.com/Object-Tracking-Based-on-Shape-Using-OpenCV-on-Bra/)

### Project Inspiration
- [Smart Birdfeeder (Unit Inspiration)](https://github.com/stcline/smart-birdfeeder)

### PLTW EDD Design Process
- [Engineering Design Process — Science Buddies](https://www.sciencebuddies.org/science-fair-projects/engineering-design-process/engineering-design-process-steps)

***

*PLTW EDD · Lynwood High School · Computer Vision Systems Unit*

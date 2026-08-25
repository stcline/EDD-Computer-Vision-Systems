# 🤖 Day 5: Module A — Color Tracking with OpenCV

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to build your first real object-tracking system using color as the detection signal. Color tracking is the simplest and fastest computer vision technique you'll learn, and it's a great warm-up before moving into heavier machine-learning-based detection later this unit. 🚀

By the end of this session you and your partner should be able to:
- Convert live video to HSV and build a color mask using `cv2.inRange`
- Use trackbars to interactively tune a color range in real time
- Find the centroid of a detected color blob and draw a marker on it
- Explain why color tracking fails under certain lighting or background conditions

---

## 🎨 Part 1: Review — Why HSV for Color Tracking?

Recall from Day 3: HSV separates color (**Hue**) from lighting conditions (**Saturation**, **Value**), which makes it far more robust than RGB for isolating a specific color across different lighting.

```python
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
```

Every script in this lesson starts from the Picamera2 + OpenCV pattern you built on Day 3. Keep that capture loop structure — you're only adding new processing steps inside it.

---

## 🎛️ Part 2: Building Interactive Trackbars

Instead of guessing at HSV threshold numbers, we can build sliders that let you tune the color range live while watching the mask update.

Create `~/cv_project/scripts/color_tracking.py`:

```python
import cv2
import numpy as np
from picamera2 import Picamera2

def nothing(x):
    pass

cv2.namedWindow('Trackbars')
cv2.createTrackbar('H Min', 'Trackbars', 0, 179, nothing)
cv2.createTrackbar('H Max', 'Trackbars', 179, 179, nothing)
cv2.createTrackbar('S Min', 'Trackbars', 100, 255, nothing)
cv2.createTrackbar('S Max', 'Trackbars', 255, 255, nothing)
cv2.createTrackbar('V Min', 'Trackbars', 100, 255, nothing)
cv2.createTrackbar('V Max', 'Trackbars', 255, 255, nothing)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        h_min = cv2.getTrackbarPos('H Min', 'Trackbars')
        h_max = cv2.getTrackbarPos('H Max', 'Trackbars')
        s_min = cv2.getTrackbarPos('S Min', 'Trackbars')
        s_max = cv2.getTrackbarPos('S Max', 'Trackbars')
        v_min = cv2.getTrackbarPos('V Min', 'Trackbars')
        v_max = cv2.getTrackbarPos('V Max', 'Trackbars')

        lower = np.array([h_min, s_min, v_min])
        upper = np.array([h_max, s_max, v_max])
        mask = cv2.inRange(hsv, lower, upper)
        result = cv2.bitwise_and(frame, frame, mask=mask)

        cv2.imshow('Original', frame)
        cv2.imshow('Mask', mask)
        cv2.imshow('Result', result)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Lab Activity: Tune to a Real Object

- Place a brightly colored object (a ball, marker cap, sticky note) in front of the camera
- Adjust the trackbars until only your object appears white in the **Mask** window and everything else is black
- Record the six values that worked in your notebook — you'll hard-code these into the next script

---

## 🎯 Part 3: Tracking the Centroid

Once you know your HSV range, build a script that locks onto the object and tracks its center point using contours.

```python
import cv2
import numpy as np
from picamera2 import Picamera2

# Replace these with YOUR tuned values from Part 2
LOWER = np.array([35, 100, 100])
UPPER = np.array([85, 255, 255])

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        mask = cv2.inRange(hsv, LOWER, UPPER)
        mask = cv2.erode(mask, None, iterations=2)
        mask = cv2.dilate(mask, None, iterations=2)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        if contours:
            largest = max(contours, key=cv2.contourArea)
            if cv2.contourArea(largest) > 300:  # ignore tiny noise blobs
                (x, y), radius = cv2.minEnclosingCircle(largest)
                M = cv2.moments(largest)
                cx = int(M["m10"] / M["m00"])
                cy = int(M["m01"] / M["m00"])

                cv2.circle(frame, (int(x), int(y)), int(radius), (0, 255, 0), 2)
                cv2.circle(frame, (cx, cy), 5, (0, 0, 255), -1)
                cv2.putText(frame, f"({cx},{cy})", (cx + 10, cy),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 2)

        cv2.imshow('Color Tracking', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### What's Happening Here?

- `erode`/`dilate` remove small noise specks from the mask before analysis
- `cv2.findContours` finds all the blob outlines in the mask
- `cv2.moments` calculates the mathematical centroid (center of mass) of the largest blob
- The `area > 300` check prevents the system from "locking on" to tiny background noise

---

## 💻 Terms to Know

- **HSV mask:** A binary (black/white) image showing only pixels that fall within a specified Hue-Saturation-Value range
- **Trackbar:** An interactive GUI slider in OpenCV used to adjust a numeric parameter live
- **Erosion/Dilation:** Morphological operations that shrink (erode) or grow (dilate) white regions in a binary mask; used together to remove small noise blobs
- **Contour:** A curve that traces the outline of a connected white region in a binary mask
- **Centroid:** The geometric center point of a shape, calculated from image moments
- **Image moments:** Mathematical properties of a shape (area, center of mass) calculated from pixel positions
- **Bounding shape:** A geometric shape (circle, rectangle) drawn around a detected region to visualize its location and size

## 📝 Next Steps

- Save your tuned HSV values in your notebook — you'll need them again for future labs
- Push `color_tracking.py` to your GitHub repository (with co-authored commit!)
- Tomorrow: you'll extend this into shape detection, classifying objects by their geometry rather than color

## 📚 Resources

- [Color Detection Based Object Tracking — Instructables](https://www.instructables.com/Color-Detection-Based-Object-Tracking/)
- [OpenCV Contours Tutorial](https://docs.opencv.org/4.x/d4/d73/tutorial_py_contours_begin.html)
- [Image Moments — OpenCV Docs](https://docs.opencv.org/4.x/d8/d23/classcv_1_1Moments.html)
- [HSV Color Picker Tool (for reference values)](https://alloyui.com/examples/color-picker/hsv)

---
---

# 🤖 Day 6: Module A — Shape Detection with Contours

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to detect and classify objects by their **geometry** rather than their color — a technique that works even on plain-colored or multi-colored objects, and is far more resistant to lighting changes than color tracking. 🚀

By the end of this session you and your partner should be able to:
- Use Canny edge detection to find object boundaries
- Classify contours as triangles, rectangles, or circles based on vertex count
- Use the Hough Circle Transform to detect circular objects directly
- Compare the strengths and weaknesses of color tracking vs. shape detection

---

## 🖊️ Part 1: Edge Detection with Canny

Shape detection starts with finding edges — sharp transitions in brightness that outline objects.

```python
gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5, 5), 0)
edges = cv2.Canny(blurred, 50, 150)
```

The two numbers (`50, 150`) are the lower and upper thresholds — pixels above the upper threshold are definitely edges, pixels below the lower threshold are definitely not, and pixels in between are edges only if connected to a strong edge.

---

## 🔺 Part 2: Classifying Shapes by Vertex Count

Once we have contours, we can approximate each one as a simplified polygon and count its corners to classify the shape.

Create `~/cv_project/scripts/shape_detection.py`:

```python
import cv2
from picamera2 import Picamera2

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

def classify_shape(contour):
    perimeter = cv2.arcLength(contour, True)
    approx = cv2.approxPolyDP(contour, 0.04 * perimeter, True)
    vertices = len(approx)

    if vertices == 3:
        return "Triangle"
    elif vertices == 4:
        x, y, w, h = cv2.boundingRect(approx)
        aspect_ratio = w / float(h)
        return "Square" if 0.9 <= aspect_ratio <= 1.1 else "Rectangle"
    elif vertices == 5:
        return "Pentagon"
    elif vertices > 8:
        return "Circle"
    else:
        return f"{vertices}-gon"

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        blurred = cv2.GaussianBlur(gray, (5, 5), 0)
        _, thresh = cv2.threshold(blurred, 60, 255, cv2.THRESH_BINARY_INV)

        contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        for contour in contours:
            if cv2.contourArea(contour) < 500:
                continue

            shape_name = classify_shape(contour)
            M = cv2.moments(contour)
            if M["m00"] == 0:
                continue
            cx = int(M["m10"] / M["m00"])
            cy = int(M["m01"] / M["m00"])

            cv2.drawContours(frame, [contour], -1, (0, 255, 0), 2)
            cv2.putText(frame, shape_name, (cx - 40, cy),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 2)

        cv2.imshow('Shape Detection', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

> **Lighting tip:** This script uses simple thresholding (`cv2.threshold`), which works best with high-contrast objects on a plain, contrasting background (e.g., dark shapes on white paper). Uneven lighting will cause false detections — this is a great discussion point for your notebook reflection.

---

## ⭕ Part 3: Detecting Circles with Hough Transform

Vertex counting struggles with true circles (they have no vertices). The **Hough Circle Transform** is a specialized algorithm for detecting circles directly from edges.

```python
import cv2
import numpy as np
from picamera2 import Picamera2

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        blurred = cv2.medianBlur(gray, 5)

        circles = cv2.HoughCircles(
            blurred, cv2.HOUGH_GRADIENT, dp=1, minDist=50,
            param1=50, param2=30, minRadius=10, maxRadius=150
        )

        if circles is not None:
            circles = np.uint16(np.around(circles))
            for (x, y, r) in circles[0, :]:
                cv2.circle(frame, (x, y), r, (0, 255, 0), 2)
                cv2.circle(frame, (x, y), 2, (0, 0, 255), 3)

        cv2.imshow('Circle Detection', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

---

## 🧪 Part 4: Lab Challenge

Combine what you've learned: build a script that detects shapes on a table (coins, cut-out paper shapes, blocks) and **counts** how many of each shape type appear in the frame. Display running totals in the corner of the screen (e.g., "Triangles: 2, Circles: 3").

---

## 💻 Terms to Know

- **Canny edge detection:** An algorithm that identifies sharp intensity gradients in an image to outline object boundaries
- **Threshold:** A pixel-value cutoff used to convert a grayscale image into pure black-and-white (binary)
- **`cv2.approxPolyDP`:** A function that simplifies a contour into a polygon with fewer vertices, used for shape classification
- **Aspect ratio:** The ratio of width to height of a bounding box, used to distinguish squares from rectangles
- **Hough Circle Transform:** An algorithm specialized for detecting circular shapes in an image, independent of vertex-counting methods
- **Bounding rectangle:** The smallest upright rectangle that fully contains a contour

## 📝 Next Steps

- Push `shape_detection.py` to your GitHub repository
- Complete **Engineering Notebook Entry #3:** Include screenshots of both labs, note what lighting/background conditions caused failures, and compare color tracking (Day 5) vs. shape detection (today) — which would you choose for your final project and why?
- Tomorrow: you move into machine-learning-based object detection with TensorFlow Lite

## 📚 Resources

- [Object Tracking Based on Shape — Instructables](https://www.instructables.com/Object-Tracking-Based-on-Shape-Using-OpenCV-on-Bra/)
- [Canny Edge Detection — OpenCV Docs](https://docs.opencv.org/4.x/da/d22/tutorial_py_canny.html)
- [Hough Circle Transform — OpenCV Docs](https://docs.opencv.org/4.x/d4/d70/tutorial_hough_circle.html)
- [Contour Approximation Explained — LearnOpenCV](https://learnopencv.com/contour-detection-using-opencv-python-c/)

---
---

# 🤖 Day 7: Module B — Intro to Object Detection with TensorFlow Lite

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to move beyond hand-coded rules (color, shape) into **machine-learning-based object detection**. Today you'll run a pre-trained neural network that can recognize 90 categories of everyday objects — no training required. 🚀

By the end of this session you and your partner should be able to:
- Explain what TensorFlow Lite is and why it's suited to Raspberry Pi
- Install and run a pre-trained object detection model
- Interpret bounding boxes, class labels, and confidence scores
- Discuss the tradeoffs between hand-coded CV and machine-learning-based CV

---

## 🧠 Part 1: What Is TensorFlow Lite?

**TensorFlow Lite (TFLite)** is a lightweight version of Google's TensorFlow machine learning framework, specifically optimized to run on low-power devices like the Raspberry Pi. Unlike the color and shape techniques from Days 5–6 (which use rules you wrote), TFLite uses a **pre-trained neural network** that has already learned to recognize objects from millions of example images.

### The Model We'll Use

We'll use a pre-trained **SSD MobileNet** model trained on the **COCO dataset** — a public dataset of everyday images labeled with 90 object categories (person, car, dog, bird, chair, bottle, and more).

- **SSD (Single Shot Detector):** finds all objects in an image in a single pass, giving both a location (bounding box) and a class label for each
- **MobileNet:** a neural network architecture designed to be fast and small enough to run on mobile/embedded devices

---

## ⚙️ Part 2: Install & Download the Model

> **Compatibility note:** `tflite-runtime` pip wheels are only published for specific Python versions. If `pip install tflite-runtime` fails with "no matching distribution," check your Python version with `python3 --version` — you may need to install TensorFlow's full package instead (`pip install tensorflow`) and import `tensorflow.lite` rather than the standalone `tflite_runtime` package. Ask your instructor if you hit this error.

```bash
source ~/cv_env/bin/activate
pip install tflite-runtime
# If that fails:
# pip install tensorflow
```

Download the pre-trained model and label file:

```bash
cd ~/cv_project/models
wget https://storage.googleapis.com/download.tensorflow.org/models/tflite/coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip
unzip coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip
```

This gives you `detect.tflite` (the model) and `labelmap.txt` (the 90 class names).

---

## 📦 Part 3: Running Object Detection

Create `~/cv_project/scripts/object_detect.py`:

```python
import cv2
import numpy as np
from picamera2 import Picamera2

try:
    from tflite_runtime.interpreter import Interpreter
except ImportError:
    from tensorflow.lite.python.interpreter import Interpreter

MODEL_PATH = '/home/pi/cv_project/models/detect.tflite'
LABEL_PATH = '/home/pi/cv_project/models/labelmap.txt'
CONFIDENCE_THRESHOLD = 0.5

with open(LABEL_PATH, 'r') as f:
    labels = [line.strip() for line in f.readlines()]

interpreter = Interpreter(model_path=MODEL_PATH)
interpreter.allocate_tensors()
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()
input_height = input_details[0]['shape'][1]
input_width = input_details[0]['shape'][2]

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        frame_h, frame_w, _ = frame.shape

        resized = cv2.resize(frame, (input_width, input_height))
        input_data = np.expand_dims(resized, axis=0)

        interpreter.set_tensor(input_details[0]['index'], input_data)
        interpreter.invoke()

        boxes = interpreter.get_tensor(output_details[0]['index'])[0]
        classes = interpreter.get_tensor(output_details[1]['index'])[0]
        scores = interpreter.get_tensor(output_details[2]['index'])[0]

        for i in range(len(scores)):
            if scores[i] > CONFIDENCE_THRESHOLD:
                ymin, xmin, ymax, xmax = boxes[i]
                x1, y1 = int(xmin * frame_w), int(ymin * frame_h)
                x2, y2 = int(xmax * frame_w), int(ymax * frame_h)

                class_id = int(classes[i])
                label = labels[class_id] if class_id < len(labels) else "Unknown"

                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
                cv2.putText(frame, f"{label}: {scores[i]:.2f}", (x1, y1 - 10),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

        cv2.imshow('Object Detection', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### What Do the Outputs Mean?

- **Bounding box:** the rectangle location of a detected object, given as normalized coordinates (0–1) that must be scaled to your frame size
- **Class ID:** a number corresponding to a label in `labelmap.txt` (e.g., 0 = person, 17 = cat)
- **Confidence score:** how sure the model is about its prediction (0.0–1.0); we discard anything below our threshold (0.5) to reduce false positives

---

## 🗣️ Part 4: Class Discussion

- What objects around the room does the model correctly identify? What does it miss or misidentify?
- How does changing `CONFIDENCE_THRESHOLD` change the results? Try 0.3 vs. 0.7.
- Unlike your Day 5–6 scripts, you didn't write any rules about what a "dog" or "chair" looks like — where did that knowledge come from?

---

## 💻 Terms to Know

- **TensorFlow Lite (TFLite):** A lightweight machine learning inference framework optimized for mobile and embedded devices
- **SSD (Single Shot Detector):** A neural network architecture that detects multiple objects and their locations in a single pass over an image
- **MobileNet:** A neural network architecture designed for efficiency on low-power hardware
- **COCO dataset:** Common Objects in Context — a large public dataset with ~200,000 labeled images across 90 object categories, commonly used to pre-train detection models
- **Inference:** The process of running new data through an already-trained machine learning model to get a prediction
- **Bounding box:** A rectangle marking the location of a detected object in an image
- **Confidence score:** A model's estimated probability (0–1) that a detection is correct
- **Confidence threshold:** A cutoff value used to discard low-confidence detections and reduce false positives

## 📝 Next Steps

- Push `object_detect.py` and your model files to GitHub (note: large model files may need `.gitignore` or Git LFS — ask your instructor)
- Tomorrow: you'll filter and log detections to a CSV file, building toward your final project's data-logging needs

## 📚 Resources

- [TensorFlow Lite Python Guide](https://www.tensorflow.org/lite/guide/python)
- [TensorFlow Lite Object Detection on Raspberry Pi — TensorFlow Examples](https://github.com/tensorflow/examples/blob/master/lite/examples/object_detection/raspberry_pi/README.md)
- [How to Perform Object Detection with TFLite on Raspberry Pi — Digi-Key](https://www.digikey.com/en/maker/projects/how-to-perform-object-detection-with-tensorflow-lite-on-raspberry-pi/b929e1519c7c43d5b2c6f89984883588)
- [COCO Dataset Overview](https://cocodataset.org/#home)

---
---

# 🤖 Day 8: Module B — Filtering Detections & Logging to CSV

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to turn raw object detections into structured, useful data. Detecting an object is only half the job — a real system needs to log *what* was seen and *when*, which is exactly what your final project will need to do. 🚀

By the end of this session you and your partner should be able to:
- Filter detections to only the object classes relevant to your project
- Log detection events to a timestamped CSV file
- Avoid duplicate/repeated logging of the same continuous detection
- Analyze your own logged data for accuracy patterns

---

## 🔍 Part 1: Filtering by Class Label

Most projects only care about one or two object classes (e.g., "bird" for a feeder, "person" for a security system), not all 90 COCO categories. Filter detections down to just what matters:

```python
TARGET_CLASSES = ['bird', 'cat', 'dog']  # customize for your project

# Inside your detection loop, after computing 'label':
if label not in TARGET_CLASSES:
    continue  # skip this detection entirely
```

---

## 📝 Part 2: Logging Detections to CSV

Create `~/cv_project/scripts/detection_logger.py` by extending your Day 7 script with logging:

```python
import cv2
import numpy as np
import csv
import time
from datetime import datetime
from picamera2 import Picamera2

try:
    from tflite_runtime.interpreter import Interpreter
except ImportError:
    from tensorflow.lite.python.interpreter import Interpreter

MODEL_PATH = '/home/pi/cv_project/models/detect.tflite'
LABEL_PATH = '/home/pi/cv_project/models/labelmap.txt'
LOG_PATH = '/home/pi/cv_project/data/detections.csv'
CONFIDENCE_THRESHOLD = 0.5
TARGET_CLASSES = ['bird', 'cat', 'dog', 'person']  # customize this
COOLDOWN_SECONDS = 5  # prevent duplicate logs of the same continuous detection

with open(LABEL_PATH, 'r') as f:
    labels = [line.strip() for line in f.readlines()]

interpreter = Interpreter(model_path=MODEL_PATH)
interpreter.allocate_tensors()
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()
input_height = input_details[0]['shape'][1]
input_width = input_details[0]['shape'][2]

# Initialize CSV file with headers if it doesn't exist
try:
    with open(LOG_PATH, 'x', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['timestamp', 'label', 'confidence'])
except FileExistsError:
    pass

last_logged = {}  # tracks last log time per class, for cooldown

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        frame_h, frame_w, _ = frame.shape

        resized = cv2.resize(frame, (input_width, input_height))
        input_data = np.expand_dims(resized, axis=0)

        interpreter.set_tensor(input_details[0]['index'], input_data)
        interpreter.invoke()

        boxes = interpreter.get_tensor(output_details[0]['index'])[0]
        classes = interpreter.get_tensor(output_details[1]['index'])[0]
        scores = interpreter.get_tensor(output_details[2]['index'])[0]

        for i in range(len(scores)):
            if scores[i] < CONFIDENCE_THRESHOLD:
                continue

            class_id = int(classes[i])
            label = labels[class_id] if class_id < len(labels) else "Unknown"

            if label not in TARGET_CLASSES:
                continue

            ymin, xmin, ymax, xmax = boxes[i]
            x1, y1 = int(xmin * frame_w), int(ymin * frame_h)
            x2, y2 = int(xmax * frame_w), int(ymax * frame_h)

            cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
            cv2.putText(frame, f"{label}: {scores[i]:.2f}", (x1, y1 - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

            now = time.time()
            if label not in last_logged or (now - last_logged[label]) > COOLDOWN_SECONDS:
                last_logged[label] = now
                timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
                with open(LOG_PATH, 'a', newline='') as f:
                    writer = csv.writer(f)
                    writer.writerow([timestamp, label, f"{scores[i]:.2f}"])
                print(f"Logged: {timestamp} - {label} ({scores[i]:.2f})")

        cv2.imshow('Detection Logger', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Why the Cooldown Timer?

Without it, a detected object sitting still for 10 seconds would generate hundreds of log entries (once per frame). The `COOLDOWN_SECONDS` logic ensures each class is only logged once per cooldown window — turning noisy frame-by-frame detections into meaningful discrete "visit" events.

---

## 📊 Part 3: Lab Activity — Structured Testing

Run your logger for 10 minutes while different objects/people pass in front of the camera. Then answer in your notebook:

| Trial | Object Shown | Distance | Lighting | Detected? | Confidence | Notes |
|---|---|---|---|---|---|---|
| 1 | | | | | | |
| 2 | | | | | | |
| ... | | | | | | |

Aim for **at least 10 trials** varying distance and lighting.

---

## 💻 Terms to Know

- **CSV (Comma-Separated Values):** A plain-text file format for storing tabular data, easily opened in Excel/Sheets or read programmatically
- **Cooldown timer:** A logic pattern that prevents repeated logging of the same ongoing event within a set time window
- **Class filtering:** Restricting a model's output to only the object categories relevant to your application
- **False positive:** A detection the model reports that is not actually correct
- **False negative:** A real object that the model fails to detect

## 📝 Next Steps

- Push `detection_logger.py` and your generated `detections.csv` file to GitHub
- Complete **Engineering Notebook Entry #4:** Include your 10-trial results table above, and write a paragraph analyzing where detection failed and why (lighting? distance? occlusion?)
- Tomorrow: you begin Module C — pose estimation with MediaPipe

## 📚 Resources

- [Python CSV Module Documentation](https://docs.python.org/3/library/csv.html)
- [TensorFlow Lite Wildlife Camera Example — SP Tech](https://spltech.co.uk/how-to-run-object-detection-with-tensorflow-lite-and-a-raspberry-pi-to-build-a-wildlife-camera/)
- [Precision vs. Recall Explained — Google ML Crash Course](https://developers.google.com/machine-learning/crash-course/classification/precision-and-recall)

---
---

# 🤖 Day 9: Module C — Pose Estimation Basics with MediaPipe

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to detect and track the human body using pose estimation — a technique with applications from fitness coaching to safety monitoring. Today you'll get a skeleton overlay running live on your own body. 🚀

By the end of this session you and your partner should be able to:
- Explain what MediaPipe is and how pose landmark detection works
- Install MediaPipe and resolve common Python version compatibility issues
- Run live pose detection and visualize the 33-point body skeleton
- Access individual landmark coordinates in code

---

## ⚠️ Part 1: Important — Python Version Compatibility

**Read this before installing anything today.** MediaPipe's pip package does not currently support Python 3.13, which ships with newer Raspberry Pi OS (Trixie-based) releases. If your Pi is running Python 3.13 (check with `python3 --version`), a plain `pip install mediapipe` will fail with "no matching distribution."

**Check your Python version first:**

```bash
python3 --version
```

**If you have Python 3.13:** talk to your instructor — the class may need to either use a Raspberry Pi OS Bookworm image (Python 3.11, fully supported) for these two lessons, or your instructor may have a pre-tested workaround ready. Do not spend lab time fighting this alone — flag it immediately.

**If you have Python 3.11 or 3.12:** you're good to proceed normally.

```bash
source ~/cv_env/bin/activate
pip install mediapipe
python3 -c "import mediapipe as mp; print(mp.__version__)"
```

---

## 🦴 Part 2: What Is Pose Estimation?

MediaPipe's Pose solution detects **33 body landmarks** — key points like shoulders, elbows, wrists, hips, knees, and ankles — from a single camera frame, without any special hardware like depth sensors.

Each landmark includes:
- **x, y:** normalized position (0.0–1.0) within the frame
- **z:** relative depth (smaller = closer to camera)
- **visibility:** confidence that the landmark is actually visible (not occluded)

---

## 🧍 Part 3: Running Live Pose Detection

Create `~/cv_project/scripts/pose_basic.py`:

```python
import cv2
import mediapipe as mp
from picamera2 import Picamera2

mp_pose = mp.solutions.pose
mp_drawing = mp.solutions.drawing_utils
pose = mp_pose.Pose(min_detection_confidence=0.5, min_tracking_confidence=0.5)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        results = pose.process(rgb_frame)

        if results.pose_landmarks:
            mp_drawing.draw_landmarks(
                frame, results.pose_landmarks, mp_pose.POSE_CONNECTIONS
            )

        cv2.imshow('Pose Estimation', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

> **Note:** MediaPipe expects **RGB** images, while OpenCV/Picamera2 frames are in **BGR**. That's why we convert with `cv2.COLOR_BGR2RGB` before passing to `pose.process()`, but keep drawing on the original BGR `frame` for correct display colors.

Stand back far enough that your whole body is visible in the frame and confirm the skeleton overlay tracks your movement.

---

## 🔢 Part 4: Accessing Individual Landmarks

Each of the 33 landmarks has an index number. A few useful ones:

| Landmark | Index |
|---|---|
| Nose | 0 |
| Left shoulder | 11 |
| Right shoulder | 12 |
| Left elbow | 13 |
| Right elbow | 14 |
| Left wrist | 15 |
| Right wrist | 16 |
| Left hip | 23 |
| Right hip | 24 |
| Left knee | 25 |
| Right knee | 26 |

```python
if results.pose_landmarks:
    landmarks = results.pose_landmarks.landmark
    right_wrist = landmarks[16]
    print(f"Right wrist position: x={right_wrist.x:.2f}, y={right_wrist.y:.2f}")
```

### Lab Activity

Modify your script to print the coordinates of your **right wrist** and **right shoulder** to the terminal on every frame. Raise your hand above your head and watch how the y-value changes (remember: in image coordinates, y increases *downward*).

---

## 💻 Terms to Know

- **MediaPipe:** Google's open-source framework for building real-time perception pipelines, including pose, face, and hand tracking
- **Landmark:** A specific tracked point on a detected object (e.g., a body joint or facial feature)
- **Pose estimation:** The task of identifying the spatial positions of key body joints from an image or video
- **Normalized coordinates:** Position values scaled between 0.0 and 1.0 relative to image width/height, independent of resolution
- **Visibility score:** A confidence value indicating whether a landmark is actually visible or likely occluded
- **RGB vs. BGR:** Two different color channel orderings; MediaPipe expects RGB while OpenCV defaults to BGR

## 📝 Next Steps

- Push `pose_basic.py` to your GitHub repository
- Experiment with different landmark indices — try printing the position of your nose or ankles
- Tomorrow: you'll calculate joint angles and build a rep-counting system

## 📚 Resources

- [MediaPipe Pose Documentation](https://developers.google.com/mediapipe/solutions/vision/pose_landmarker)
- [Pose Estimation & Face Landmark Guide — Core Electronics](https://core-electronics.com.au/guides/pose-and-face-landmark-raspberry-pi/)
- [MediaPipe Setup Guide for Python](https://developers.google.com/edge/mediapipe/solutions/setup_python)

---
---

# 🤖 Day 10: Module C — Joint Angles & Gesture Logic

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to turn raw pose landmarks into meaningful measurements. You'll calculate joint angles and use them to build a working exercise rep counter — a real application of pose estimation. 🚀

By the end of this session you and your partner should be able to:
- Calculate the angle between three body landmarks using vector math
- Build a state machine that counts repetitions based on angle thresholds
- Display live feedback (angle values, rep count) on the video feed
- Test your rep counter and evaluate its accuracy

---

## 📐 Part 1: Calculating Joint Angles

To measure something like "is the elbow bent?" we calculate the angle formed by three points: the joint itself and the two segments connected to it (e.g., shoulder–elbow–wrist for elbow angle).

```python
import math

def calculate_angle(a, b, c):
    """Calculate the angle at point b, formed by points a-b-c."""
    a = [a.x, a.y]
    b = [b.x, b.y]
    c = [c.x, c.y]

    radians = math.atan2(c[1] - b[1], c[0] - b[0]) - math.atan2(a[1] - b[1], a[0] - b[0])
    angle = abs(radians * 180.0 / math.pi)

    if angle > 180.0:
        angle = 360 - angle

    return angle
```

This function takes three landmark objects and returns the angle (in degrees) at the middle point.

---

## 💪 Part 2: Building a Rep Counter

Create `~/cv_project/scripts/pose_angles.py`:

```python
import cv2
import mediapipe as mp
import math
from picamera2 import Picamera2

mp_pose = mp.solutions.pose
mp_drawing = mp.solutions.drawing_utils
pose = mp_pose.Pose(min_detection_confidence=0.5, min_tracking_confidence=0.5)

def calculate_angle(a, b, c):
    a = [a.x, a.y]
    b = [b.x, b.y]
    c = [c.x, c.y]
    radians = math.atan2(c[1] - b[1], c[0] - b[0]) - math.atan2(a[1] - b[1], a[0] - b[0])
    angle = abs(radians * 180.0 / math.pi)
    if angle > 180.0:
        angle = 360 - angle
    return angle

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

rep_count = 0
stage = None  # "up" or "down"

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        results = pose.process(rgb_frame)

        if results.pose_landmarks:
            landmarks = results.pose_landmarks.landmark

            shoulder = landmarks[mp_pose.PoseLandmark.LEFT_SHOULDER.value]
            elbow = landmarks[mp_pose.PoseLandmark.LEFT_ELBOW.value]
            wrist = landmarks[mp_pose.PoseLandmark.LEFT_WRIST.value]

            angle = calculate_angle(shoulder, elbow, wrist)

            # Rep counting logic for a bicep curl
            if angle > 160:
                stage = "down"
            if angle < 40 and stage == "down":
                stage = "up"
                rep_count += 1

            mp_drawing.draw_landmarks(frame, results.pose_landmarks, mp_pose.POSE_CONNECTIONS)

            cv2.putText(frame, f"Angle: {int(angle)}", (10, 30),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.8, (255, 255, 255), 2)
            cv2.putText(frame, f"Reps: {rep_count}", (10, 70),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2)

        cv2.imshow('Rep Counter', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### How the State Machine Works

- **`stage = "down"`** when the arm is extended (angle > 160°)
- **`stage = "up"`** — and the rep counts — only when the arm curls (angle < 40°) *after* having been in the "down" state
- This prevents double-counting small movements or counting a rep that never fully extended

---

## 🧪 Part 3: Lab Challenge — Test & Tune

- Test your rep counter with a partner performing 10 real bicep curls. How many did it count correctly?
- Adjust the angle thresholds (160°/40°) if your counter is over- or under-counting
- Try switching to a different exercise (e.g., squats using hip-knee-ankle angles) — which landmarks would you need?

| Trial | Actual Reps | Counted Reps | Notes |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |

---

## 💻 Terms to Know

- **Joint angle:** The angle formed at a body joint by the two connected limb segments, calculated using vector math
- **`atan2`:** A trigonometric function that calculates the angle of a vector, accounting for all four quadrants (more robust than plain `atan`)
- **State machine:** A programming pattern where the system's behavior depends on a current "state" (e.g., "up" or "down") that transitions based on conditions
- **Threshold-based logic:** Decision-making based on comparing a measured value against a fixed cutoff point
- **Rep counter:** A system that counts discrete repetitions of a movement based on detecting transitions between two positions

## 📝 Next Steps

- Push `pose_angles.py` to your GitHub repository
- Complete **Engineering Notebook Entry #5:** Diagram the landmarks you used with their index numbers, explain your angle calculation, and include your rep counter test results table
- Tomorrow: you begin Module D — face detection

## 📚 Resources

- [MediaPipe Pose Landmarker Guide](https://developers.google.com/mediapipe/solutions/vision/pose_landmarker)
- [Pose Estimation Angle Calculation — freeCodeCamp](https://www.freecodecamp.org/news/how-to-use-mediapipe-to-count-reps/)
- [Understanding atan2 — Math Stack Exchange](https://math.stackexchange.com/questions/1596513/why-is-atan2-preferred-over-arctan-in-navigation)

---
---

# 🤖 Day 11: Module D — Face Detection with Haar Cascades & Face Mesh

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to detect human faces using two different techniques — a fast classical method (Haar Cascades) and a detailed modern method (MediaPipe Face Mesh) — and understand when to use each. 🚀

By the end of this session you and your partner should be able to:
- Detect and count faces using OpenCV's Haar Cascade classifier
- Detect detailed facial landmarks using MediaPipe Face Mesh
- Explain the tradeoffs between the two approaches
- Articulate at least one ethical concern related to facial recognition technology

---

## 👤 Part 1: Face Detection with Haar Cascades

Haar Cascades are a fast, classical (pre-deep-learning) face detection method built into OpenCV. They work by scanning the image at multiple scales looking for facial patterns (eyes, nose bridge, etc.).

Create `~/cv_project/scripts/face_detect.py`:

```python
import cv2
from picamera2 import Picamera2

face_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + 'haarcascade_frontalface_default.xml'
)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

        faces = face_cascade.detectMultiScale(
            gray, scaleFactor=1.1, minNeighbors=5, minSize=(60, 60)
        )

        for (x, y, w, h) in faces:
            cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)

        cv2.putText(frame, f"Faces: {len(faces)}", (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.8, (255, 255, 255), 2)

        cv2.imshow('Face Detection', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Tuning Parameters

- **`scaleFactor`:** how much the image is scaled down at each detection pass; smaller = more thorough but slower
- **`minNeighbors`:** how many overlapping detections are required to confirm a real face; higher = fewer false positives but may miss real faces
- **`minSize`:** the smallest face size (in pixels) to detect; filters out tiny false positives

---

## 🕸️ Part 2: Detailed Landmarks with MediaPipe Face Mesh

Haar Cascades only give you a bounding box. **MediaPipe Face Mesh** maps **468 individual points** across the face — useful for expression analysis, filters, or precise measurements.

> **Reminder:** This requires MediaPipe, so confirm your Python version compatibility from Day 9 before proceeding.

```python
import cv2
import mediapipe as mp
from picamera2 import Picamera2

mp_face_mesh = mp.solutions.face_mesh
mp_drawing = mp.solutions.drawing_utils
mp_drawing_styles = mp.solutions.drawing_styles
face_mesh = mp_face_mesh.FaceMesh(
    max_num_faces=2, min_detection_confidence=0.5, min_tracking_confidence=0.5
)

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        results = face_mesh.process(rgb_frame)

        if results.multi_face_landmarks:
            for face_landmarks in results.multi_face_landmarks:
                mp_drawing.draw_landmarks(
                    image=frame,
                    landmark_list=face_landmarks,
                    connections=mp_face_mesh.FACEMESH_TESSELATION,
                    landmark_drawing_spec=None,
                    connection_drawing_spec=mp_drawing_styles.get_default_face_mesh_tesselation_style()
                )

        cv2.imshow('Face Mesh', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

---

## ⚖️ Part 3: Ethics Discussion

Facial detection and recognition technology raises real ethical questions your class should discuss before moving to full recognition tomorrow:

- **Consent:** Should people be notified when a camera is capturing and analyzing their face?
- **Bias:** Facial recognition systems have historically performed worse on certain skin tones and genders due to biased training data — how might that affect a system you design?
- **Surveillance:** Where's the line between a helpful security system and invasive monitoring?
- **Data storage:** If your system saves face images or encodings, who has access to that data, and how long should it be kept?

**Discussion prompt for your notebook:** If you build a face recognition system for your final project, what safeguards would you put in place to use this technology responsibly?

---

## 💻 Terms to Know

- **Haar Cascade:** A classical machine learning method for object detection using trained pattern classifiers; fast but less accurate than deep learning approaches
- **Face Mesh:** MediaPipe's dense facial landmark model, mapping 468 points across the face
- **`scaleFactor` / `minNeighbors`:** Tuning parameters that control the sensitivity and false-positive rate of Haar Cascade detection
- **Algorithmic bias:** Systematic errors in a model's predictions caused by unrepresentative or skewed training data
- **Facial landmark:** A specific tracked point on a face (e.g., corner of an eye, tip of the nose)

## 📝 Next Steps

- Push `face_detect.py` and your Face Mesh script to GitHub
- Tomorrow: you'll move from detecting *that* a face is present to recognizing *whose* face it is

## 📚 Resources

- [Haar Cascade Face Detection — OpenCV Docs](https://docs.opencv.org/4.x/db/d28/tutorial_cascade_classifier.html)
- [MediaPipe Face Mesh Documentation](https://developers.google.com/mediapipe/solutions/vision/face_landmarker)
- [Pan/Tilt Face Tracking with Raspberry Pi and OpenCV — PyImageSearch](https://pyimagesearch.com/2019/04/01/pan-tilt-face-tracking-with-a-raspberry-pi-and-opencv/)
- [Gender Shades: Bias in Facial Recognition — MIT Media Lab](https://www.media.mit.edu/projects/gender-shades/overview/)

---
---

# 🤖 Day 12: Module D — Face Recognition

## 🤓 Overview and Learning Outcomes

The goal of this lab assignment is to move from detecting *a* face to recognizing *whose* face it is — building a known-vs-unknown classifier, the same core technology behind phone face unlock and security systems. 🚀

By the end of this session you and your partner should be able to:
- Encode known faces from reference photos
- Compare live camera faces against known encodings
- Build a simple known/unknown classification system
- Evaluate recognition accuracy across different conditions

---

## 📷 Part 1: Install face_recognition

```bash
source ~/cv_env/bin/activate
sudo apt install -y cmake libboost-all-dev
pip install face_recognition
```

> **Heads up:** `face_recognition` depends on `dlib`, which compiles from source and can take **20–30+ minutes** to install on a Raspberry Pi 4. Start this install at the very beginning of class.

---

## 🖼️ Part 2: Collecting Reference Photos

Each partner should capture 1–2 clear, well-lit photos of their own face, facing the camera directly.

```bash
rpicam-still -o ~/cv_project/images/known_faces/partner_a.jpg
rpicam-still -o ~/cv_project/images/known_faces/partner_b.jpg
```

Name each file after the person it contains — the filename becomes the label your system displays.

---

## 🧠 Part 3: Building the Recognition System

Create `~/cv_project/scripts/face_recognize.py`:

```python
import cv2
import face_recognition
import os
from picamera2 import Picamera2

KNOWN_FACES_DIR = '/home/pi/cv_project/images/known_faces'

known_encodings = []
known_names = []

for filename in os.listdir(KNOWN_FACES_DIR):
    if filename.lower().endswith(('.jpg', '.jpeg', '.png')):
        path = os.path.join(KNOWN_FACES_DIR, filename)
        image = face_recognition.load_image_file(path)
        encodings = face_recognition.face_encodings(image)
        if encodings:
            known_encodings.append(encodings[0])
            known_names.append(os.path.splitext(filename)[0])

print(f"Loaded {len(known_names)} known faces: {known_names}")

picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": 'XRGB8888', "size": (640, 480)}
))
picam2.start()

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

        # Resize down for faster processing, then scale results back up
        small_frame = cv2.resize(frame, (0, 0), fx=0.5, fy=0.5)
        rgb_small = cv2.cvtColor(small_frame, cv2.COLOR_BGR2RGB)

        face_locations = face_recognition.face_locations(rgb_small)
        face_encodings = face_recognition.face_encodings(rgb_small, face_locations)

        for (top, right, bottom, left), face_encoding in zip(face_locations, face_encodings):
            matches = face_recognition.compare_faces(known_encodings, face_encoding, tolerance=0.6)
            name = "Unknown"

            if True in matches:
                match_index = matches.index(True)
                name = known_names[match_index]

            # Scale coordinates back up to original frame size
            top, right, bottom, left = top * 2, right * 2, bottom * 2, left * 2

            color = (0, 255, 0) if name != "Unknown" else (0, 0, 255)
            cv2.rectangle(frame, (left, top), (right, bottom), color, 2)
            cv2.putText(frame, name, (left, top - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, color, 2)

        cv2.imshow('Face Recognition', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Interrupted by user")

finally:
    picam2.stop()
    cv2.destroyAllWindows()
```

### Key Concept: Tolerance

The `tolerance=0.6` parameter controls how strict the matching is. Lower values (e.g., 0.4) require a closer match (fewer false positives, more false negatives/missed matches); higher values (e.g., 0.7) are more lenient (more false positives).

---

## 🧪 Part 4: Lab Activity — Accuracy Testing

Test your system under varied conditions and record results:

| Trial | Person | Lighting | Angle | Result (Correct/Wrong/Unknown) |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| ... | | | | |

Try at least 10 trials, varying lighting (bright/dim), angle (straight-on/side profile), and distance.

---

## ⚖️ Part 5: Ethics Reflection

Building on yesterday's discussion, write a reflection paragraph addressing:

- What would happen if your system misidentified an unknown person as a known one? What are the consequences depending on the application (unlocking a door vs. a harmless greeting)?
- How would you communicate to users that a facial recognition system is monitoring an area?
- If this were deployed for real, what data privacy safeguards would you build in?

---

## 💻 Terms to Know

- **Face encoding:** A numerical representation (128-dimensional vector) of a face's unique features, used for comparison rather than storing the actual image
- **`face_recognition` library:** A Python library built on `dlib` that provides face detection, encoding, and comparison functions
- **Tolerance:** A threshold controlling how similar two face encodings must be to be considered a match
- **Known/unknown classifier:** A system that compares an input against a set of known reference examples and either matches it or labels it "unknown"
- **False accept / false reject:** Errors where an unauthorized face is incorrectly matched (false accept) or an authorized face is incorrectly rejected (false reject)

## 📝 Next Steps

- Push `face_recognize.py` to your GitHub repository
- Complete **Engineering Notebook Entry #6:** Include your 10-trial accuracy table and your ethics reflection paragraph
- This completes Phase 2! Tomorrow you move into Phase 3 — formal design planning for your final project using everything you've learned in Modules A–D

## 📚 Resources

- [face_recognition Library Documentation](https://face-recognition.readthedocs.io/en/latest/readme.html)
- [dlib Library Overview](http://dlib.net/)
- [How to Build a Face-Tracking System with Raspberry Pi and OpenCV](https://nguyenrobot.medium.com/how-to-build-a-face-tracking-with-raspberry-and-opencv-d449cd1ac282)
- [Facial Recognition Ethics — Brookings Institution](https://www.brookings.edu/articles/facial-recognition-technology-and-its-impact-on-privacy/)

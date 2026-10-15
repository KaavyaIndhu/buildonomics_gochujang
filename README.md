# Kine-Sense

**An ambient Indian Sign Language (ISL) communication and wellness system for elderly individuals living alone.**

Kine-Sense connects an elder-facing mirror interface with a family-facing dashboard. It uses webcam-based sign recognition to help users communicate needs, relay alerts to family members, and support remote check-ins.

> **Project status:** The repository includes the Phase 1 communication flow and additional wellness-monitoring components. See the limitations section for known constraints.

## Key Features

- **Webcam-based sign recognition:** Detects hand landmarks and classifies a defined vocabulary of ISL signs.
- **Sign-to-sentence composition:** Combines confirmed signs into a message that can be sent to family members.
- **Family dashboard:** Receives communication alerts and supports check-ins back to the mirror.
- **Wellness monitoring:** Includes components for fall detection, gait and posture metrics, and wellness trends.
- **Real-time communication:** Uses Socket.IO to relay messages between the mirror, dashboard, and server.
- **Client-side inference:** Runs the exported sign classifier in the browser rather than training a model on the user's device.

## Tech Stack

| Component | Technologies |
|---|---|
| Mirror interface | React, Vite, Tailwind CSS |
| Family dashboard | React, Vite, Tailwind CSS |
| Real-time server | Node.js, Socket.IO |
| Landmark extraction | MediaPipe |
| Model training | Python, scikit-learn |
| Model inference | JavaScript |
| Data processing | Python, JSON |

## Repository Structure

```text
buildonomics_gochujang/
├── mirror/       # Elder-facing camera and communication UI
├── dashboard/   # Family-facing alerts and wellness UI
├── server/       # Real-time message relay and authentication
├── ml/           # Landmark extraction and classifier training
├── README.md
├── CLAUDE.md
└── Ghar-Sanket-PRD.md
```

## Architecture

1. The mirror captures webcam frames and extracts hand landmarks using MediaPipe.
2. The landmarks are normalized into a fixed-length numerical representation.
3. A JavaScript classifier predicts the sign using weights exported from the trained Python model.
4. A confirmation mechanism smooths predictions across recent frames.
5. Confirmed signs are composed into a sentence.
6. Socket.IO relays the resulting message to the family dashboard.
7. The dashboard can send check-in messages back to the mirror.

The server relays messages between the interfaces. The README describes the communication relay as non-persistent; verify the current server implementation before making broader claims about data storage.

## Machine Learning Pipeline

### 1. Dataset preparation

The training pipeline uses static sign images for a vocabulary of 19 labels. The project documentation notes that the original plan to use video sequences changed when suitable video data was unavailable.

### 2. Landmark extraction

`ml/src/extract_landmarks.py` uses MediaPipe's HandLandmarker to extract 21 three-dimensional landmarks per hand.

### 3. Normalization

`ml/src/normalize.py` recenters landmarks on the wrist and scales them using a hand reference distance. The left- and right-hand representations are combined into a 126-value vector.

### 4. Classifier training

`ml/src/train_classifier.py` trains a scikit-learn pipeline using `StandardScaler` and an `MLPClassifier` with hidden layers of 64 and 32 neurons.

The project reports **97.2% cross-validated accuracy**. This result should be interpreted cautiously because the dataset has class imbalance and some labels have very few training examples.

### 5. Model export and inference

`ml/src/export_model.py` exports the model parameters to JSON. The browser loads these parameters and runs a JavaScript forward pass for inference.

Training occurs offline; the browser uses the exported model.

## Sign Vocabulary

The documented vocabulary contains 19 labels:

`AFRAID`, `AGREE`, `ASSISTANCE`, `BAD`, `DOCTOR`, `GOOD MORNING`, `HOME`, `HOW ARE YOU`, `HUNGRY`, `I NEED HELP`, `PAIN`, `PROBLEM`, `SICK`, `STAND`, `STOP`, `THIRSTY`, `UNDERSTAND`, `WARN`, and `YOU`.

## Real-Time Communication

The server uses Socket.IO to relay communication between the two interfaces:

- Mirror to dashboard: sign-related alerts with timestamps and confidence information.
- Dashboard to mirror: text-based check-in messages.

The server also contains authentication and database modules. Review the current implementation and configuration before deploying the application or describing its production readiness.

## Running Locally

### Prerequisites

- Node.js and npm
- Python and the dependencies listed in `ml/requirements.txt` if working on the ML pipeline
- Webcam access for browser-based sign recognition

### Start the server

```bash
cd server
npm install
npm run dev
```

### Start the mirror

In a separate terminal:

```bash
cd mirror
npm install
npm run dev
```

### Start the dashboard

In another terminal:

```bash
cd dashboard
npm install
npm run dev
```

Open the dashboard's local Vite URL in your browser. The repository documentation uses port `5174` for the dashboard, `5173` for the mirror, and `4000` for the server. Confirm the actual ports in the current Vite configuration before running the application.

For the intended user flow and additional implementation context, see `CLAUDE.md` and `Ghar-Sanket-PRD.md`.

## Known Limitations

- Some vocabulary labels have very few training images, which can cause misclassification.
- Signs that depend on movement may be difficult to recognize using a classifier trained on static handshapes.
- The classifier does not currently have a dedicated “no sign” class, so it may produce false-positive predictions when nobody is signing.
- The reported cross-validation result may not reflect real-world performance on new users, lighting conditions, or camera angles.
- Wellness alerts and fall-detection components require further testing before they should be relied on for safety-critical use.

## Privacy and Safety Considerations

The documented inference pipeline runs in the browser, and the project states that video is not stored or transmitted during sign recognition. Verify the complete data flow, including logging, server messages, authentication, and wellness processing, before making privacy guarantees.

Kine-Sense is a prototype intended to support communication and wellness awareness. It should not be treated as a substitute for emergency services or professional care.

## My Contributions

This was a collaborative hackathon project. My individual contributions will be documented here based on the components and tasks I personally implemented.

- **Implementation:** To be specified.
- **Technical decisions and problem-solving:** To be specified.
- **Integration and testing:** To be specified.

The project includes work by multiple contributors. See the original team repository for the broader project history.

## Acknowledgements

Developed collaboratively as a hackathon project. Credit belongs to the respective contributors for their work on the codebase and supporting materials.

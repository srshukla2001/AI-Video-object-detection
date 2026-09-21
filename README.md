```markdown
# Ask the Footage: Colab AI Server

This repository contains a Google Colab notebook that functions as a lightweight API server for video content analysis. It leverages a free Colab GPU to process uploaded videos against a text prompt, identifying and extracting relevant frames.

## Features

- **Video Analysis**: Takes an uploaded video file and a natural language text prompt (e.g., "a red truck", "a person wearing a yellow jacket").
- **Frame Sampling**: Samples frames from the video at a configurable interval.
- **Open-Vocabulary Object Detection**: Utilizes Grounding DINO, an open-vocabulary object detector, to check each sampled frame against the provided text prompt. This allows for flexible and dynamic search queries without pre-defined object categories.
- **Results & Annotations**: Returns every frame that matches the prompt with a confidence score above a specified threshold. Each result includes:
    - The timestamp of the matched frame.
    - A confidence score for the detection.
    - An annotated image with the detected object highlighted.
    - Data is returned as JSON over a public URL, making it easily consumable by external applications.

## How It Works

The Colab notebook sets up a FastAPI application that exposes two endpoints:

- `GET /health`: A simple endpoint to confirm the server is reachable and to check the device (GPU/CPU) being used.
- `POST /analyze`: Accepts a video file, a text prompt, and scan settings (interval, threshold). It processes the video using the `process_video` function and returns the analysis results.

## Technologies Used

- **Google Colab**: Provides a free GPU environment for accelerated processing.
- **FastAPI**: A modern, fast (high-performance) web framework for building APIs with Python 3.7+.
- **`pyngrok`**: Integrates `ngrok` to create secure public URLs for the FastAPI server running in Colab.
- **`transformers` library**: For loading and utilizing the Grounding DINO model (`IDEA-Research/grounding-dino-tiny`).
- **`Pillow` (PIL)**: For image manipulation and drawing bounding boxes.
- **`OpenCV` (`cv2`)**: For video frame extraction.

## Setup and Usage

1.  **Open in Google Colab**: Open the `ask-the-footage.ipynb` notebook in Google Colab.
2.  **Change Runtime Type**: Go to `Runtime > Change runtime type` and select `T4 GPU` for faster processing. (CPU will also work but will be significantly slower).
3.  **Get `ngrok` Auth Token**: Obtain a free `ngrok` authentication token from [dashboard.ngrok.com/get-started/your-authtoken](https://dashboard.ngrok.com/get-started/your-authtoken).
4.  **Install Dependencies**: Run the first code cell to install all necessary Python packages.
5.  **Load Detection Model**: Run the second code cell to load the Grounding DINO model.
6.  **Configure `ngrok`**: Paste your `ngrok` auth token into the designated variable in the `Go live` section of the notebook.
7.  **Start Server**: Run the `Go live` cell. This cell will block as the server starts and will print a public URL. Copy this URL.
8.  **Integrate**: Use the provided public URL in your client application (e.g., the paired Next.js app `video-ai-search`) to send video analysis requests.

## API Endpoints

### `GET /health`

Returns the server status and the processing device.

**Response Example:**
```json
{
  "status": "ok",
  "device": "cuda"
}
```

### `POST /analyze`

Analyzes an uploaded video for objects matching a given prompt.

**Request Body (multipart/form-data):**
- `video`: The video file to upload.
- `prompt`: The text query for object detection (e.g., "a red truck").
- `interval` (optional): Frame sampling interval in seconds (default: `1.0`).
- `threshold` (optional): Confidence threshold for detections (default: `0.3`).

**Response Example:**
```json
{
  "prompt": "a red truck",
  "count": 2,
  "frames_scanned": 120,
  "results": [
    {
      "timestamp_sec": 5.2,
      "timestamp_str": "00:00:05",
      "score": 0.789,
      "label": "a red truck",
      "box": {"xmin": 100, "ymin": 150, "xmax": 200, "ymax": 250},
      "image_base64": "data:image/jpeg;base64,..." (base64 encoded JPEG)
    },
    // ... more results
  ]
}
```

## Notes and Limitations

-   **Not Real-time Streaming**: This server processes uploaded video clips. For short clips, it typically takes seconds to a minute, depending on length and scan interval.
-   **Ephemeral `ngrok` URLs**: The public `ngrok` URL changes every time the server cell is restarted. You'll need to re-copy it into your client application each time.
-   **Colab Session Limits**: Free Colab sessions have inactivity and runtime limits. Keep the tab open during testing.
-   **Prompt Phrasing**: Grounding DINO performs best with concise, literal noun phrases (e.g., "a red truck" is better than "can you find any trucks that are red").
-   **Model Choice**: For higher accuracy at the cost of speed, consider changing the model in step 2 to `"IDEA-Research/grounding-dino-base"`.
```

# Fitness Video Segmentation App

This application helps you segment fitness videos from YouTube into individual exercise segments using pose detection. It analyzes body movements to automatically identify different exercise sections within workout videos.

## Features

- Download YouTube fitness videos for processing
- Automated video segmentation based on pose detection
- Exercise labeling and organization
- Web interface for managing videos and segments
- MinIO storage integration for video segments
- Thumbnail generation for segments
- Search and filter capabilities

## Requirements

- Python 3.7+
- MinIO server (for storage)
- OpenCV and MediaPipe for pose detection
- Flask for the web interface

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/cheedli/Fitness_Videos_Segmentation.git
cd fitness-video-segmentation
```

### 2. Create and activate a virtual environment (recommended)

```bash
python -m venv venv
# On Windows
venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```


### 4. Set up MinIO

The app uses MinIO for storage. You need to have a MinIO server running.

1. [Download and install MinIO](https://min.io/download)
2. Start MinIO server:

```bash
# Example for local development (using default credentials)
minio server /path/to/data
```

By default, the app expects MinIO to be running at `localhost:9000` with credentials `minioadmin:minioadmin`.

### 5. Configure the application

Edit the MinIO configuration in the application if needed:

```python
# MinIO Configuration
MINIO_ENDPOINT = 'localhost:9000'  # Change if your MinIO server is elsewhere
MINIO_ACCESS_KEY = 'minioadmin'    # Replace with your access key 
MINIO_SECRET_KEY = 'minioadmin'    # Replace with your secret key
MINIO_SECURE = False               # Set to True if using HTTPS
MINIO_BUCKET_NAME = 'fitness-videos'  # Bucket for storing videos
```

## Running the Application

### 1. Start the Flask server

```bash
python app.py
```

The application will be accessible at `http://localhost:5000`

### 2. Using the application

1. **Add Videos**:
   - Go to "Add Video" and enter a YouTube video ID
   - Fill in the metadata (title, body part, difficulty, etc.)
   - Submit to queue for processing

2. **Process Videos**:
   - Videos will be downloaded and processed automatically
   - You can track processing status on the status page

3. **View & Manage Segments**:
   - Access video details to see all generated segments
   - Label exercise segments
   - Delete unwanted segments

## How the Segmentation Process Works

The application uses MediaPipe's pose detection to identify exercise segments in fitness videos:

### 1. Pose Detection with MediaPipe

- MediaPipe's pose estimation model identifies 33 key body landmarks (joints) in each video frame
- Each landmark has 3D coordinates (x, y, z) and a visibility score
- The application tracks these landmarks through the video

### 2. Joint Angle Calculation

- The system calculates important joint angles using sets of three landmarks
- For example, the elbow angle is calculated using shoulder, elbow, and wrist landmarks
- Eight key joint angles are tracked:
  - Left and right elbow angles
  - Left and right shoulder angles
  - Left and right knee angles
  - Left and right hip angles

### 3. Pose Difference Detection

- The application compares poses between frames using a weighted approach:
  - 60% weight to angular changes (joint angle differences)
  - 40% weight to positional differences of key landmarks
- Different body parts are weighted by importance (e.g., core/trunk movements have higher weights)
- Only landmarks with good visibility are included in calculations

### 4. Segment Boundary Detection

- When the pose difference exceeds a threshold (default: 0.12) for several consecutive frames (default: 5), a new exercise segment is created
- Minimum segment duration ensures meaningful exercise blocks (default: 3 seconds)
- Each identified segment is saved as a separate video file with metadata

### 5. Visualization

- The system can generate visualization videos showing the detected pose skeleton, segment boundaries, and difference metrics

This approach effectively identifies transitions between different exercises in fitness videos, as these transitions typically involve significant changes in body position and joint angles.

## Troubleshooting

### Video Download Issues

If you have issues downloading videos directly, you can:

1. Use the "Upload Local Video" feature to manually upload videos
2. Ensure you have yt-dlp installed: `pip install yt-dlp`
3. Check your internet connection

### Processing Errors

If segmentation fails:

1. Check the error message in the status page
2. Verify your OpenCV and MediaPipe installations
3. Try lowering the video resolution if your system has memory constraints

### Storage Issues

If segments are not properly saved or accessible:

1. Make sure your MinIO server is running
2. Check the MinIO credentials in the application
3. Verify the bucket exists in MinIO
4. Use the "Check MinIO Uploads" feature to verify uploads

## Advanced Configuration

### Segmentation Parameters

You can adjust segmentation sensitivity in the `segment_fitness_video` function:

```python
segments = segment_fitness_video(
    video_path=video_path,
    output_dir=output_dir,
    vid_link=vid_link,
    threshold=0.12,  # Increase for less segments, decrease for more
    min_segment_duration=3,  # Minimum segment length in seconds
    confirmation_frames=5,  # Number of frames needed to confirm a change
    skip_frames=2,  # Process every nth frame for efficiency
    visualize=True  # Creates a visualization video
)
```

### Directory Structure

The application uses the following directories:

- `videos/` - Downloaded YouTube videos
- `segmentation/` - Segmented exercise clips
- `test_videos/` - Local videos for testing without downloading
- `templates/` - HTML templates for the web interface
- `static/` - Static assets for the web interface



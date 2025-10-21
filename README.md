# Feed Level Monitoring System

A comprehensive computer vision system for automated feed level monitoring in agricultural settings using YOLO object detection and real-time web dashboard.

## 🌟 Overview

This system uses a custom-trained YOLO model to monitor feed levels in containers and provides real-time alerts and status updates through a web interface. The system is designed to run on Raspberry Pi with camera module and can classify feed levels into five categories from empty to full.

## 🚀 Features

- **Real-time Object Detection**: Custom YOLO11n model for feed level classification
- **Live Video Streaming**: Annotated video feed with detection overlays
- **Web Dashboard**: Responsive web interface with real-time statistics
- **Smart Alert System**: Email notifications with confirmation mechanism
- **Configurable Settings**: Adjustable confidence thresholds and detection intervals
- **System Monitoring**: CPU temperature and memory usage tracking
- **Detection History**: Visual chart showing detection patterns over time

## 📋 Feed Level Classification

The system detects five feed level states:

| Class | Description | Alert Level |
|-------|-------------|-------------|
| `grain0` | The feeder is empty, needs refill ASAP! | 🔴 Critical |
| `grain25` | Feed is almost empty, refill required promptly | 🟡 Warning |
| `grain50` | Feed is at 50%, preparation recommended | 🔵 Info |
| `grain75` | Feed is sufficient, no action needed | 🟢 Normal |
| `grain100` | Feeder just refilled, no action needed | 🟢 Normal |

## 🛠️ System Architecture

### Components
- **Backend**: Python Flask server with YOLO detection
- **Frontend**: HTML/CSS/JavaScript dashboard
- **Detection**: Custom YOLO11n model (`my_model.pt`)
- **Camera**: Raspberry Pi Camera Module
- **Alerts**: SMTP email notifications

### File Structure
```
feed-monitoring-system/
│
├── yolo_detect8.py          # Main Flask application
├── dashboard.html           # Web dashboard template
├── my_model.pt             # Custom YOLO model
├── requirements.txt         # Python dependencies
├── templates/              # Flask templates directory
└── README.md               # This file
```

## 📦 Installation

### Prerequisites
- Raspberry Pi 3/4/5 with Raspberry Pi OS (Bullseye or newer)
- Raspberry Pi Camera Module (v1 or v2)
- Python 3.7 or higher
- Internet connection

### 1. Hardware Setup
```bash
# Enable camera interface
sudo raspi-config
# Navigate to Interface Options → Camera → Enable
# Reboot after enabling
sudo reboot
```

### 2. Software Installation

#### Option A: Quick Install with requirements.txt
```bash
# Clone or download the project files
mkdir feed-monitoring-system
cd feed-monitoring-system

# Place all project files in this directory:
# - yolo_detect8.py
# - dashboard.html
# - my_model.pt
# - requirements.txt

# Create virtual environment (recommended)
python3 -m venv feed_monitor
source feed_monitor/bin/activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

#### Option B: Manual Installation
```bash
# Create virtual environment
python3 -m venv feed_monitor
source feed_monitor/bin/activate

# Install core packages
pip install ultralytics opencv-python picamera2 flask pillow numpy
pip install secure-smtplib email-validator psutil requests
```

### 3. Model Preparation

#### Training Process Overview (Using Google Colab)
1. **Data Collection**: Capture 400-500 images of feed containers at different levels
2. **Labeling**: Use Label Studio to annotate images with five classes
3. **Export**: Export annotations in YOLO format
4. **Training**: Train YOLO11n model using Google Colab
5. **Export**: Download trained model as `my_model.pt`

#### Expected Model Structure
Ensure your `my_model.pt` file is in the project root directory and contains:
- 5 output classes: `grain0`, `grain25`, `grain50`, `grain75`, `grain100`
- Trained on image size 640x640
- Validation mAP50 > 0.85 (recommended)

### 4. Email Configuration

#### Gmail Setup (Recommended)
1. Enable 2-factor authentication on your Gmail account
2. Generate app-specific password:
   - Go to Google Account → Security → 2-Step Verification → App passwords
   - Generate password for "Mail" application
   - Copy the 16-character password

3. Update email credentials in `yolo_detect8.py`:
```python
EMAIL_ADDRESS = 'your_email@gmail.com'
EMAIL_PASSWORD = 'your_16_char_app_password'  # Use app password, not regular password
RECIPIENT_EMAIL = 'recipient@gmail.com'
```

#### Other Email Providers
For other providers (Outlook, Yahoo, etc.), update SMTP settings:
```python
SMTP_SERVER = 'smtp.yourprovider.com'  # e.g., 'smtp.outlook.com'
SMTP_PORT = 587  # Usually 587 for TLS
```

## 🚀 Usage

### Starting the System
```bash
# Navigate to project directory
cd feed-monitoring-system

# Activate virtual environment
source feed_monitor/bin/activate

# Run the application
python yolo_detect8.py
```

### Expected Output
```
* Serving Flask app 'yolo_detect8'
* Debug mode: off
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000
* Running on http://[YOUR_PI_IP]:5000
```

### Accessing the Dashboard
1. Find your Raspberry Pi's IP address:
   ```bash
   hostname -I
   ```

2. Open web browser on any device connected to the same network
3. Navigate to: `http://[RASPBERRY_PI_IP]:5000`
   - Example: `http://192.168.1.100:5000`

4. The dashboard will load with live video feed and statistics

## 📊 Dashboard Guide

### Live Feed Panel
- Real-time camera feed with YOLO detection annotations
- Bounding boxes showing detected feed levels
- Confidence scores displayed on detections

### Detection Status Panel
- **FPS**: Frames per second of detection pipeline (typically 5-15 FPS on Pi 4)
- **Total Detections**: Cumulative detection count since startup
- **Last Detection**: Most recent detection with timestamp
- **Confidence**: Confidence score of last detection (0-100%)
- **CPU Temperature**: System temperature monitoring with warning above 70°C
- **Status Message**: Current feed level status with color-coded alerts
- **Confirmation Progress**: Visual progress bar showing confirmation status

### Controls Panel
- **Email Alerts**: Toggle email notifications on/off
- **Confidence Threshold**: Adjust detection sensitivity (0.1-0.9)
  - Higher values = fewer but more confident detections
  - Lower values = more detections but potential false positives
- **Detection Interval**: Set seconds between detections (1-60)
  - Longer intervals reduce CPU usage
- **Confirmations Needed**: Number of consistent detections required for confirmation (1-20)
  - Higher values reduce false alerts but increase response time

### Detection History Panel
- Bar chart showing detection frequency for each class
- Color-coded bars matching alert levels
- Updates in real-time as new detections occur

## ⚙️ Configuration

### Default Settings
```python
{
    'email_alerts': True,           # Enable email notifications
    'min_confidence': 0.5,          # 50% confidence threshold
    'detection_enabled': True,      # Enable detection pipeline
    'detection_interval': 10,       # Check every 10 seconds
    'confirm_count': 10             # Require 10 consistent detections
}
```

### Performance Optimization Tips

#### For Raspberry Pi 3:
```python
# In the Controls panel, set:
detection_interval = 15  # seconds
confirm_count = 8        # confirmations
min_confidence = 0.6     # higher threshold
```

#### For Raspberry Pi 4/5:
```python
# Can handle more frequent detection
detection_interval = 5   # seconds
confirm_count = 5        # confirmations
min_confidence = 0.4     # more sensitive
```

## 🐛 Troubleshooting

### Common Issues and Solutions

#### 1. Camera Not Detected
```bash
# Test camera detection
vcgencmd get_camera
# Expected: supported=1 detected=1

# If not detected:
sudo raspi-config
# Interface Options → Camera → Enable → Reboot
```

#### 2. Model Loading Errors
- Verify `my_model.pt` is in the same directory as `yolo_detect8.py`
- Check file permissions: `chmod 644 my_model.pt`
- Ensure model was trained with correct class names

#### 3. Email Sending Failures
- Verify app password (16 characters, no spaces)
- Check internet connectivity: `ping google.com`
- Test SMTP settings with simple Python script
- Check spam folder for test emails

#### 4. High CPU Temperature
```bash
# Monitor temperature
vcgencmd measure_temp

# If consistently above 70°C:
# - Reduce detection frequency
# - Add heat sink
# - Improve ventilation
# - Use lower resolution in camera settings
```

#### 5. Web Interface Not Accessible
- Check firewall settings: `sudo ufw status`
- Verify Python/Flask is running without errors
- Check if port 5000 is available: `netstat -tulpn | grep 5000`

### Debug Mode
For troubleshooting, you can enable debug output by modifying `yolo_detect8.py`:
```python
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, threaded=True, debug=True)
```

## 🔧 Advanced Configuration

### Changing Camera Resolution
Modify in `yolo_detect8.py`:
```python
CAMERA_WIDTH = 640    # Lower for better performance
CAMERA_HEIGHT = 480   # Higher for better accuracy
```

### Custom Status Messages
Update the messages dictionary:
```python
'messages': {
    'grain0': 'Your custom empty message',
    'grain25': 'Your custom low message',
    # ... etc
}
```

### System Service Setup (Auto-start on boot)
```bash
# Create service file
sudo nano /etc/systemd/system/feed-monitor.service
```

```ini
[Unit]
Description=Feed Level Monitoring System
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/feed-monitoring-system
ExecStart=/home/pi/feed-monitoring-system/feed_monitor/bin/python /home/pi/feed-monitoring-system/yolo_detect8.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start service
sudo systemctl enable feed-monitor.service
sudo systemctl start feed-monitor.service
```

## 📈 Model Training Details

### Dataset Specifications
- **Total Images**: 2,000-2,500 (400-500 per class)
- **Image Size**: 640x640 pixels
- **Classes**: 5 (grain0, grain25, grain50, grain75, grain100)
- **Split**: 80% training, 20% validation

### Training Commands (Google Colab)
```python
from ultralytics import YOLO

# Load pre-trained model
model = YOLO('yolo11n.pt')

# Train with custom dataset
results = model.train(
    data='dataset.yaml',
    epochs=100,
    imgsz=640,
    batch=32,
    device=0,  # Use GPU in Colab
    patience=15,
    lr0=0.01,
    weight_decay=0.0005,
    save=True,
    cache=True
)
```

### Expected Performance Metrics
- **mAP50**: > 0.85
- **Precision**: > 0.80
- **Recall**: > 0.80
- **Inference Speed**: 10-30ms on Raspberry Pi 4

## 🔒 Security Considerations

- ✅ Use app passwords instead of main email passwords
- ✅ Run in virtual environment
- ✅ Regular system updates
- ⚠️ Change default credentials if used in production
- ⚠️ Consider VPN for remote access
- ⚠️ Regular security updates for Raspberry Pi OS

## 📞 Support

### Getting Help
1. Check the troubleshooting section above
2. Verify all dependencies are installed correctly
3. Ensure camera and model files are properly configured

### Performance Tips
- Use Raspberry Pi 4 or newer for better performance
- Ensure adequate power supply (5V 3A recommended)
- Use quality microSD card (Class 10 or better)
- Keep system cool with proper heat dissipation

## 📄 License

This project is licensed under the MIT License. See LICENSE file for details.

## 🙏 Acknowledgments

- **Ultralytics** for YOLO implementation and continuous improvements
- **Raspberry Pi Foundation** for hardware and camera support
- **Label Studio** for excellent annotation tools
- **Google Colab** for accessible training resources

---

**Maintenance Note**: Regularly update your system and dependencies for security and performance improvements:

```bash
# Monthly maintenance
sudo apt update && sudo apt upgrade -y
source feed_monitor/bin/activate
pip install --upgrade -r requirements.txt
```

**Farm Safety**: This system is an assistive tool. Always perform manual checks and maintain traditional monitoring methods as backup.

# 🍜 Poke Bowl Inventory System

**Production-ready computer vision inventory system for NVIDIA Jetson Orin Nano**

A real-time object detection and inventory tracking system designed for restaurant environments. Uses YOLO for detection, temporal smoothing for stability, and a local web interface for monitoring.

---

## 📋 System Overview

### Hardware
- **Compute**: NVIDIA Jetson Orin Nano
- **Camera**: USB megapixel camera (UVC-compliant)
- **Display**: Hamtysan 7" HDMI monitor (or any HDMI display)
- **OS**: JetPack 6.x (Ubuntu 22.04)

### Features
- ✅ Real-time YOLO-based object detection with GPU acceleration
- ✅ Temporal smoothing for stable inventory counts
- ✅ WebSocket-based live video streaming
- ✅ Automatic startup on boot (systemd)
- ✅ Graceful camera disconnect/reconnect handling
- ✅ Low-latency, production-ready architecture
- ✅ Headless operation with web-based UI

---

## 🗂️ Repository Structure

```
Poke-Bowl---updated-January/
│
├── backend/
│   ├── main.py              # Application entry point
│   ├── camera.py            # USB camera handler with reconnection
│   ├── detector.py          # YOLO inference wrapper
│   ├── inventory.py         # Inventory tracking with smoothing
│   └── server.py            # Web server and streaming
│
├── frontend/
│   └── index.html           # Web UI (video feed + counts)
│
├── config/
│   └── config.yaml          # System configuration
│
├── deployment/
│   ├── pokebowl-inventory.service    # Systemd service file
│   ├── chromium-kiosk.service        # Browser kiosk mode service
│   ├── install_service.sh            # Service installation script
│   └── setup_autostart.sh            # Full auto-start setup
│
├── best.pt                  # Trained YOLO model
├── requirements.txt         # Python dependencies
├── dataset/                 # Training data (preserved)
└── Images/                  # Raw images (preserved)
```

---

## 🚀 Quick Start (Jetson Orin Nano)

### Prerequisites

1. **JetPack 6.x** installed on Jetson Orin Nano
2. **USB camera** connected
3. **HDMI display** connected (optional for headless operation)

### Step 1: Clone Repository

```bash
cd ~
git clone <repository-url> Poke-Bowl---updated-January
cd Poke-Bowl---updated-January
```

### Step 2: Install System Dependencies

```bash
sudo apt-get update
sudo apt-get install -y \
    python3-pip \
    python3-dev \
    libopencv-dev \
    python3-opencv \
    v4l-utils \
    chromium-browser \
    gstreamer1.0-tools \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad
```

### Step 3: Install PyTorch (Jetson-Optimized)

**IMPORTANT**: Use the official NVIDIA PyTorch wheel for Jetson:

```bash
# Download PyTorch for JetPack 6.x
wget https://developer.download.nvidia.com/compute/redist/jp/v60/pytorch/torch-2.1.0a0+41361538.nv23.06-cp310-cp310-linux_aarch64.whl

# Install
pip3 install torch-2.1.0a0+41361538.nv23.06-cp310-cp310-linux_aarch64.whl
```

### Step 4: Install Torchvision

```bash
sudo apt-get install libjpeg-dev zlib1g-dev

git clone --branch v0.16.0 https://github.com/pytorch/vision torchvision
cd torchvision
python3 setup.py install --user
cd ..
```

### Step 5: Install Python Dependencies

```bash
pip3 install -r requirements.txt
```

### Step 6: Verify Camera

```bash
# List available cameras
v4l2-ctl --list-devices

# Test camera capture (Ctrl+C to stop)
ffplay /dev/video0
```

Update `config/config.yaml` if your camera is not at `/dev/video0`:

```yaml
camera:
  index: 0  # Change to 1 for /dev/video1, etc.
```

### Step 7: Test System Manually

```bash
cd backend
python3 main.py
```

You should see:
```
============================================================
Poke Bowl Inventory System
============================================================
Camera opened: 1280x720 @ 30fps
Model loaded in X.XXs
System ready!
Web interface available at: http://0.0.0.0:8080
============================================================
```

Open browser and navigate to `http://<jetson-ip>:8080`

### Step 8: Setup Auto-Start

```bash
cd deployment
sudo bash setup_autostart.sh
```

This will:
1. Install the backend service
2. Install the Chromium kiosk mode service
3. Enable both to start on boot

### Step 9: Reboot and Test

```bash
sudo reboot
```

After reboot, the system should automatically:
1. Start the inventory backend
2. Launch Chromium in fullscreen showing the web interface

---

## 🎛️ Configuration

Edit `config/config.yaml` to customize system behavior:

### Camera Settings

```yaml
camera:
  index: 0           # V4L2 device index
  width: 1280        # Resolution width
  height: 720        # Resolution height
  fps: 30            # Target FPS
```

### Detection Settings

```yaml
detector:
  model_path: best.pt           # Path to YOLO model
  conf_threshold: 0.25          # Confidence threshold
  iou_threshold: 0.45           # NMS IoU threshold
  imgsz: 640                    # YOLO input size
  device: '0'                   # '0' for GPU, 'cpu' for CPU
  half: true                    # FP16 precision (recommended)
```

### Inventory Smoothing

```yaml
inventory:
  smoothing_window: 10          # Frames to average
  smoothing_method: median      # median, mean, or mode
```

### Web Server

```yaml
server:
  host: '0.0.0.0'              # Listen on all interfaces
  port: 8080                   # HTTP port
```

---

## 🔧 System Management

### Service Commands

```bash
# Start the system
sudo systemctl start pokebowl-inventory

# Stop the system
sudo systemctl stop pokebowl-inventory

# Restart the system
sudo systemctl restart pokebowl-inventory

# Check status
sudo systemctl status pokebowl-inventory

# View live logs
sudo journalctl -u pokebowl-inventory -f

# Disable auto-start
sudo systemctl disable pokebowl-inventory

# Enable auto-start
sudo systemctl enable pokebowl-inventory
```

### Kiosk Mode Commands

```bash
# Start browser kiosk
sudo systemctl start chromium-kiosk

# Stop browser kiosk
sudo systemctl stop chromium-kiosk

# Check kiosk status
sudo systemctl status chromium-kiosk
```

### Manual Testing (Without Service)

```bash
cd ~/Poke-Bowl---updated-January/backend
python3 main.py
```

Press `Ctrl+C` to stop.

---

## 📊 Monitoring and Debugging

### View Logs

```bash
# System logs
sudo journalctl -u pokebowl-inventory -f

# Application logs (if file logging enabled)
tail -f /tmp/pokebowl_inventory.log
```

### Check System Resources

```bash
# GPU usage
tegrastats

# CPU/Memory
htop

# Camera status
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --all
```

### Performance Metrics

Access the web interface at `http://<jetson-ip>:8080` to see:
- Live FPS
- Inference time
- Frame count
- Active connections

Or use the API:

```bash
# Health check
curl http://localhost:8080/health

# Statistics
curl http://localhost:8080/api/stats
```

---

## 🐛 Troubleshooting

### Camera Not Detected

```bash
# Check if camera is recognized
lsusb

# Check video devices
ls -l /dev/video*

# Test camera
v4l2-ctl --list-formats-ext -d /dev/video0
```

**Solution**: Update `config/config.yaml` with the correct camera index.

### CUDA Out of Memory

**Symptoms**: Model fails to load or inference crashes

**Solutions**:
1. Enable half precision in `config.yaml`:
   ```yaml
   detector:
     half: true
   ```

2. Reduce input size:
   ```yaml
   detector:
     imgsz: 416  # or 320
   ```

3. Close other applications using GPU

### Low FPS / High Latency

**Solutions**:
1. Lower camera resolution:
   ```yaml
   camera:
     width: 640
     height: 480
   ```

2. Reduce streaming FPS:
   ```yaml
   stream:
     target_fps: 15
   ```

3. Lower YOLO input size:
   ```yaml
   detector:
     imgsz: 416
   ```

### Service Won't Start

```bash
# Check service status
sudo systemctl status pokebowl-inventory

# View error logs
sudo journalctl -u pokebowl-inventory -n 50

# Check permissions
ls -la ~/Poke-Bowl---updated-January/backend/main.py

# Make sure it's executable
chmod +x ~/Poke-Bowl---updated-January/backend/main.py
```

### Web Interface Not Loading

1. Check if service is running:
   ```bash
   sudo systemctl status pokebowl-inventory
   ```

2. Test direct connection:
   ```bash
   curl http://localhost:8080
   ```

3. Check firewall:
   ```bash
   sudo ufw status
   sudo ufw allow 8080
   ```

4. Verify network access:
   ```bash
   ifconfig  # Note the IP address
   # Access from another device: http://<jetson-ip>:8080
   ```

---

## 🔬 Development and Testing

### Run Tests

```bash
cd backend

# Test camera
python3 -c "from camera import USBCamera; cam = USBCamera(); cam.open(); print(cam.get_info())"

# Test detector
python3 -c "from detector import YOLODetector; det = YOLODetector('../best.pt'); det.load(); print(det.get_info())"
```

### Development Mode

For development with auto-reload, modify `backend/main.py` or run directly:

```bash
cd backend
python3 main.py
```

Edit code and restart manually.

---

## 📦 Model Training

The system uses a pre-trained YOLO model (`best.pt`) for detecting 40 product classes.

### Classes Detected

See `dataset/pokebowl_dataset/data.yaml` for the full list of 40 classes including:
- Beverages (Coke, Sprite, Perrier, etc.)
- Fruits (Mango, Cantaloupe, Strawberry, etc.)
- Specialty items (Philadelphia rolls, etc.)

### Retraining

To retrain the model with new data:

1. Add images and labels to `dataset/pokebowl_dataset/`
2. Update `data.yaml` with class names
3. Train using Ultralytics:

```bash
from ultralytics import YOLO

model = YOLO('yolo11n.pt')  # or yolov8n.pt
results = model.train(
    data='dataset/pokebowl_dataset/data.yaml',
    epochs=100,
    imgsz=640,
    device=0
)
```

4. Replace `best.pt` with the new trained model

---

## 🔒 Security Considerations

### Production Deployment

1. **Change default port**: Edit `config/config.yaml`
2. **Restrict access**: Set `host: '127.0.0.1'` for localhost only
3. **Add authentication**: Implement in `backend/server.py` (not included)
4. **Use HTTPS**: Add reverse proxy (nginx/caddy) with SSL
5. **Firewall rules**:
   ```bash
   sudo ufw enable
   sudo ufw allow 22     # SSH
   sudo ufw allow 8080   # Web interface
   ```

---

## 📈 Performance Optimization

### Jetson Power Mode

Set maximum performance:

```bash
# Check current mode
sudo nvpmodel -q

# Set to maximum performance (mode 0)
sudo nvpmodel -m 0

# Set fan to maximum
sudo jetson_clocks
```

### GPU Memory Management

Monitor GPU memory:

```bash
tegrastats
```

If running low on memory:
1. Enable FP16 precision (`half: true`)
2. Reduce batch size (currently 1 for real-time)
3. Lower input resolution

---

## 📝 License

[Specify your license here]

---

## 🤝 Contributing

[Add contribution guidelines if applicable]

---

## 📧 Support

For issues, questions, or feature requests:
- Check the troubleshooting section above
- Review logs: `sudo journalctl -u pokebowl-inventory -f`
- [Add contact/support information]

---

## 🙏 Acknowledgments

- **Ultralytics YOLO**: https://github.com/ultralytics/ultralytics
- **NVIDIA Jetson**: https://developer.nvidia.com/embedded/jetson
- Built for restaurant inventory management

---

## 📅 Changelog

### Version 1.0.0 (January 2026)
- Initial production release
- YOLO-based detection with 40 product classes
- Real-time WebSocket streaming
- Temporal smoothing for inventory stability
- Auto-start systemd service
- Chromium kiosk mode integration

---

**System Status**: ✅ Production Ready

**Last Updated**: January 2026


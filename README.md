# ***Hailo-10H and Raspberry Pi AI HAT+ 2 Installation Instructions for Raspberry Pi OS on Raspberry Pi 5***
Written by: XtendedGreg February 2, 2026 [XtendedGreg Youtube Channel](https://www.youtube.com/@xtendedgreg)

[![Watch the video](https://img.youtube.com/vi/NXKM3wsw4Ow/maxresdefault.jpg)](https://youtube.com/live/NXKM3wsw4Ow)

## Prerequisites
- Hardware: Raspberry Pi 5 (8GB RAM recommended).
- Accelerator: Raspberry Pi AI HAT+ (40 TOPS / Hailo-10H version).
- Cooling: Active Cooler installed + AI HAT heatsink installed (Mandatory).
- OS: Raspberry Pi OS Bookworm (64-bit).

## 1. Preparation
### Upgrade Packages
```
sudo apt update
sudo apt upgrade
```

### Change PCIe Speed
```
sudo raspi-config
```
Advanced > PCIe Speed = 3
(reboot)

## 2. Install Drivers
```
sudo apt install hailo-h10-all
hailortcli fw-control identify
```

## 3. Download and Compile Models
### Install Dependencies
```
sudo apt install -y git cmake g++ wget libssl-dev libcurl4-openssl-dev
```

### Clone GIT Repository
```
cd ~
git clone https://github.com/hailo-ai/hailo_model_zoo_genai.git
```

### Compile Models
```
cd ~/hailo_model_zoo_genai/
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . -j4
```

## 4. Move files to runtime locations
### Exec Files
```
cd ~/hailo_model_zoo_genai/genai/build
mkdir -p ~/.local/bin
cp ./src/apps/server/hailo-ollama ~/.local/bin/
```

### Config Files
```
mkdir -p ~/.config/hailo-ollama/
cp ../config/hailo-ollama.json ~/.config/hailo-ollama/
```

### Models
```
mkdir -p ~/.local/share/hailo-ollama
cp -r ../models/ ~/.local/share/hailo-ollama
```

## 5. Add Exec path to PATH
```
export PATH=$HOME/.local/bin:$PATH
```

## 6. Start the Hailo Server
```
hailo-ollama
```
- Server will run in the foreground showing any commands that come to the server
- Press CTRL + C to exit server

## 7. Install Docker
```
curl -sSL https://get.docker.com | sh
```

## 8. Setup user permissions
```
sudo usermod -aG docker $USER
```

## 9. Run Docker and the Hailo Server
```
sudo docker run -d   --network=host   -e OLLAMA_BASE_URL=http://127.0.0.1:8000   -v open-webui:/app/backend/data   --name open-webui   --restart always   ghcr.io/open-webui/open-webui:main
hailo-ollama
```

## 10. Connect to Web UI
### IN A WEB BROWSER GOTO: http://127.0.0.1:8080
- Create Login
- The login is local only so the email address does not need to be real
- After login, the chat interface will be displayed but there will be no models available

## 11. Install Models
### Get list of available models
```
curl --silent http://localhost:8000/hailo/v1/list
```

### Choose a model from the list and request the server download it
```
curl http://localhost:8000/api/pull   -H "Content-Type: application/json"   -d '{ "model": "[MODEL-NAME-FROM-LIST-HERE]" }'
```
Repeat this step with additional models.

#### Example
```
curl http://localhost:8000/api/pull   -H "Content-Type: application/json"   -d '{ "model": "qwen2.5-coder:1.5b" }'
```

## 12. Reboot
```
reboot
```

# Install Scripts to Launch on Boot
## 1. Install Dependencies
```
sudo apt install git
```

## 2. Clone This Repository
```
cd ~
git clone https://github.com/XtendedGreg/raspberrypi-hailo-10.git
cd raspberrypi-hailo-10/root
```

## 3. Copy files
### Copy Hailo Server Launch Script
```
sudo cp usr/local/bin/hailo-server /usr/local/bin/
sudo chmod +x /usr/local/bin/hailo-server
```

### Copy Systemd Service Script
```
sudo cp etc/systemd/system/hailo-server.service /etc/systemd/system/
sudo chmod +x /etc/systemd/system/hailo-server.service
sudo sed -i "s/\[USER\]/$USER/g" /etc/systemd/system/hailo-server.service
```

## 4. Enable Service to Start on Boot and Start Service Now
```
sudo systemctl enable hailo-server.service
sudo systemctl start hailo-server.service
```

## 5. Reboot to Test
```
reboot
```

# Updates
## 1. Update GIT Repository
```
cd ~/raspberrypi-hailo-10/root
git pull
```

## 2. Copy files
### Copy Hailo Server Launch Script
```
sudo cp usr/local/bin/hailo-server /usr/local/bin/
sudo chmod +x /usr/local/bin/hailo-server
```

### Copy Systemd Service Script
```
sudo cp etc/systemd/system/hailo-server.service /etc/systemd/system/
sudo chmod +x /etc/systemd/system/hailo-server.service
sudo sed -i "s/\[USER\]/$USER/g" /etc/systemd/system/hailo-server.service
```

## 3. Reload Systemd
```
sudo systemctl daemon-reload
```

## 4. Restart hailo-server
```
sudo systemctl restart hailo-server.service
```

# Monitor Server Status
## Tail Logs
Shows connection status and responses.
```
tail -f /var/log/hailo-server/hailo-server.log
```

## Systemd Service Status
Check if the service is running or if it has crashed.
```
sudo systemctl status hailo-server.service
```

# ⚽ Online Soccer Robot: Ultra-Low Latency Streaming

![status](https://img.shields.io/badge/status-production--ready-green)
![latency](https://img.shields.io/badge/latency-~100ms-blue)
![webrtc](https://img.shields.io/badge/streaming-WebRTC-orange)

This repository contains the production-grade configuration for a real-time, high-stability video pipeline designed for remote-controlled robots.

---

## 🏗 System Architecture & Flow

The pipeline is engineered to bypass standard buffering mechanisms to achieve sub-200ms latency over public networks.

1. **Source:** USB Camera (DShow)
2. **Encoder:** FFmpeg (h264_nvenc or libx264) with zero-latency tuning.
3. **Ingest:** RTSP (UDP) to MediaMTX.
4. **Signaling & Transport:** MediaMTX handles WebRTC ICE candidates.
5. **Gateway:** Caddy Reverse Proxy (Provides Essential HTTPS/TLS).
6. **Client:** Web Browser (WebRTC Player).

---

## 📋 Prerequisites & Tools

Before starting, download and extract the following binaries:

* **[MediaMTX](https://github.com/bluenviron/mediamtx/releases):** The core media server.
* **[FFmpeg](https://ffmpeg.org/download.html):** The "Swiss army knife" for video encoding.
* **[Caddy Server](https://caddyserver.com/download):** Required for automatic HTTPS (WebRTC security requirement).

---

## 🛠 Step 1: Infrastructure & Network Setup

### 1.1 DNS Configuration
WebRTC requires a **Secure Origin** (HTTPS) to function on mobile browsers.
1. Map a subdomain (e.g., `robot.yourdomain.com`) to your Public IP.
2. Create an **A Record** in your DNS panel (Cloudflare, ArvanCloud, etc.).

### 1.2 Router Port Forwarding
Ensure the following ports are open and directed to your streaming host:
* **TCP 80 & 443:** For Caddy (HTTP/HTTPS & SSL Challenge).
* **TCP 8554:** For RTSP ingest.
* **UDP 8889:** For WebRTC Media transport.
* **UDP 8189:** For WebRTC Signaling.

---

## ⚙️ Step 2: MediaMTX Configuration (`mediamtx.yml`)

The default config is not optimized for NAT traversal. Modify these specific lines:

```yaml
webrtc: yes
webrtcAddress: :8889
webrtcICEUDP: yes
webrtcEncryption: optional

# CRITICAL: Prevent internal IP leakage
webrtcIPsFromInterfaces: false

# Replace with your domain or Public IP
webrtcAdditionalHosts:
  - robot.yourdomain.com

# Google's STUN server for NAT traversal
webrtcICEServers:
  - urls: [stun:stun.l.google.com:19302]
```

---

## 🔒 Step 3: Caddy Setup (The SSL Bridge)

Browsers block WebRTC features on non-localhost `http` links. Caddy automates the SSL process. Create a file named `Caddyfile` in your Caddy folder:

```text
robot.yourdomain.com {
    reverse_proxy localhost:8889
}
```

Run Caddy: `caddy run` (It will automatically fetch SSL certificates from Let’s Encrypt).

---

## 🚀 Step 4: The Production FFmpeg Command

This command is the result of extensive debugging to solve Clock Drift and Buffer Bloat. It forces a strict 20 FPS stream to match the browser’s jitter buffer.

```bash
./ffmpeg.exe -f dshow -i video="USB Camera" \
  -vcodec libx264 -preset ultrafast -tune zerolatency \
  -profile:v baseline -level 3.0 -pix_fmt yuv420p \
  -vf "fps=20" -g 20 -b:v 1M -maxrate 1M -bufsize 200k \
  -x264-params "nal-hrd=cbr:force-cfr=1:rc-lookahead=0:sync-lookahead=0:mbtree=0" \
  -f rtsp -rtsp_transport udp rtsp://localhost:8554/birdview
```

**Why this works:**
* `force-cfr=1`: Forces a Constant Frame Rate, preventing the “latency buildup” over time.
* `rc-lookahead=0`: Disables the encoder from “waiting” to see future frames.
* `bufsize 200k`: A tiny buffer ensures that if the network lags, frames are dropped rather than queued (No Latency Accumulation).

---

## 🔬 Engineering Insights: Solving “Buffer Bloat”

During testing, we noticed `jitterBufferDelay` in Chrome rising indefinitely. This was caused by the encoder running slightly faster than real-time (`speed=1.1x`).

**The Symptoms:**
* `dup` frames in FFmpeg logs.
* Video delay starting at 100ms and reaching 5s after ten minutes.

**The Fix:**
By using `-vf "fps=20"` combined with `force-cfr=1`, we synchronized the FFmpeg internal clock with the physical camera’s hardware clock, resulting in a permanent `speed=1.00x` and stable latency.

---

## 📊 Performance Benchmarks

| Metric | Performance |
| :--- | :--- |
| **Glass-to-Glass Latency** | ~120ms (Local) / ~180ms (4G Network) |
| **Stability** | 24h+ Continuous Stream without Drift |
| **CPU Usage** | <15% on Core i5 (Software Encoding) |
| **Network** | Constant 1Mbps (CBR) |

---

## 🔧 Troubleshooting

* **“ICE Connection Failed”**: Check if UDP port `8889` is open on your router.
* **“Not Secure” Warning**: Ensure Caddy is running and ports `80`/`443` are not blocked by a firewall.
* **Video Stuttering**: Lower the `-b:v` (bitrate) to `500k` if your upload speed is limited.

---

## 📝 License

Distributed under the MIT License. Built with ☕ and passion for Robotics.

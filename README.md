⚽ Online Soccer Robot

Ultra-Latency WebRTC Streaming for Remote Robot Control

A real-time video streaming pipeline designed for controlling a remote soccer robot over the internet with extremely low latency.



Architecture

USB Camera → FFmpeg Encoder → MediaMTX Server → WebRTC → Browser



FFmpeg Command

./ffmpeg.exe -f dshow -i video="USB Camera" \
-vcodec libx264 \
-preset ultrafast \
-tune zerolatency \
-profile:v baseline \
-level 3.0 \
-pix_fmt yuv420p \
-vf "fps=20" \
-g 20 \
-b:v 1M \
-maxrate 1M \
-bufsize 200k \
-x264-params "nal-hrd=cbr:force-cfr=1:rc-lookahead=0:sync-lookahead=0:mbtree=0" \
-f rtsp \
-rtsp_transport udp \
rtsp://localhost:8554/birdview




MediaMTX WebRTC Configuration

webrtc: yes
webrtcAddress: :8889
webrtcEncryption: optional

webrtcIPsFromInterfaces: false

webrtcAdditionalHosts:
  - YOUR_PUBLIC_IP_OR_DOMAIN

webrtcICEServers:
  - urls: [stun:stun.l.google.com:19302]




Required Port Forwarding

TCP





80



443



8554

UDP





8189



8889



Reverse Proxy (Caddy)

robot.example.com {
  reverse_proxy localhost:8889
}




Results





Encoder speed ≈ 1.00x



Duplicated frames: 0



Average jitter buffer delay ≈ 0.1s

System is stable and suitable for remote robot control over the internet.



License: MIT

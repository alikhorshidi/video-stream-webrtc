# video-stream-webrtc
Ultra-Low Latency WebRTC Streaming for Remote-Controlled Robots 

Robot WebRTC Streaming Guide

Ultra-Low Latency WebRTC Robot Streaming Guide 🤖 Stream like a Pro

This repository provides a comprehensive, step-by-step technical guide to setting up a high-performance, real-time video streaming system for an Online Footballer Robot. By combining FFmpeg, MediaMTX, and Caddy, we achieve sub-second latency stable enough for remote competitive control.



🏗 System Architecture





Capture & Encode: FFmpeg captures USB camera feed and encodes it to H.264 (Baseline Profile).



Streaming Server: MediaMTX acts as the core RTSP/WebRTC gateway.



Secure Tunneling: Caddy provides HTTPS (essential for WebRTC on mobile browsers).



Network Traversal: Port forwarding and STUN/ICE handling for remote internet access.



1. MediaMTX Configuration ()

The backbone of the project. We use WebRTC for the lowest possible latency.

Key Modifications

In your mediamtx.yml, ensure these critical sections are tuned for NAT traversal:





WebRTC Core Settings





webrtc: true



webrtcAddress: :8889



webrtcLocalUDPAddress: :8189 — crucial for ICE candidates



Network Traversal





webrtcIPsFromInterfaces: false



webrtcAdditionalHosts: [YOUR_PUBLIC_IP] — use your public IP or domain here



STUN Server for Peer-to-Peer Handshake





webrtcICEServers2:





url: stun:stun.l.google.com:19302



Path Configuration





paths:





all_others:





source: publisher



overridePublisher: true



2. FFmpeg: The "Secret Sauce" Command

After extensive debugging, including handling buffer bloat and clock drift, this command ensures a steady (1.000\times) speed and eliminates cumulative lag on mobile browsers.

./ffmpeg.exe -f dshow -i video="USB Camera" \
-vcodec libx264 -preset ultrafast -tune zerolatency \
-profile:v baseline -level 3.0 -pix_fmt yuv420p \
-vf "fps=20" -g 20 -b:v 1M -maxrate 1M -bufsize 200k \
-x264-params "nal-hrd=cbr:force-cfr=1:rc-lookahead=0:sync-lookahead=0:mbtree=0" \
-f rtsp -rtsp_transport udp rtsp://localhost:8554/birdview


Technical Breakdown





-profile:v baseline: Vital for mobile browser hardware decoder compatibility.



-vf "fps=20": Forces a constant frame rate to prevent jitterBufferDelay growth in WebRTC.



-tune zerolatency: Removes the look-ahead buffer in x264.



-bufsize 200k: Small buffer size prevents buffer bloat spikes.



3. Router & Network (Port Forwarding)

WebRTC requires specific ports to be open on your modem to allow external traffic to reach your local server.

ProtocolPortDescriptionTCP80 / 443HTTP/HTTPS (Caddy signaling)TCP8554RTSP (FFmpeg input)UDP8889WebRTC MediaUDP8189WebRTC ICE / Signaling



Pro Tip: Assign a Static IP to your PC in the router's DHCP settings so the forwarding rules don't break.



4. Caddy: Subdomain & HTTPS Setup

Modern browsers, especially on Android and iOS, refuse to run WebRTC or access camera/sensors over insecure http. You need a subdomain and SSL.





Point your subdomain, for example robot.yourdomain.com, to your Public IP.



Install Caddy.



Create a Caddyfile:

robot.yourdomain.com {
    reverse_proxy localhost:8889
}




Run caddy run. It will automatically handle Let's Encrypt SSL certificates for you.



5. Troubleshooting & Performance Analysis

Chrome WebRTC Internals

Open chrome://webrtc-internals on your mobile browser while streaming.





jitterBufferDelay: Should be stable around 0.1s. If it climbs, check if FFmpeg is dropping frames or if the bitrate is too high for your upload speed.



framesDecoded: Should increase steadily without massive gaps.

Common Fixes





Black Screen on Mobile: Usually a firewall issue. Ensure UDP 8189 is forwarded correctly.



Resumed reading with rate > 1.0: If FFmpeg shows this, remove any -re or -framerate flags from the input side. Let the camera clock dictate the speed.



🛠 Tools Used





MediaMTX



FFmpeg



Caddy Server



Created with passion for the Robot Footballer Project.

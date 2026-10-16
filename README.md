# RouterWatch — Tenda F3 monitor

RouterWatch is a starter Android app plus a small Python agent for an always-on Windows/Linux laptop.

## Important status
This bundle contains the Android project source and monitoring agent. It is **not a compiled APK**: the current build environment does not have the Android SDK/Gradle Android build components needed to compile and sign an APK. Build instructions are below.

## How it works
- The laptop runs `agent/routerwatch_agent.py` on the same LAN as the Tenda F3 routers.
- Add routers by name, MAC address and LAN IP in the Android app.
- The agent pings each IP, then checks the OS neighbour/ARP table for the configured MAC.
- The app polls the agent API and shows Online/Offline, last seen, and last check.
- For remote monitoring, connect the phone to the laptop through a private VPN such as Tailscale or WireGuard. Do not expose the agent port directly to the public internet.

A ping response alone is not sufficient to prove a specific MAC is present; RouterWatch checks the neighbour table as a best-effort identity check. Some operating systems may not retain an ARP entry, and some routers may block ICMP. In those cases the status can be Unknown/Offline even when the router is functioning. Validate with your specific F3 firmware and laptop OS.

## Run the agent (Python 3.10+)
1. Install Python 3.10 or later on the laptop.
2. From the `agent` directory run:
   `python routerwatch_agent.py`
3. Open `http://LAPTOP-LAN-IP:8765/health` from a browser on the same LAN.
4. Edit `agent/routers.json` to add routers, or use the app's Add Router form.
5. The laptop firewall must allow TCP 8765 from your trusted LAN/VPN only.

The agent binds to `0.0.0.0:8765` for LAN/VPN access. Restrict access with your host firewall and VPN. This prototype has no authentication or TLS; only use on a trusted private network/VPN. Do not port-forward it to the internet.

## Build the Android APK
Requirements: Android Studio (current stable), Android SDK Platform 35, JDK 17 or compatible Android Studio JDK.
1. Open the `android` folder in Android Studio.
2. Let Gradle sync and install any requested SDK components.
3. Build > Build Bundle(s) / APK(s) > Build APK(s).
4. The debug APK is normally created at `android/app/build/outputs/apk/debug/app-debug.apk`.

No APK is included because this environment has no Android SDK/build setup to compile and sign it.

## First-run configuration
- Start the agent on the laptop.
- In the app, set Agent URL to `http://192.168.x.x:8765` for local use.
- For remote use, set it to the laptop's private VPN address, e.g. `http://100.x.y.z:8765`.
- Ensure the app phone and laptop can reach one another over the selected LAN/VPN.

## API
- `GET /health`
- `GET /routers`
- `POST /routers` JSON: `{"name":"F3 router","mac":"AA:BB:CC:DD:EE:FF","ip":"192.168.0.1"}`
- `DELETE /routers?id=1`

This is a starter prototype, not a production-hardened monitoring service.

## Build without Android Studio (phone-only cloud build)
This project includes a GitHub Actions workflow at `.github/workflows/android-apk.yml`.
If you can get the source files into a GitHub repository, GitHub's cloud runner can build the debug APK:
1. Create a GitHub account/repository and put the contents of this project in the repository's `main` branch.
2. Open the repository's **Actions** tab and run **Build RouterWatch APK** (or push a commit to `main`).
3. Open the completed workflow run and download the `RouterWatch-debug-apk` artifact.
4. Extract the downloaded artifact ZIP on your phone; tap `app-debug.apk` to install. Android may ask you to allow installs from that browser/file manager.
This creates a debug APK, not a Play Store release. Keep the source in a private repository if desired.

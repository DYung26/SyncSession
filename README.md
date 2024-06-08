# SyncSession
## Project Overview
Welcome to SyncSession, a powerful Chrome extension designed to enhance online collaboration by synchronizing web sessions in real-time. This extension allows users to extract session data from a host's webpage, broadcast it to a WebSocket server, and share the session data among all members. Real-time text communication is also facilitated, ensuring seamless interaction across all connected sessions.
## Objectives
- Enables real-time sharing of web sessions among multiple users
- Facilitates synchronised browsing and interaction on a host’s webpage.
## Technology Stack
- **Front-end**: HTML, CSS, JavaScript
- **Back-end**: WebSocket server
## Architecture
- **Host Extraction**: Captures session data from the host's webpage.
- WebSocket Server: Facilitates real-time data broadcasting.
- Member Connection: Members retrieve session data and synchronize with the host.
- Real-time Chat(Text broadcast): Text messages are broadcast across all sessions instantly.
## Installation Instructions
### From Chrome Web Store
Coming soon...(Stay tuned!)
### Manual Installation
1.	Download the Extension Package
   - Download the ZIP file from [here](https://github.com/DYung26/SyncSession/raw/main/SyncSession.zip).
3.	Enable Developer Mode in Chrome
   - Go to chrome://extensions/.
  	- Enable "Developer mode" by toggling the switch in the top right corner.
5.	Load the Extension
	- Click on "Load unpacked" and select the directory where the extension files are located.
	- The extension will be installed and activated immediately.
## Demo and Screenshots
## Manual Installation of SyncSession
Check out this demo video for manual installation:
<div align="center">
  <a href="https://www.youtube.com/watch?v=0dHltlL0zK8">
    <img src="https://img.youtube.com/vi/0dHltlL0zK8/0.jpg" alt="Manual Installation" width="500"/>
  </a>
</div>

## Session Transfer on SyncSession
Check out this demo video on how SyncSession transfers a `https://app.imperial.learnchameleon.com/menu` session from one browser to another:
<div align="center">
  <a href="https://1drv.ms/v/s!Ag9jtTl2cMLljBxf5VCmd2otrQvy">
    <img src="https://img.youtube.com/vi/ibw97d61PZ4/0.jpg" alt="Session Transfer" width="500"/>
  </a>
</div>

SyncSession seamlessly transfers sessions from one browser to another. It also works well for cross-device synchronization, as SyncSession's WebSocket server will be hosted on [Chameleon](https://app.imperial.learnchameleon.com).

## Text Sync on SyncSession
**Browser1 (Host):** Starts a session and begins broadcasting text updates.

**Browser2 (Member):** Connects to the session, sends and receives real-time text updates from other members.

Check out this demo video on how SyncSession handles text synchronization across browsers:

<div align="center">
  <a href="https://1drv.ms/v/s!Ag9jtTl2cMLljB00JCvudsJSYvvy">
    <img src="https://img.youtube.com/vi/emU97nE_9VM/0.jpg" alt="Watch the video" width="560" height="315"/>
  </a>
</div>
SyncSession ensures real-time text synchronization across all connected devices, with the WebSocket server to be hosted on Chameleon.

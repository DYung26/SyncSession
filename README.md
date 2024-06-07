# SyncSession
## Introduction
## Project Overview
Welcome to SyncSession, a powerful Chrome extension designed to enhance online collaboration by synchronizing web sessions in real-time. This extension allows users to extract session data from a host's webpage, broadcast it to a WebSocket server, and share the session data among all members. Real-time text communication is also facilitated, ensuring seamless interaction across all connected sessions.
## Objectives
- Enables real-time sharing of web sessions among multiple users
- Facilitates synchronised browsing and interaction on a host’s webpage.
## Demo and Screenshots
## Live Demo
## Installation Instructions
## From Chrome Web Store
Coming soon...
## Manual Installation
1.	Download the Extension Package
   - Download the ZIP file from here.
3.	Enable Developer Mode in Chrome
   - Go to chrome://extensions/.
  	- Enable "Developer mode" by toggling the switch in the top right corner.
5.	Load the Extension
	- Click on "Load unpacked" and select the directory where the extension files are located.
	- The extension will be installed and activated immediately.
## Technical Details
## Technology Stack
- **Front-end**: HTML, CSS, JavaScript
- **Back-end**: WebSocket server
Architecture
- **Host Extraction**: Captures session data from the host's webpage.
- WebSocket Server: Facilitates real-time data broadcasting.
- Member Connection: Members retrieve session data and synchronize with the host.
- Real-time Chat(Text broadcast): Text messages are broadcast across all sessions instantly.
## Core Components
1.	manifest.json
```json
{
    "manifest_version": 3,
    "name": "Collaborative Workspace Sharing",
    "version": "1.0",
    "description": "Share your session on the platform with others.",
    "permissions": [
      "cookies",
      "activeTab",
      "storage",
      "webRequest"
    ],
    "host_permissions": [
      "https://app.imperial.learnchameleon.com/*",
      "https://api.imperial.learnchameleon.com/*"
    ],
    "background": {
      "service_worker": "background.js"
    },
    "action": {
      "default_popup": "popup.html",
      "default_icon": {
        "16": "icons/1.jpg",
        "48": "icons/2.jpg",
        "128": "icons/3.jpg"
      }
    },
    "content_scripts": [
      {
        "matches": ["https://app.imperial.learnchameleon.com/*"],
        "js": ["content.js", "content1.js"]
      }
    ],
    "icons": {
      "16": "icons/1.jpg",
      "48": "icons/2.jpg",
      "128": "icons/3.jpg"
    }
  }
```
2. background.js
```js
// background.js

let socket;
let isSocketConnected = false;
let messageQueue = [];

// Connect to the WebSocket server
function connectWebSocket() {
  socket = new WebSocket('ws://localhost:8080');

  socket.onopen = () => {
    console.error('WebSocket connection opened.');
    isSocketConnected = true;
    // Process any queued messages
    while (messageQueue.length > 0) {
      const { data, sessionId, browserId } = messageQueue.shift();
      sendData(data, sessionId, browserId);
    }
  };

  socket.onmessage = (event) => {
    const message = JSON.parse(event.data);
    if (message.type === 'TEXT_UPDATE') {
      // Relay the message to the content script
      chrome.runtime.sendMessage(message, (response) => { // (tabs[0].id, message, (response) => {
        if (chrome.runtime.lastError) {
          console.error('Error sending message to content script:', chrome.runtime.lastError);
        }
      });
    }
  };

  socket.onerror = (error) => {
    console.error('WebSocket error:', error);
    isSocketConnected = false;
  };

  socket.onclose = () => {
    console.log('WebSocket connection closed.');
    isSocketConnected = false;
  };
}

// Send data through the WebSocket
function sendData(data, sessionId, browserId) {
  if (isSocketConnected) {
    socket.send(JSON.stringify({ type: 'TEXT_CARET', data: data, sessionId: sessionId, browserId: browserId }));
  } else {
    messageQueue.push({ data, sessionId, browserId });
    if (socket.readyState === WebSocket.CLOSED || socket.readyState === WebSocket.CLOSING) {
      connectWebSocket();
    }
  }
}

// Listen for messages from content scripts
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'TEXT_CARET') {
    const { data, sessionId, browserId } = message;
    sendData(data, sessionId, browserId);
    connectWebSocket();
    sendResponse({ status: 'success' });
  }
});

// Connect WebSocket on script load
connectWebSocket();


// Listen for messages from the popup window
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'SET_SESSION_DATA') {
    const sessionData = message.data;
    const sessionToken = sessionData[0].sessionToken;
    const userId = sessionData[0].userId;
    const workspaceId = sessionData[0].workspaceId;
    const accessToken = sessionData[0].accessToken;
    // Set localStorage items
    if (sessionToken) {
        chrome.storage.local.set({ 'sessionToken': sessionToken });
    }
    if (userId) {
        chrome.storage.local.set({ 'userId': userId });
    }
    if (workspaceId) {
        chrome.storage.local.set({ 'workspaceId': workspaceId });
    }

    // Set the accessToken cookie
    chrome.cookies.set({
        url: 'https://api.imperial.learnchameleon.com/menu',
        name: 'accessToken',
        value: accessToken,
        domain: 'api.imperial.learnchameleon.com',
        expirationDate: Math.floor((Date.now() + 86400000) / 1000), // Expires in 1 day
        secure: true,
        httpOnly: true,
        sameSite: 'strict'
    }, (cookie) => {
        if (cookie) {
            sendResponse({ status: 'success' });
        } else {
            sendResponse({ status: 'error' });
        }
    });

    // Indicate that we will asynchronously respond
    return true;
  }
});

// Listen for messages from the popup window
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'UNSET_SESSION_DATA') {
    // Set localStorage items
    chrome.storage.local.remove('sessionToken');
    chrome.storage.local.remove('userId');
    chrome.storage.local.remove('workspaceId');
    // Set the accessToken cookie
    chrome.cookies.remove({
      url: 'https://api.imperial.learnchameleon.com/menu',
      name: 'accessToken'
    }, (cookie) => {
        if (cookie) {
            sendResponse({ status: 'success' });
        } else {
            sendResponse({ status: 'error' });
        }
    });
    // Indicate that it will asynchronously respond
    return true;
  }
});
```
4. content1.js
```js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'TEXT_SYNC') {
    const inputHandler = (event) => {
      if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA') {
        const data = {
          type: event.target.tagName,
          value: event.target.value,
          id: event.target.id,
          class: event.target.className,
          name: event.target.name,
          timestamp: Date.now()
        };
        data.targetIdentifier = getTargetIdentifier(event.target);
        const sessionId = message.sessionId;
        const browserId = message.browserId;
        sendData(data, sessionId, browserId);
      }
    };
    document.addEventListener('input', inputHandler);
    sendResponse({ status: 'success' });
  }
  // console.error("okkk");
  if (message.type === 'TEXT_UPDATE') {
    writeDataToPage(message.data);
  }
  return true;
});

function sendData(data, sessionId, browserId) {
  chrome.runtime.sendMessage({
    type: 'TEXT_CARET',
    data: data,
    sessionId: sessionId,
    browserId: browserId
  }, (response) => {
    if (response && response.status === 'success') {
      console.log('Text message sent successfully.');
    } else {
      console.error('Failed to send text message.');
    }
  });
}

function getTargetIdentifier(target) {
  if (target.id) {
    return `#${target.id}`;
  } else if (target.name) {
    return `[name="${target.name}"]`;
  } else if (target.className) {
    return `.${target.className.split(' ').join('.')}`;
  } else {
    return target.tagName.toLowerCase();
  }
}
  
function writeDataToPage(data) {
  let targetInput;
  if (data.targetIdentifier.startsWith('#')) {
    targetInput = document.querySelector(data.targetIdentifier);
  } else if (data.targetIdentifier.startsWith('.')) {
    targetInput = document.querySelector(data.targetIdentifier);
  } else if (data.targetIdentifier.startsWith('[name=')) {
    targetInput = document.querySelector(data.targetIdentifier);
  } else {
    targetInput = document.querySelector(data.targetIdentifier);
  }
  
  if (targetInput) {
    targetInput.value = data.value;
  } else {
    console.warn('Target input not found');
  }
}
```
## Usage Instructions
1. Starting a Session
   - Click on the extension icon
   - If 

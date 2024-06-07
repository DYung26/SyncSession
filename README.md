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
3. content.js
```js
// content.js


function generateSessionId() {
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  const length = 10;
  let sessionId = '';
  for (let i = 0; i < length; i++) {
    sessionId += characters.charAt(Math.floor(Math.random() * characters.length));
  }
  return sessionId;
}

// Function to update caret and textbox position
function updateCaretPosition(input, customCaret, floatingTextbox) {
  const caretPos = input.selectionStart;
  const rect = input.getBoundingClientRect();
  const textBeforeCaret = input.value.slice(0, caretPos);
  const span = document.createElement('span');
  span.style.visibility = 'hidden';
  span.style.whiteSpace = 'pre';
  span.textContent = textBeforeCaret;
  document.body.appendChild(span);

  const spanRect = span.getBoundingClientRect();
  const caretTop = rect.top + window.scrollY;
  const caretLeft = rect.left + spanRect.width + window.scrollX;

  customCaret.style.top = `${caretTop}px`;
  customCaret.style.left = `${caretLeft}px`;

  floatingTextbox.style.top = `${caretTop - 30}px`;
  floatingTextbox.style.left = `${caretLeft}px`;

  document.body.removeChild(span);
}

// Attach event listeners to input elements
function attachCaret(input, socket, sessionId, browserId) {
  const customCaret = document.createElement('div');
  customCaret.id = 'customCaret';
  customCaret.style.position = 'absolute';
  customCaret.style.width = '2px';
  customCaret.style.height = '20px';
  customCaret.style.backgroundColor = 'blue';
  customCaret.style.zIndex = '1000';
  document.body.appendChild(customCaret);

  const floatingTextbox = document.createElement('input');
  floatingTextbox.id = 'floatingTextbox';
  floatingTextbox.type = 'text';
  floatingTextbox.style.position = 'absolute';
  floatingTextbox.style.width = '100px';
  floatingTextbox.style.zIndex = '1001';
  document.body.appendChild(floatingTextbox);

  input.addEventListener('input', () => {
    updateCaretPosition(input, customCaret, floatingTextbox);

    const text = input.value;
    const caretColor = getRandomColor();

    if (socket.readyState === WebSocket.OPEN) {
      const message = JSON.stringify({ type: 'TEXT_CARET', data: { sessionId, browserId, text, caretColor } });
      socket.send(message);
    }
  });

  input.addEventListener('click', () => {
    updateCaretPosition(input, customCaret, floatingTextbox);
  });
  input.addEventListener('keyup', () => {
    updateCaretPosition(input, customCaret, floatingTextbox);
  });
  input.addEventListener('keydown', () => {
    updateCaretPosition(input, customCaret, floatingTextbox);
  });
}

// Function to extract session data
function getSessionData() {
    // Example: Extracting session token and user ID from localStorage
    const sessionToken = localStorage.getItem('sessionToken'); // Assuming the session token is stored in localStorage
    const userId = localStorage.getItem('userId'); // Assuming the user ID is stored in localStorage
    // Extracting workspace ID from the URL
    const workspaceId = window.location.pathname.split('/').pop(); // Assuming the workspace ID is part of the URL
    // Create session data object
    // Retrieve HttpOnly cookies and add them to sessionData
    const accessToken = getCookie('accessToken', (accessToken) => {
      sessionData.accessToken = accessToken;
      console.log('Session data received:', sessionData);
    });
    const sessionData = {
      sessionToken,
      userId,
      workspaceId,
      accessToken,
    };

    return sessionData;
}

// Function to get the value of a specific cookie
function getCookie(name, callback) {
  chrome.cookies.get({ url: 'https://api.imperial.learnchameleon.com/menu', name: name }, (cookie) => {
    if (cookie) {
      console.log('Cookie value:', cookie.value);
      callback(cookie.value);
    } else {
        callback(null);
    }
  });
}

function getOrCreateBrowserId() {
  let browserId = localStorage.getItem('browserId');
  if (!browserId) {
    browserId = 'browser_' + Math.random().toString(36).substr(2, 9);
    localStorage.setItem('browserId', browserId);
  }
  return browserId;
}

/*const browserId = getOrCreateBrowserId();*/

function getRandomColor() {
  // Generate random RGB values
  const r = Math.floor(Math.random() * 256);
  const g = Math.floor(Math.random() * 256);
  const b = Math.floor(Math.random() * 256);
  // Construct color string in hex format
  return '#' + r.toString(16).padStart(2, '0') + g.toString(16).padStart(2, '0') + b.toString(16).padStart(2, '0');
}

function renderCursor(cursorData, browserId) {
  // Create or update cursor element with respective color
  let cursorElement = document.getElementById(cursorData.browserId);
  if (!cursorElement) {
    cursorElement = document.createElement('div');
    cursorElement.id = cursorData.browserId;
    document.body.appendChild(cursorElement);
  }
  cursorElement.style.position = 'absolute';
  cursorElement.style.left = cursorData.x + 'px';
  cursorElement.style.top = cursorData.y + 'px';
  cursorElement.style.width = '10px';
  cursorElement.style.height = '10px';
  cursorElement.style.backgroundColor = cursorData.color;
}
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

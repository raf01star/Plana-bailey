<div align='center'>HT-baileys</div>

<div align="center">
  <img src="https://files.catbox.moe/vi3npg.png" alt="HT-baileys Banner" width="100%" />
  <br/><br/>

  <a href="https://heavstal-tech.vercel.app">
    <img src="https://img.shields.io/badge/Maintainer-Heavstal_Tech-007acc?style=for-the-badge&logo=vercel&logoColor=white" alt="Heavstal Tech" />
  </a>
  <a href="https://www.npmjs.com/package/@heavstaltech/baileys">
    <img src="https://img.shields.io/badge/NPM-@heavstaltech/baileys-cb3837?style=for-the-badge&logo=npm&logoColor=white" alt="NPM" />
  </a>
  <a href="https://github.com/HeavstalTech/HT-baileys">
    <img src="https://img.shields.io/badge/Version-1.0.1-2ea44f?style=for-the-badge&logo=github&logoColor=white" alt="Version" />
  </a>
  <a href="https://github.com/HeavstalTech/HT-baileys/blob/master/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-fab005?style=for-the-badge" alt="License" />
  </a>
</div>

---

## Introduction

**HT-baileys** is an advanced fork of the WhatsApp Web API library, engineered and maintained by **Heavstal Tech**.

This library builds upon the stability of the original Baileys architecture while integrating extended features required for modern, production-grade automation. It provides a robust solution for enterprise bots, featuring optimized connection headers, custom pairing code logic, and native support for WhatsApp Channels (Newsletters), Interactive Messages, and more.

WhatsApp Baileys is an open-source library designed to help developers build automation solutions and integrations with WhatsApp efficiently and directly. Using websocket technology without the need for a browser, this library supports a wide range of features such as message management, chat handling, group administration, as well as interactive messages and action buttons for a more dynamic user experience.

Actively developed and maintained, baileys continuously receives updates to enhance stability and performance. One of the main focuses is to improve the pairing and authentication processes to be more stable and secure. Pairing features can be customized with your own codes, making the process more reliable and less prone to interruptions.

This library is highly suitable for building business bots, chat automation systems, customer service solutions, and various other communication automation applications that require high stability and comprehensive features. With a lightweight and modular design, baileys is easy to integrate into different systems and platforms.

## Features Overview

| Feature | Description |
| :--- | :--- |
| 💬 **Newsletters & Channels** | Full support for sending text, media, and managing WhatsApp Channels. |
| 🔘 **Interactive Messages** | Native support for buttons, lists, carousel, and product messages. |
| 🤖 **AI Message Icon** | Customize message appearances with an optional AI icon (`ai: true`). |
| 🔑 **Custom Pairing Codes** | **Heavstal Exclusive:** Define custom alphanumeric pairing codes (e.g., `HEAVSTAL`). |
| 📊 **Polls & Events** | Create polls with vote tracking and schedule WhatsApp events. |
| 🖼️ **Albums & High-Res Media** | Send album messages and upload full-size profile pictures. |
| 🛠️ **Stability Fixes** | Enhanced Libsignal logs and connection management. |

---

## Installation

### Stable Release
```bash
npm install @heavstaltech/baileys
```

### Edge Release
```bash
npm install @heavstaltech/baileys@latest
```

---

## Getting Started

### Basic Connection

The following example demonstrates how to initialize a socket connection using `HT-baileys`.

```ts
import makeWASocket, { 
    useMultiFileAuthState, 
    DisconnectReason, 
    Browsers 
} from '@heavstaltech/baileys'

async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info')

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false, // Set to true if using QR scanning
        browser: Browsers.macOS("Desktop"),
        syncFullHistory: true
    })

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update
        
        if(connection === 'close') {
            const shouldReconnect = (lastDisconnect?.error as any)?.output?.statusCode !== DisconnectReason.loggedOut
            console.log('Connection closed. Reconnecting:', shouldReconnect)
            
            if(shouldReconnect) {
                connectToWhatsApp()
            }
        } else if(connection === 'open') {
            console.log('Connection opened successfully')
        }
    })

    sock.ev.on('creds.update', saveCreds)
}

connectToWhatsApp()
```

---

## Core Feature Implementation

### 1. Custom Pairing Code
HT-baileys supports the definition of custom alphanumeric pairing codes, allowing for branded connection flows.

```ts
if (!sock.authState.creds.registered) {
    // Ensure the number format is correct (Country Code + Number)
    const phoneNumber = "2349000000000"
    
    // Define your branded pairing code
    const customCode = "HEAVSTAL" 
    
    setTimeout(async () => {
        const code = await sock.requestPairingCode(phoneNumber, customCode)
        console.log(`Pairing Code: ${code}`)
    }, 3000)
}
```

### 2. Utilities
Useful helper functions included in the library.

**Check Banned Number**
```javascript
await sock.checkWhatsApp(jid)
```

### 3. Newsletter Management
Tools for managing and interacting with WhatsApp Channels.

**Get Channel ID by URL**
```javascript
await sock.newsletterId(url)
```

**Create & Update Channel**
```ts
const channel = await sock.newsletterCreate("Heavstal Updates", "Official Channel Description")

// Update Metadata
await sock.newsletterUpdateName(channel.id, "Heavstal Tech")
await sock.newsletterUpdateDescription(channel.id, "Official Updates")
```

---

## Advanced Messaging Documentation

### Group Status V2
Send a status update specific to a group context.

```javascript
await sock.sendMessage(jid, {
     groupStatusMessage: {
          text: "Hello World"
     }
});
```

### Album Message
Send multiple images combined into a single album bubble.

```javascript
await sock.sendMessage(jid, { 
    albumMessage: [
        { image: { url: "https://example.com/1.jpg" }, caption: "First Photo" },
        { image: { url: "https://example.com/2.jpg" }, caption: "Second Photo" }
    ] 
}, { quoted: m });
```

### Event Message
Create and send a WhatsApp Event invitation.

```javascript
await sock.sendMessage(jid, { 
    eventMessage: { 
        isCanceled: false, 
        name: "Heavstal Launch", 
        description: "Official Launch Party", 
        location: { 
            degreesLatitude: 0, 
            degreesLongitude: 0, 
            name: "Virtual" 
        }, 
        joinLink: "https://call.whatsapp.com/video/example", 
        startTime: "1763019000", 
        endTime: "1763026200", 
        extraGuestsAllowed: false 
    } 
}, { quoted: m });
```

### Poll Message
Send a poll and display results.

```javascript
await sock.sendMessage(jid, { 
    pollResultMessage: { 
        name: "Which framework do you prefer?", 
        pollVotes: [
            { optionName: "React", optionVoteCount: "10" },
            { optionName: "Vue", optionVoteCount: "5" }
        ] 
    } 
}, { quoted: m });
```

### Product Message
Send a catalog product message with a "Buy Now" button.

```javascript
await sock.sendMessage(jid, {
    productMessage: {
        title: "Premium Service",
        description: "Lifetime access to premium features",
        thumbnail: { url: "https://example.com/image.jpg" },
        productId: "PROD001",
        retailerId: "RETAIL001",
        url: "https://example.com/product",
        body: "Product Details",
        footer: "Special Offer",
        priceAmount1000: 50000,
        currencyCode: "USD",
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Buy Now",
                    url: "https://example.com/buy"
                })
            }
        ]
    }
}, { quoted: m });
```

### Request Payment Message
Send a payment request with a custom background and sticker.

```javascript
// Assuming 'm' is the message object you are quoting
let quotedType = m.quoted?.mtype || '';
let quotedContent = JSON.stringify({ [quotedType]: m.quoted }, null, 2);

await sock.sendMessage(jid, {
    requestPaymentMessage: {
        currency: "IDR",
        amount: 10000000,
        from: m.sender,
        sticker: JSON.parse(quotedContent),
        background: {
            id: "100",
            fileLength: "0",
            width: 1000,
            height: 1000,
            mimetype: "image/webp",
            placeholderArgb: 0xFF00FFFF,
            textArgb: 0xFFFFFFFF,     
            subtextArgb: 0xFFAA00FF   
        }
    }
}, { quoted: m });
```

### Interactive Messages (Native Flow)
Complex interactive messages including buttons, copy-to-clipboard, and lists.

**Native Flow with Buttons & Lists:**
```javascript
await sock.sendMessage(jid, {    
    interactiveMessage: {      
        header: "Heavstal Tech",
        title: "Interactive Menu",      
        footer: "Powered by HT-baileys",      
        image: { url: "https://example.com/image.jpg" },      
        nativeFlowMessage: {        
            messageParamsJson: JSON.stringify({          
                limited_time_offer: {            
                    text: "Special Offer",            
                    url: "https://heavstal-tech.vercel.app",            
                    copy_code: "DISCOUNT20",            
                    expiration_time: Date.now() + 86400000          
                },               
            }),        
            buttons: [          
                {            
                    name: "single_select",            
                    buttonParamsJson: JSON.stringify({              
                        title: "Open Menu",              
                        sections: [                
                            {                  
                                title: "Main Options",                  
                                highlight_label: "Popular",                  
                                rows: [                    
                                    {                      
                                        title: "Check Status",                      
                                        description: "View system status",                      
                                        id: "status_row"                    
                                    }                  
                                ]                
                            }              
                        ]            
                    })          
                },          
                {            
                    name: "cta_copy",            
                    buttonParamsJson: JSON.stringify({              
                        display_text: "Copy ID",              
                        id: "123456789",              
                        copy_code: "HT-1234"            
                    })          
                }        
            ]      
        }    
    }  
}, { quoted: m });
```

**Interactive Message with Document:**
*Note: Documents must be passed as buffers.*

```javascript
import fs from 'fs';

await sock.sendMessage(jid, {
    interactiveMessage: {
        header: "Documentation",
        title: "Read Guidelines",
        footer: "Heavstal Tech",
        document: fs.readFileSync("./manual.pdf"),
        mimetype: "application/pdf",
        fileName: "manual.pdf",
        jpegThumbnail: fs.readFileSync("./thumb.jpg"),
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Visit Website",
                    url: "https://heavstal-tech.vercel.app",
                    merchant_url: "https://heavstal-tech.vercel.app"
                })
            }
        ]
    }
}, { quoted: m });
```

---

## About Heavstal Tech

**Heavstal Tech** is a forward-thinking technology organization dedicated to building robust tools and ecosystems for the modern web. From automation libraries to full-stack applications, we prioritize performance, scalability, and developer experience.

🌐 **Official Website:** [https://heavstal-tech.vercel.app](https://heavstal-tech.vercel.app)

## Disclaimer

**HT-baileys** is an independent project maintained by **Heavstal Tech**. It is not affiliated with, authorized, maintained, sponsored, or endorsed by WhatsApp Inc. or Meta Platforms, Inc.

This software is provided "as is", without warranty of any kind. Users are responsible for ensuring their usage complies with WhatsApp's Terms of Service.

---

<div align="center">
  <sub>© 2025 - 2026 Heavstal Tech. All rights reserved.</sub>
</div>

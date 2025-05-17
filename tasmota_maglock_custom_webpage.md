# Tasmota Mag Lock Control with Custom Web Page

## Overview

This document explains how to use a Shelly device flashed with Tasmota to control a mag lock, and how to serve a simple custom web page for lock/unlock control on the local network. It covers both the scripting (Berry) and firmware modification (C++) approaches, with a focus on embedding a custom HTML page in the Tasmota firmware.

---

## Table of Contents
1. [Berry in Tasmota](#1-berry-in-tasmota)
2. [Tasmota Web UI and Custom Pages](#2-tasmota-web-ui-and-custom-pages)
3. [Serving a Custom HTML Page (C++ Approach)](#3-serving-a-custom-html-page-c-approach)
    - [Step-by-Step Guide](#step-by-step-guide)
    - [Advanced Example: Showing Lock Status](#advanced-example-showing-lock-status)
4. [Limitations](#4-limitations)
5. [Security](#5-security)
6. [Further Expansion](#6-further-expansion)
7. [Troubleshooting](#7-troubleshooting)
8. [Alternatives](#8-alternatives)
9. [References](#9-references)

---

## 1. Berry in Tasmota
- **Berry** is a lightweight scripting language integrated into Tasmota.
- It allows automation and custom logic, but **cannot serve custom HTML pages**—it is for logic only.
- [Berry Documentation](https://tasmota.github.io/docs/Berry/)

---

## 2. Tasmota Web UI and Custom Pages
- Tasmota has a built-in web server for device control/configuration, accessible on the local LAN.
- The default web UI allows toggling relays (e.g., for a mag lock).
- **You cannot simply upload an HTML file** to Tasmota; the web UI is generated from C/C++ code.
- For advanced customization, you must edit and recompile the firmware.

---

## 3. Serving a Custom HTML Page (C++ Approach)

### Step-by-Step Guide

#### **Step 1: Get the Tasmota Source Code**
1. Clone the Tasmota repository:
   ```sh
   git clone https://github.com/arendst/Tasmota.git
   ```
2. Open the project in your preferred IDE (e.g., VS Code, PlatformIO, Arduino IDE).

#### **Step 2: Locate the Web Server Handler**
- The main web server logic is in `tasmota/xdrv_01_webserver.ino`.
- Look for the function that handles HTTP requests. In recent Tasmota versions, this is typically `HandleRoot()` or a similar function that processes web requests.

#### **Step 3: Add Your Custom Endpoint**
1. Find a place to insert your code:
   - Search for existing endpoint checks, e.g., `if (webserver->uri() == F("/")) { ... }`
   - Insert your custom check before the default handler or 404 handler.
2. Insert the custom HTML handler and PIN management logic:

```cpp
#define NUM_PINS 4
#define PIN_LENGTH 4
#define MASTER_CODE_LENGTH 8
const char* master_code = "12345678"; // Change this to your desired master code

// Helper to get a PIN from settings
String get_pin(int idx) {
    char pin[PIN_LENGTH + 1] = {0};
    memcpy(pin, Settings->free + idx * PIN_LENGTH, PIN_LENGTH);
    return String(pin);
}

// Helper to set a PIN in settings
void set_pin(int idx, const String& value) {
    for (int i = 0; i < PIN_LENGTH; i++) {
        Settings->free[idx * PIN_LENGTH + i] = (i < value.length()) ? value[i] : 0;
    }
    SettingsSave(1); // Save settings to flash
}

// Check if a PIN is valid
bool is_valid_pin(const String& pin) {
    if (pin.length() != PIN_LENGTH) return false;
    for (int i = 0; i < NUM_PINS; i++) {
        if (pin == get_pin(i)) return true;
    }
    return false;
}

// Web handler for /lock
if (webserver->uri() == F("/lock")) {
    // Handle POST for PIN management
    if (webserver->method() == HTTP_POST) {
        String master = webserver->arg("master");
        int slot = webserver->hasArg("slot") ? webserver->arg("slot").toInt() : -1;
        String newpin = webserver->arg("newpin");
        if (master == master_code && slot >= 0 && slot < NUM_PINS && newpin.length() == PIN_LENGTH) {
            set_pin(slot, newpin);
            webserver->send(200, "text/html", "PIN updated! <a href='/lock'>Back</a>");
        } else {
            webserver->send(200, "text/html", "Invalid input or master code! <a href='/lock'>Back</a>");
        }
        return;
    }
    // Handle GET for unlocking
    if (webserver->hasArg("pin")) {
        String pin = webserver->arg("pin");
        if (is_valid_pin(pin)) {
            ExecuteCommand("Power1 1", SRC_WEBGUI);
            webserver->send(200, "text/html", "Unlocked! <a href='/lock'>Back</a>");
        } else {
            webserver->send(200, "text/html", "Invalid PIN! <a href='/lock'>Back</a>");
        }
        return;
    }
    // Serve the PIN entry and management form
    String html = F(
        "<h2>Mag Lock Control</h2>"
        "<form method='get'>"
        "<label>Enter PIN:</label> <input type='password' name='pin' maxlength='4' pattern='\\d{4}' required />"
        "<input type='submit' value='Unlock' />"
        "</form>"
        "<hr>"
        "<h3>PIN Management (Admin Only)</h3>"
        "<form method='post'>"
        "<label>Master Code:</label> <input type='password' name='master' maxlength='8' pattern='\\d{8}' required />"
        "<label>Change PIN slot (0-3):</label> <input type='number' name='slot' min='0' max='3' required />"
        "<label>New PIN:</label> <input type='password' name='newpin' maxlength='4' pattern='\\d{4}' required />"
        "<input type='submit' value='Set PIN' />"
        "</form>"
        "<hr>"
        "<b>Current PINs (for admin reference, remove in production):</b><ul>"
    );
    for (int i = 0; i < NUM_PINS; i++) {
        html += "<li>Slot " + String(i) + ": " + get_pin(i) + "</li>";
    }
    html += "</ul>";
    webserver->send(200, "text/html", html);
    return;
}
```

**Enhancements and Best Practices:**
- Added input validation (length and digit pattern) for PINs and master code.
- Improved user feedback for errors and success.
- Display current PINs for admin reference (remove this in production).
- All logic is robust and checks for valid input before updating settings.

#### **Step 4: Recompile and Flash Tasmota**
1. Compile the firmware using PlatformIO or your chosen tool.
2. Flash the firmware to your Shelly device.

#### **Step 5: Access Your Custom Page**
- On your local network, open a browser and go to:  
  `http://<device-ip>/lock`
- You should see your custom Mag Lock Control page.

#### **Step 6: (Optional) Move Code to a Custom Module**
- For maintainability, consider creating a new module (e.g., `xdrv_99_lockcontrol.ino`) and register your handler there.
- Follow Tasmota's [custom driver/module documentation](https://tasmota.github.io/docs/Custom-Drivers/) for guidance.

---

### Advanced Example: Showing Lock Status

Here's a more advanced snippet that fetches and displays the lock status:

```cpp
if (webserver->uri() == F("/lock")) {
    String html = F(
        "<!DOCTYPE html>"
        "<html><head><title>Mag Lock Control</title></head><body>"
        "<h1>Mag Lock Control</h1>"
        "<div id='status'>Status: ...</div>"
        "<button onclick=\"toggleLock()\">Toggle Lock</button>"
        "<script>"
        "function updateStatus() {"
        "  fetch('/cm?cmnd=Power').then(r => r.text()).then(t => {"
        "    document.getElementById('status').innerText = 'Status: ' + t;"
        "  });"
        "}"
        "function toggleLock() {"
        "  fetch('/cm?cmnd=Power%20TOGGLE').then(updateStatus);"
        "}"
        "updateStatus();"
        "</script>"
        "</body></html>"
    );
    webserver->send(200, "text/html", html);
    return;
}
```
- This page shows the current lock status and updates it after toggling.

---

## 4. Limitations
- Tasmota does **not** support uploading or serving arbitrary HTML files.
- All web content must be embedded in the firmware source.
- Berry scripting is for logic/automation, not UI.
- For static file serving, consider alternatives like ESPHome.

---

## 5. Security
- The web UI is only accessible on the local network unless you open ports (not recommended).
- Use Tasmota's web authentication for extra security.
- You can add authentication checks to your custom endpoint if needed.
- **Do not display PINs in production.** The example above shows PINs for admin reference only—remove this for real deployments.
- Always change the master code from the default before flashing.

---

## 6. Further Expansion

Below are several ways to expand and enhance your Tasmota mag lock project:

### 6.1 Show Lock Status
**Definition:** Display the current state of the mag lock (locked/unlocked) on your custom web page.

**How to implement:**
- Use JavaScript to fetch the relay state from Tasmota's `/cm?cmnd=Power` endpoint.
- Update the web page dynamically to reflect the current status.

**Example:** See the [Advanced Example: Showing Lock Status](#advanced-example-showing-lock-status) above for a full code snippet.

---

### 6.2 Add Authentication
**Definition:** Restrict access to your custom lock control page so only authorized users can operate the mag lock.

**How to implement:**
- Enable Tasmota's built-in web authentication via the device's web UI (Configuration > Configure Other > Web Admin Password).
- Optionally, add authentication checks in your custom endpoint handler in C++:

```cpp
if (!HttpCheckPriviledgedAccess()) {
    return; // User is not authenticated
}
```
- This ensures only authenticated users can access the `/lock` page.

---

### 6.3 Use Berry for Automation
**Definition:** Add automation logic, such as auto-relocking after a set time, using Berry scripting.

**How to implement:**
- Write a Berry script that listens for unlock events and triggers a timer to re-lock after a delay.
- Upload the script via the Tasmota web UI (Console > Berry Script).

**Example:**
```berry
def on_unlock():
    print("Unlocked, will re-lock in 10 seconds")
    timer.set(10, lambda() => power(1))  # Re-lock after 10 seconds

event.power_on = on_unlock
```
- This script will automatically re-lock the mag lock 10 seconds after it is unlocked.

---

### 6.4 Move to a Custom Module
**Definition:** For larger or more complex projects, create a dedicated `.ino` file (custom module) for your lock control logic.

**How to implement:**
- Create a new file in the Tasmota `tasmota` directory, e.g., `xdrv_99_lockcontrol.ino`.
- Register your module and handler following the [Tasmota Custom Drivers/Modules documentation](https://tasmota.github.io/docs/Custom-Drivers/).
- This keeps your code organized and maintainable, especially if you plan to add more features.

**Example structure:**
```cpp
// xdrv_99_lockcontrol.ino
#include <Tasmota.h>

void HandleLockPage() {
    // Your custom endpoint logic here
}

void RegisterLockControl() {
    // Register your handler with the web server
}

// Add initialization and registration code as per Tasmota's custom module guidelines
```

---

### 6.5 Add a Master 8-Digit Code for PIN Management
**Definition:** Require a master 8-digit code to add or change any of the 4-digit user PINs. This adds an extra layer of security for PIN management.

**How to implement:**
- Hardcode a master 8-digit code in your firmware (e.g., `const char* master_code = "12345678";`).
- Only allow adding or changing 4-digit PINs if the correct master code is provided.
- The master code is not used for unlocking, only for PIN management.

**How it works:**
- To unlock: Enter a 4-digit PIN as before.
- To add/change a PIN: Enter the master 8-digit code, the slot (0–3), and the new 4-digit PIN. Only if the master code is correct will the PIN be updated.

**Security Note:**
- The master code is hardcoded in the firmware. Change it before compiling and flashing.
- For even more security, you could obfuscate or encrypt the master code, but for most LAN use cases, this is sufficient.

---

## How Many PIN Codes Can Be Stored?

On a Shelly 1 (ESP8266) running Tasmota, PIN codes are typically stored in the `Settings->free[]` array, which is a block of user-available bytes in Tasmota's settings.

- **Default size:** 32 bytes (sometimes 64 bytes in newer Tasmota versions)
- **Each PIN:** 4 digits (4 bytes if stored as ASCII)

**Calculation:**
- 32 bytes / 4 bytes per PIN = 8 PIN codes
- 64 bytes / 4 bytes per PIN = 16 PIN codes (if available)

| Settings Free Bytes | PIN Length | Max PINs Storable |
|---------------------|------------|-------------------|
| 32                  | 4          | 8                 |
| 64                  | 4          | 16                |

**Note:**
- You may want to reserve some of `Settings->free[]` for other custom settings or future use.
- If you use a master code or other data, subtract that from the available space.
- Check the size of `Settings->free[]` in the Tasmota source (`tasmota.h` or `settings.h`).

---

## 7. Troubleshooting

**Common Issues and Solutions:**

- **PINs not saving after reboot:**
  - Ensure you are using `SettingsSave(1);` after changing a PIN.
  - Check that you are not exceeding the available space in `Settings->free[]`.

- **Unlock not working:**
  - Make sure the relay is correctly mapped to `Power1` in your Tasmota configuration.
  - Check that the entered PIN matches exactly (4 digits, no spaces).

- **Cannot change PINs:**
  - Verify the master code is correct and matches the hardcoded value.
  - Ensure all form fields are filled and valid (slot 0–3, 4-digit PIN, 8-digit master code).

- **Web page not loading:**
  - Confirm your custom endpoint is correctly registered and there are no syntax errors in your code.
  - Check device IP and network connectivity.

- **Security warning:**
  - Do not display PINs in production. Remove the PIN list from the HTML for real deployments.
  - Always use a strong, unique master code.

---

## 8. Alternatives
- **ESPHome**: Easier static web serving with the `web_server` component.
- **External Web Server**: Host a custom page elsewhere and use HTTP requests to control Tasmota.

---

## 9. References
- [Tasmota Berry Scripting](https://tasmota.github.io/docs/Berry/)
- [Tasmota Source Code](https://github.com/arendst/Tasmota)
- [ESPHome Web Server](https://esphome.io/components/web_server.html)
- [Tasmota Custom Drivers/Modules](https://tasmota.github.io/docs/Custom-Drivers/) 
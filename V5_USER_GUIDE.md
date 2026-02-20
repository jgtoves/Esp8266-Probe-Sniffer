# Version 5 - Dual-Mode Operation User Guide

## 🎉 What's New in V5

V5 solves the fundamental problem: **the ESP8266 cannot run an Access Point and capture probes simultaneously**. 

The solution: **Two separate modes** that you can switch between!

## 🔄 The Two Modes

### 📊 Dashboard Mode
- **AP is ON** - You can connect and access the web interface
- **Sniffing is OFF** - Not capturing probes
- **Purpose**: View captured data, configure settings, upload to WiGLE

### 📡 Sniffing Mode  
- **AP is OFF** - No web interface, you'll lose connection
- **Sniffing is ON** - Actively capturing probe requests
- **Purpose**: Collect probe data from nearby devices

## 🚀 How to Use V5

### Initial Setup (Same as Before)

1. Flash `esp8266_wigle_READY_TO_FLASH_v5.bin`
2. Connect to `ESP8266-Setup` (password: `setup123`)
3. Visit http://192.168.4.1
4. Configure your WiFi and WiGLE credentials
5. Device restarts in **Dashboard Mode**

### Starting Your First Capture

1. **Connect to your configured AP** (e.g., "mywifi")
2. **Visit** http://192.168.4.1
3. **You'll see a big blue button**: "▶️ START SNIFFING"
4. **Click it** and confirm
5. **You'll lose connection** - this is normal!
6. **Wait 5 minutes** (default sniffing duration)
7. **Reconnect** to the same AP
8. **View captured probes** in the dashboard!

## 🎯 Detailed Workflow

### Step 1: Dashboard Mode (Initial State)

```
┌─────────────────────────────────────┐
│  📡 ESP8266 Probe Request Sniffer   │
├─────────────────────────────────────┤
│  Total: 0 | Memory: 0 | Uploaded: 0 │
│  Mode: Dashboard (Not Sniffing)     │
│                                     │
│  [▶️ START SNIFFING]                │
│                                     │
│  [Upload] [Download] [Clear] [⚙️]   │
└─────────────────────────────────────┘
```

**What you can do:**
- View previously captured probes
- Download CSV
- Upload to WiGLE
- Change settings
- Clear data

**What you cannot do:**
- Capture new probes (sniffing is off)

### Step 2: Click "START SNIFFING"

You'll see a confirmation:
```
Start sniffing mode?
The AP will shut down and you will lose connection.
The device will automatically return to dashboard mode after 5 minutes.

[OK] [Cancel]
```

Click **OK**.

### Step 3: Sniffing Mode (AP Shuts Down)

```
Serial Monitor Output:

=== Starting Sniffing Mode ===
Shutting down AP and web server...
=== Sniffing Mode Active ===
Duration: 300 seconds
Capturing probe requests...

PROBE: A4:83:E7:12:34:56 -> HomeNetwork (RSSI: -45 dBm, CH: 6)
PROBE: B8:27:EB:AB:CD:EF -> CoffeeShop (RSSI: -67 dBm, CH: 1)
PROBE: C0:FF:EE:12:34:56 -> OfficeWiFi (RSSI: -52 dBm, CH: 11)
...
```

**What's happening:**
- AP is OFF (you can't connect)
- Device is in Station mode
- Promiscuous sniffing is active
- Channels hopping 1-13 every 250ms
- Probes being captured to memory

**What you should do:**
- Wait 5 minutes (or your configured duration)
- Device will automatically return to Dashboard Mode
- AP will restart with same name

### Step 4: Automatic Return to Dashboard

After 5 minutes:

```
Serial Monitor Output:

Sniffing duration completed. Returning to dashboard mode...

Stopping sniffing mode...
Probe capture disabled.

=== Starting Dashboard Mode ===
AP Started: YES
AP IP: 192.168.4.1
Web server started!
📱 Connect to AP and visit: http://192.168.4.1
=== Dashboard Mode Active ===
```

### Step 5: View Captured Data

1. **Reconnect** to your AP (e.g., "mywifi")
2. **Visit** http://192.168.4.1
3. **See your captured probes** in the table!

```
┌─────────────────────────────────────┐
│  📡 ESP8266 Probe Request Sniffer   │
├─────────────────────────────────────┤
│  Total: 142 | Memory: 142           │
│  Mode: Dashboard (Not Sniffing)     │
│                                     │
│  [▶️ START SNIFFING]                │
│                                     │
│  Time  | MAC Address      | SSID    │
│  2m 15s| A4:83:E7:12:34:56| HomeNet │
│  2m 10s| B8:27:EB:AB:CD:EF| Coffee  │
│  ...                                │
└─────────────────────────────────────┘
```

## ⚙️ Configuration Options

### Sniffing Duration

Default: **5 minutes (300 seconds)**

To change, edit in code before compiling:
```cpp
int sniffingDuration = 300;  // Change to desired seconds
```

Options:
- `300` = 5 minutes
- `600` = 10 minutes
- `1800` = 30 minutes
- `3600` = 1 hour
- `0` = Unlimited (must manually stop)

### Manual Stop (Advanced)

If you set `sniffingDuration = 0`, sniffing runs indefinitely.

To stop manually, you'll need to:
1. Connect a button to GPIO (e.g., GPIO0/FLASH button)
2. Add code to detect button press
3. Call `stopSniffingMode()` and `startDashboardMode()`

Or simply:
- Reset the ESP8266 (it will restart in Dashboard Mode)

## 📊 Understanding the Modes

### Why Two Modes?

The ESP8266 has **one WiFi radio** that cannot do three things at once:
1. Act as Access Point (for web interface)
2. Connect to WiFi (for uploads)
3. Sniff promiscuously (for probe capture)

**Solution**: Separate these into two modes:
- **Dashboard Mode**: AP + Web Interface (no sniffing)
- **Sniffing Mode**: Promiscuous capture (no AP)

### Mode Transition Diagram

```
┌─────────────────┐
│ Dashboard Mode  │
│ - AP ON         │
│ - Web Interface │
│ - View Data     │
└────────┬────────┘
         │
         │ Click "START SNIFFING"
         ↓
┌─────────────────┐
│ Sniffing Mode   │
│ - AP OFF        │
│ - Capture ON    │
│ - 5 min timer   │
└────────┬────────┘
         │
         │ Timer expires
         ↓
┌─────────────────┐
│ Dashboard Mode  │
│ - AP ON         │
│ - Web Interface │
│ - View Data     │
└─────────────────┘
```

## 🎯 Use Cases

### Scenario 1: Quick Capture

**Goal**: Capture probes for 5 minutes in a coffee shop

1. Connect to dashboard
2. Click "START SNIFFING"
3. Put ESP8266 in your bag
4. Wait 5 minutes
5. Reconnect and view data
6. Upload to WiGLE

### Scenario 2: Extended Monitoring

**Goal**: Monitor for 30 minutes

1. Change `sniffingDuration = 1800` in code
2. Compile and flash
3. Click "START SNIFFING"
4. Come back in 30 minutes
5. View captured data

### Scenario 3: Multiple Captures

**Goal**: Capture at multiple locations

1. Location A: Start sniffing, wait 5 min
2. Reconnect, download CSV (backup)
3. Location B: Start sniffing again, wait 5 min
4. Reconnect, view combined data
5. Upload all to WiGLE

### Scenario 4: Review Before Clearing

**Goal**: Check data quality before next capture

1. After sniffing completes
2. Reconnect and review data
3. If good: Upload to WiGLE
4. If bad: Clear and try again
5. Start new capture

## 💡 Tips & Best Practices

### Maximizing Captures

1. **Place ESP8266 in central location** - More devices in range
2. **Use external antenna** - Better signal reception
3. **Busy areas** - Coffee shops, airports, malls
4. **Peak times** - More people = more probes
5. **Multiple 5-min sessions** - Better than one long session

### Managing Data

1. **Download CSV regularly** - Backup before clearing
2. **Clear after upload** - Free up memory for new captures
3. **Check memory usage** - Max 200 probes in RAM
4. **Upload periodically** - Don't lose data

### Power Management

1. **Use power bank** - For portable operation
2. **Good USB cable** - Prevents disconnections
3. **Stable power** - Prevents mid-capture resets

### Troubleshooting

**Problem**: Can't reconnect after sniffing
- **Solution**: Wait full 5 minutes, AP takes time to restart

**Problem**: No probes captured
- **Solution**: Make sure devices are nearby and actively searching for WiFi

**Problem**: Lost connection during sniffing
- **Solution**: Normal! Wait for timer to expire

**Problem**: Want to stop early
- **Solution**: Reset ESP8266, it will restart in Dashboard Mode

## 📱 Serial Monitor Output

### Dashboard Mode
```
=== DASHBOARD MODE ===
Ready to capture probes.
Click 'Start Sniffing' in the web interface to begin.
```

### Sniffing Mode
```
=== Starting Sniffing Mode ===
Shutting down AP and web server...
=== Sniffing Mode Active ===
Duration: 300 seconds
Capturing probe requests...

PROBE: A4:83:E7:12:34:56 -> HomeNetwork (RSSI: -45 dBm, CH: 6)
PROBE: B8:27:EB:AB:CD:EF -> CoffeeShop (RSSI: -67 dBm, CH: 1)
...
```

### Returning to Dashboard
```
Sniffing duration completed. Returning to dashboard mode...

=== Starting Dashboard Mode ===
AP Started: YES
AP IP: 192.168.4.1
=== Dashboard Mode Active ===
```

## 🔧 Advanced: Customization

### Change Default Duration

Edit `main.cpp`:
```cpp
int sniffingDuration = 600;  // 10 minutes
```

### Add Physical Stop Button

Connect button to GPIO0 (FLASH button), add to `loop()`:
```cpp
if (digitalRead(0) == LOW && isSniffing) {
  stopSniffingMode();
  startDashboardMode();
}
```

### LED Status Indicator

Add LED to show mode:
```cpp
// Dashboard Mode: LED solid
// Sniffing Mode: LED blinking

if (isSniffing) {
  digitalWrite(LED_PIN, (millis() / 500) % 2);  // Blink
} else {
  digitalWrite(LED_PIN, HIGH);  // Solid
}
```

## 📊 Comparison: V4 vs V5

| Feature | V4 (Broken) | V5 (Working) |
|---------|-------------|--------------|
| **AP Visibility** | ✅ Works | ✅ Works |
| **Probe Capture** | ❌ Fails | ✅ Works |
| **Simultaneous** | ❌ Tries and fails | ✅ Separate modes |
| **User Control** | ❌ Automatic | ✅ Manual toggle |
| **Data Collection** | ❌ None | ✅ Successful |

## 🎉 Success Indicators

You know V5 is working when:

✅ AP appears in WiFi list  
✅ Can access dashboard at http://192.168.4.1  
✅ See "START SNIFFING" button  
✅ Clicking button shuts down AP (lose connection)  
✅ Serial monitor shows "Capturing probe requests..."  
✅ After 5 minutes, AP reappears  
✅ Reconnect and see captured probes in table  
✅ Probes have MAC addresses, SSIDs, RSSI values  

---

**V5 finally delivers working probe capture with a user-friendly interface!** 🚀

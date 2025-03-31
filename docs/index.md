---
layout: default
#Change the title below to reflect your project. This title will be shown on the left panel as Menu item.
title: Documentation
nav_order: 1
has_children: false
---

# NexusConnect UHF Documentation - iOS (2.0.3)
{: .fs-9 .no_toc }

NexusConnectUHF is a library that manages the connection to NexusConnectUHF device, which scans barcodes and UHF tags. It provides functionality to establish and maintain communication, receive scanned data, and process barcode and UHF tag inputs for various applications.
{: .fs-5 .fw-300 }

---
<!-- This section reserved for the Table of Contents. Don't remove it. -->
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
---

<!-- Add your documentation content below this line -->

## Installation
### Manual Installation
To integrate NexusConnectSDK into your iOS application:
1. **Obtain the SDK:** Contact IPCMobile to request the latest NexusConnectSDK library (`NexusConnectSDK.xcframework` file).
2. **Add Dependency:** Drag and drop the NexusConnectSDK.xcframework into your project's Frameworks, Libraries, and Embedded Content section in Xcode. Ensure the SDK is set to "Embed & Sign" to enable seamless integration and interaction with NexusConnect UHF devices.

## Usage
### Developer Key
To securely access NexusConnectSDK features, you need an IPCMobile developer key. Register at [developer.ipcmobile.com](https://developer.ipcmobile.com) and create a key specifically associated with your application’s package name.

### Initialization
Initialize the SDK in your application's lifecycle to ensure all required services and resources are properly configured:
```swift
NexusConnectUHF.initialize(key: "YOUR_DEVELOPER_KEY")
let nexusConnect = NexusConnectUHF.shared()
```

Always clean up resources when your application or activity is destroyed to prevent memory leaks:
```swift
nexusConnect.cleanup()
```

### Discover and Connect Devices
NexusConnect UHF devices support Bluetooth discovery, making it easy to find and connect devices nearby.

Start device discovery:
```swift
nexusConnect.startDiscoveringDevices()
```

Handle discovered devices through delegate callbacks via `NCDeviceDiscoveryDelegate`, capturing device details and signal strength:
```swift
// Set delegate
nexusConnect.setDeviceDiscoveryDelegate(self)

// Conforms to `NCDeviceDiscoveryDelegate` and implements
func onDeviceDiscovered(device: NCDevice, rssi: Int) {
    // Handle found devices
}
```

Stop discovery when devices are found or no longer needed:
```swift
nexusConnect.stopDiscoveringDevices()
```

Connect devices easily by object reference or MAC address:
```swift
nexusConnect.connectDevice(device) // Using device object
nexusConnect.connectDevice("device_mac_address") // Using MAC address
```

Disconnect when finished:
```swift
nexusConnect.disconnectDevice()
```

### Connection State Listener
Effectively monitor the device's connection state to manage user interface changes or operational logic, via `NCConnectionStateDelegate`:
```swift
// Set delegate
nexusConnect.setConnectionStateDelegate(self)

// Conforms to `NCConnectionStateDelegate` and implements
func onConnectionStateChanged(state: NCConnectionState) {    
    if state == .connected {
        // Handle connected state
    }
}
```

### Device Button Events
Respond instantly to hardware button interactions, allowing real-time operational feedback via `NCDeviceOperationDelegate`:
```swift
// Set delegate
nexusConnect.setDeviceOperationDelegate(self)

// Conform to `NCDeviceOperationDelegate` and implements
func onDeviceButtonPressed() {
    // Handle event when trigger button presses
}

func onDeviceButtonReleased() {
    // Handle event when trigger button releases
}
```

### Barcode Operations
Efficiently scan and process barcode data using integrated readers.

#### Barcode Scanning
Start and stop barcode scanning easily:
```swift
nexusConnect.startBarcodeReader()
nexusConnect.stopBarcodeReader()
```

Receive barcode data instantly, via `NCBarcodeDataDelegate`
```swift
// Set delegate
nexusConnect.setBarcodeDataDelegate(self)

// Conforms to `NCBarcodeDataDelegate` and implements
func onBarcodeData(value: String, type: NCSymbology) {
    // Handle barcode string data
    print("Barcode: \(value) - Type: \(type)")
}

// Optional delegate
func onBarcodeData(data: Data, type: NCSymbology) {
    // Handle barcode binary data
    print("Barcode Binary Data: \(data)  - Type: \(type)")
}
```

#### Scan Modes
Configure barcode scanning behavior to suit your application:
```swift
// Set barcode scan mode
nexusConnect.setBarcodeScanMode(.single)

// Retrieve barcode scan mode
let currentMode = nexusConnect.getBarcodeScanMode()
```

### UHF Operations
Manage RFID tag inventory, data reading, and writing through powerful UHF functionality.

#### Tag Reading
Start and stop RFID tag scanning quickly:
```swift
nexusConnect.startUHFReader()
nexusConnect.stopUHFReader()
```

Capture RFID tag information dynamically, via `NCUHFTagDelegate`:
```swift
// Set delegate
nexusConnect.setUHFTagDataDelegate(self)

// Conforms to `NCUHFTagDelegate` and implements
func onUHFTagDetected(tag: NCUHFTag) {
    print("UHF: \(tag.pc ?? "") \(tag.epc ?? "") [\(tag.rssi)]\n")
}
```

#### Antenna Power
Optimize RFID reader performance by adjusting antenna power settings (1-30 dBm):
```swift
let currentPower = nexusConnect.getUHFPower()
nexusConnect.setUHFPower(15)
```

#### Region Frequency
Adjust regional frequency settings for global compatibility:
```swift
let currentFrequency = nexusConnect.getUHFFrequency()
nexusConnect.setUHFFrequency(.unitedStates)
```

#### Sessions and Targets
Manage tag querying sessions and targets to optimize RFID reading efficiency:
```swift
let inventory = nexusConnect.getUHFInventory()
inventory.querySession = NCUHFInventory.QuerySession.s0
inventory.queryTarget = NCUHFInventory.QueryTarget.a
nexusConnect.setUHFInventory(inventory)
```

#### Memory Bank Configuration
Set specific memory bank modes for detailed data management on RFID tags:
```swift
let memoryBank = nexusConnect.getUHFMemoryBank()
memoryBank.mode = .epc // adjust offsets as needed
nexusConnect.setUHFMemoryBank(memoryBank)
```

#### Tracking UHF Tags
Use RSSI to effectively estimate RFID tag proximity and manage inventory more precisely:
```swift
var tagFilter = NCUHFTagFilter()
tagFilter.bankType = .epc
tagFilter.data = "e200421c8ad0601509c8683d"
tagFilter.offset = 32
tagFilter.length = 96

// Tracking a known tag using filter
nexusConnect.startTrackingUHFTag(tagFilter: tagFilter) { tag, signal, angle in
    print("Found tag: \(tag.epc!) - \(signal) - at angle: \(angle)")
}

// Stop tracking tags
nexusConnect.stopTrackingTag()
```

#### Writing and Killing Tags
Securely write data to RFID tags and manage tag lifecycle operations:
```swift
let filter = NCUHFTagFilter().apply {
    bankType = NCUHFMemoryBankType.EPC
    offset = 32
    length = 16
    data = "AABB"
}

// Write data
do {
    try self.nexusConnect.writeUHFTag(filter: filter, bankType: .epc, offset: 32, length: 96, data: "e200421cabc0601509c86a4c")
}
catch {
    print("Write error: \(error)")
}
```

Permanently disable (kill) tags:
```swift
do {
    try self.nexusConnect.killUHFTag(filter: filter, password: "TAG_PASSWORD")
}
catch {
    print("Kill error: \(error)")
}
```

## Device Information
Retrieve essential device metrics and firmware details:
```swift
let battery = nexusConnect.getBatteryLevel()
let temperature = nexusConnect.getTemperature()
let mainboardVersion = nexusConnect.getMainboardVersion()
let uhfVersion = nexusConnect.getUHFFirmwareVersion()
let bleVersion = nexusConnect.getBLEFirmwareVersion()
```

## Device Settings
Customize device alerts and feedback:
```swift
let beepOn = nexusConnect.getScanBeep()
nexusConnect.setScanBeep(false)
```

## Firmware Update
Perform seamless firmware updates with real-time progress tracking, via `NCUpdateProgressDelegate`:
```swift
// Set delegate
nexusConnect.setUpdateProgressDelegate(self)

// Conforms to `NCUpdateProgressDelegate` and implements
func onUpdateComplete(success: Bool) {
    print("Update status: \(success ? "Success" : "Failed")")
}

func onUpdateProgress(progress: Int) {
    print("Update progress: \(progress)%")
}
```


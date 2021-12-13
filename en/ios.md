# iOS

## Download

- [SDK](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.1.0/ios/CrimsonSDK.xcframework.zip)
- [Sample-CocoaPods](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.1.0/ios/CrimsonSDKExample.zip)
- [Video](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/ios/example.mp4)

## Requirement

- iOS 10.0+
- arm64
- BitCode Disabled

## Integration

### CocoaPods (iOS 10.0+) Recommend

You can use CocoaPods to install **CrimsonSDK** by adding it to your Podfile:

```ruby
platform :ios, '10.0'

use_frameworks!

pod 'CrimsonSDK', :podspec => 'https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.1.0/ios/CrimsonSDK.podspec'
```

### Manual

- Project->Target->General->Linked Frameworks, Libraries and Embedded Content
  ![](../.gitbook/assets/import_crimson_sdk.png)

#### Dependencies

- Accelerate.framework
- CoreBluetooth.framework
- libc++.tbd

### Info.plist

```swift
<key>NSBluetoothAlwaysUsageDescription</key>
<string>APP need use Bluetooth to connect Crimson</string>

<--BLE BackgroundModes-->
<key>UIBackgroundModes</key>
<array>
<string>bluetooth-central</string>
</array>
```

## Usage

### FAQ

{% page-ref page="faq.md" %}

### Scan

```swift
// scan
BLEDeviceManager.shared.startScan()
BLEDeviceManager.scannerDelegate = self

extension ScanVC: CrimsonScannerDelegate {
    func centralManagerDidUpdateState(_ state: CBManagerState) {
        if state == .poweredOn {
            BLEDeviceManager.shared.startScan()
        }
    }

    func onFoundDevices(_ devices: Array<CrimsonDevice>) {
        self.devices = devices
        //TODO: show scan result
    }
}
```

### Connect

```swift
// NOTE: startScan and stopScan should use in pairs, stopScan when connect device
BLEDeviceManager.shared.stopScan()
device.delegate = self
device.connect()

extension ScanVC: CrimsonDelegate {
    func onConnectivityChanged(_ connectivity: Connectivity) {
        if connectivity == .connected {
            let device = devices[selectedIndex]
            self.pair(device: device)
            return
        }
    }
}

// CrimsonDelegate
@objc public protocol CrimsonDelegate {
    @objc optional func onError(_ error: CrimsonError)
    @objc optional func onDeviceInfoReady(_ deviceInfo: DeviceInfo)
    @objc optional func onSystemInfo(_ systemInfo: SystemInfo)
    // batteryLevel 0~100, -1 unknown
    @objc optional func onBatteryLevelChange(_ batteryLevel: Int)
    @objc optional func onConnectivityChange(_ connectivity: Connectivity)

    //******************** NOTE: invoked after startEEG *******************
    @objc optional func onContactStateChange(_ contactState: ContactState)
    // NOTE: invoked after startIMU
    @objc optional func onOrientationChange(_ orientation: Orientation)
    @objc optional func onIMUData(_ imu: IMU)
    @objc optional func onEEGData(_ eeg: EEG)
    @objc optional func onBrainWave(_ wave: BrainWave)
    @objc optional func onAttention(_ attention: Float)
    @objc optional func onMeditation(_ meditation: Float)
    @objc optional func onBlink() // Eye blink
}
```

### Pair

```swift
func pair(device: CrimsonDevice) {
    startUIBlockingIndicator(message: "Pairing")
    device.delegate = self
    device.pair { resp in
        stopUIBlockingIndicator()
        if resp.success() {
            cmsn_print("pair success")
        } else {
            cmsn_print("pair failed")
            BLEDeviceManager.shared.startScan() //restart scan device

            if let sysResp = resp as? SysConfigResponse, sysResp.error?.code == .validatePairInfo {
                // TODO: validatePairInfo failed
            }
        }
    }
}
```

### OTA

```swift
public class CrimsonOTA {
    public static var latestVersion = ""
    public static var url = ""
    public static var desc = ""
    public static var descEN = ""
}

boolean ret = device.isNewFirmwareAvailable();
print("isNewFirmwareAvailable=\(ret)");
print("latestVersion=\(CrimsonOTA.latestVersion)");
print("desc=\(CrimsonOTA.desc)");

device.startDfu(self)
extension DFUViewControler: CrimsonOtaDelegate {
    func onSuccess() {
        logI("ota Success")
    }
    func onFailure(_ error: CrimsonError?) {
        if error != nil { logI("ota Failure, error=\(error!.message)") }
    }
    func onProgress(_ progress: Int) {
        logI("ota progress=\(progress)")
    }
}
```

### Model

```swift
@objc public enum Connectivity: Int, CaseIterable {
    case connecting = 0
    case connected = 1
    case disconnecting = 2
    case disconnected = 3
}

@objc public enum ContactState: Int, CaseIterable {
    case unknown = 0
    case contact = 1
    case noContact = 2
}

@objc public enum Orientation: Int, CaseIterable {
    case unknown
    case upward   //Normal
    case downward //UpsideDown
}

@objc public class EEG: NSObject {
    @objc public let sequenceNumber : Int
    @objc public let sampleRate: Float
    @objc public let eegData: [Float]
    @objc public let signalType: AFEDataSignalType
}

@objc public class BrainWave: NSObject {
    @objc public let delta : Double
    @objc public let theta : Double
    @objc public let alpha : Double
    @objc public let lowBeta : Double
    @objc public let highBeta : Double
    @objc public let gamma : Double
}

@objc public enum CrimsonErrorCode: Int32{
    case none = 0
    case unknown = -1
    case messageBuildingFailed = -2
    case paramsError = -3

    case deviceNotConnected = -160
    case deviceUuidUnavailable = -196

    case otaDownload = -200
    case otaDfu = -20
}


@objc public class CrimsonError: NSObject {
    @objc public let code: CrimsonErrorCode
    @objc public let message: String

    init(code:CrimsonErrorCode) {
        self.code = code;
        switch code {
            case .none:
                message = "Success"
            case .unknown:
                message = "Unknown error"
            case .messageBuildingFailed:
                message = "Message building failed"
            case .deviceNotConnected:
                message = "Device not connected"
            case .deviceUuidUnavailable:
                message = "Failed to obtain iOS device UUID" //get idfv failed
            default:
                message = "Unknown error case"
        }
    }
}

@objc public enum SysConfigErrorCode: Int32 {
    case none = 0
    case unknown = 1
    case otaFailedLowPower = 2
    case pairError = 3
    case validatePairInfo = 4
    case internalStorageError = 5
}

@objc public class SysConfigError: NSObject {
    @objc public let code: SysConfigErrorCode
    @objc public let message: String

    init(code:SysConfigErrorCode){
        self.code = code
        self.message = String(cString: sys_config_err_code_to_msg(code.rawValue))
        super.init()
    }
}
```

### StartEEG

```swift
device.delegate = self
device.startEEGStream()
```

### StartIMU

```swift
// IMU SampleRate
@objc public enum IMUDataSampleRate: Int, CaseIterable {
    case unused
    case sr12_5 //Default
}

device.delegate = self
device.startEEGStream()
device.startIMU() { (resp) in
    if resp.success() {
        self.toast("start imu success")
    } else {
        self.toast("\(resp.message())")
    }
}
```

### More

```swift
// Default is INFO
CrimsonSDK.setLogLevel(LOG_LEVEL_ERROR)
CrimsonSDK.setLogLevel(LOG_LEVEL_INFO)

public func connect()
public func disconnect()
public func pair(onResponse: @escaping CrimsonRespCallback)
public func getSystemInfo(onResponse: @escaping CrimsonRespCallback)

public func startEEGStream()
public func stopEEGStream()
public func startIMU(sampleRate: IMUDataSampleRate, onResponse: @escaping CrimsonRespCallback)
public func stopIMU(onResponse: @escaping CrimsonRespCallback)

// @param name length should be 4 ~ 18
public func setDeviceName(_ name: String, onResponse: @escaping CrimsonRespCallback)

// public typealias SKColor = UIColor
public func setLEDColor(_ color: SKColor, onResponse: @escaping CrimsonRespCallback)

// @param timeSec => seconds to put device into sleep mode, 0 for no sleep
public func setSleepIdleTime(_ timeSec: Int, onResponse: @escaping CrimsonRespCallback)

// @param intensity => vibration intensity, 0 ~ 100
public func setVibrationIntensity(_ intensity: Int, onResponse: @escaping CrimsonRespCallback)
```

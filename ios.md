# iOS

## [下载SDK](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/ios/CrimsonBlue.xcframework.zip)   [下载Example](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/ios/CrimsonBlueExample.zip)  [演示视频](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/ios/example.mp4)

## Scan 扫描

### 首次配对新设备时，需要先将头环设置为配对模式--&gt;蓝灯快闪

```swift
CrimsonBlueManager.shared.startScan()
CrimsonBlueManager.scannerDelegate = self

extension ScanVC: CrimsonScannerDelegate {
    func centralManagerDidUpdateState(_ state: CBManagerState) {
        if state == .poweredOn {
            CrimsonBlueManager.shared.startScan()
        }
    }

    func onFoundDevices(_ devices: Array<CrimsonBlueDevice>) {
        self.devices = devices
        //TODO
    }
}
```

## Connect 连接

```swift
CrimsonBlueManager.shared.stopScan()
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
```

## Pair 配对

### 首次配对新设备时，需要先将头环设置为配对模式--&gt;蓝灯快闪

```swift
func pair(device: CrimsonBlueDevice) {
    startUIBlockingIndicator(message: "Pairing")
    device.delegate = self
    device.pair { resp in
        stopUIBlockingIndicator()
        if resp.success() {
            cmsn_print("pair success")
        } else {
            cmsn_print("pair failed")
        }
    }
}
```

## StartEEG 开启传输脑电数据

```swift
device.delegate = self
device.startEEGStream()
```

## StartIMU 开启传输陀螺仪数据

```swift
device.delegate = self
device.startEEGStream()
device.startIMU(sampleRate: .sr104) { (resp) in
    if resp.success() {
        self.toast("start imu success")
    } else {
        self.toast("\(resp.message())")
    }
}
```

## CrimsonDelegate

```swift
@objc public protocol CrimsonDelegate {
    @objc optional func onError(_ error: CrimsonError)
    @objc optional func onDeviceInfoReady(_ deviceInfo: DeviceInfo)
    @objc optional func onSystemInfo(_ systemInfo: SystemInfo)

    @objc optional func onConnectivityChanged(_ connectivity: Connectivity)
    @objc optional func onContactStateChanged(_ contactState: ContactState)
    @objc optional func onOrientationChanged(_ orientation: Orientation)
    @objc optional func onLeadOffStatus(_ center: ContactState, _ side: ContactState) //center_rld, side_channels
    @objc optional func onBatteryLevelChanged(_ batteryLevel: Int)

    @objc optional func onIMUData(_ imu: IMU)
    @objc optional func onEEGData(_ eeg: EEG)
    @objc optional func onBrainWave(_ wave: BrainWave)
    @objc optional func onBlink()
    @objc optional func onAttention(_ attention: Float)
    @objc optional func onMeditation(_ meditation: Float)
}
```

## More 更多详见SDK

```swift
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


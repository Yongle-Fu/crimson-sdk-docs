# Android

## [下载Example](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/android/example.zip)  [演示视频](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/android/example.mp4)

## Scan 扫描

### 首次配对新设备时，需要先将头环设置为配对模式--&gt;蓝灯快闪

```java
showLoadingDialog();
Activity that = this;
CrimsonSDK.scanDevices(this, new CrimsonDeviceScanListener() {
    @Override
    public void onResult(List<CrimsonDevice> results) {
        dismissLoadingDialog();
        if(results.size() == 0) showMessage("No headband found");
        devices = results;
        deviceListAdapter.notifyDataSetChanged();
    }

    @Override
    public void onError(CrimsonError error) {
        dismissLoadingDialog();
        showError(error);
        if(error.getCode() == CrimsonError.ERROR_PERMISSION_DENIED){
            CrimsonPermissions.requestPermissions(that);
        } else if(error.getCode() == CrimsonError.ERROR_BLE_DISABLED) {
            CrimsonPermissions.enableBluetooth(that);
        }
    }
});
```

## Connect 连接

```java
listener = new DeviceListener();
device.setListener(listener);
device.connect(this);
```

## Pair 配对

### 首次配对新设备时，需要先将头环设置为配对模式--&gt;蓝灯快闪

```java
pairing = true;
devicePairButton.setText("Pairing");

if (device.isInPairingMode()) {
    device.pair(error -> {
        pairing = false;
        if (error == null) {
            paired = true;
            devicePairButton.setText("Paired");
        } else {
            Toast.makeText(this, "Pair failed " + error.getMessage(), Toast.LENGTH_SHORT).show();
            devicePairButton.setText("Pair");
        }
    });
} else if (device.isInNormalMode()) {
    device.validatePairInfo(error -> {
        pairing = false;
        if (error == null) {
            paired = true;
            devicePairButton.setText("Paired");
        } else {
            Toast.makeText(this, "Validate pair info failed " + error.getMessage(), Toast.LENGTH_SHORT).show();
            devicePairButton.setText("Pair");
        }
    });
}
```

## StartEEG 开启传输脑电数据

```java
int _ = device.startDataStream(error -> {
    if (error != null) {
        Log.i("startDataStream:" + error.getCode(), error.getMessage());
    }
});
```

## StartIMU 开启传输陀螺仪数据

```c
typedef enum { 
IMU_SAMPLE_RATE_UNUSED = 0, 
IMU_SAMPLE_RATE_12_5 = 0x10, 
IMU_SAMPLE_RATE_26 = 0x20, 
IMU_SAMPLE_RATE_52 = 0x30, 
IMU_SAMPLE_RATE_104 = 0x40
} IMUSampleRate;
```

```java
device.configImu(0x40, error -> {
    if (error != null) {
        Log.i("startIMU:" + error.getCode(), error.getMessage());
    }
});
```

## CrimsonDeviceListener

```java
public abstract class CrimsonDeviceListener {
    public void onError(CrimsonError error){}

    public void onConnectivityChanged(int connectivity){}
    public void onDeviceInfoReady(DeviceInfo info){}

    public void onBatteryLevelChanged(int batteryLevel){}
    public void onContactStateChanged(int state){}
    public void onOrientationChanged(int orientation){}

    public void onIMUData(IMU data){}
    public void onEEGData(EEG data){}
    public void onBrainWave(BrainWave wave){}
    public void onAttention(float attention){}
    public void onMeditation(float meditation){}
    public void onBlink(){}
}
```

## More 更多详见SDK

```java
public void connect(@NonNull Context context)
public void disconnect()
public void pair(CrimsonResponseCallback callback)
public void validatePairInfo(CrimsonResponseCallback callback)

public int startDataStream(CrimsonResponseCallback callback)
public int stopDataStream(CrimsonResponseCallback callback)
public int configImu(int sampleRate, CrimsonResponseCallback callback)

// @param name length should be 4 ~ 18
public int setDeviceName(String name, CrimsonResponseCallback callback)

/*
 * @param red     0-255
 * @param green   0-255
 * @param blue    0-255
 */
public int setLEDColor(int red, int green, int blue, CrimsonResponseCallback callback)

// @param timeSec => seconds to put device into sleep mode, 0 for no sleep
public int setSleepIdleTime(int timeSec, CrimsonResponseCallback callback)

// @param intensity => vibration intensity, 0 ~ 100
public int setVibrationIntensity(int intensity, CrimsonResponseCallback callback)
```


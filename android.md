# Android

## Download

[下载Example](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.2/android/android_example.zip)

[演示视频](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/android/example.mp4)

## Requirement

Android 6.0+

## Integration

```groovy
// build.gradle
repositories {
    maven {
        credentials {
            username 'maven-read'
            password 'tBauTKVo6IqiKvHd'
        }
        url = "https://nexus.ci.brainco.cn/repository/maven-public"
    }
}

// app/build.gradle
dependencies {
    // import crimson-sdk from maven
    api 'tech.brainco:crimsonsdk:1.0.2'
}

// manifest
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.xxx.xxx">
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
</manifest>
```

## Usage

### Scan 扫描

#### 首次配对新设备时，需要先将头环设置为 _配对_  模式--&gt;蓝灯快闪

{% page-ref page="faq.md" %}

```java
// Permissions check
if (!CrimsonPermissions.checkBluetoothFeature(this)) {
    showMessage("BLE not supported");
    return;
}

BluetoothAdapter adapter = ((BluetoothManager)this.getSystemService(Context.BLUETOOTH_SERVICE)).getAdapter();
if (adapter == null || !adapter.isEnabled()) {
    CrimsonPermissions.enableBluetooth(this);
    return;
}

// location Permission
if (!CrimsonPermissions.checkPermissions(this)) {
    CrimsonPermissions.requestPermissions(this);
    return;
}
```

```java
// startScan
showLoadingDialog();
CrimsonSDK.startScan(new CrimsonDeviceScanListener() {
    @Override
    public void onFoundDevices(List<CrimsonDevice> results) {
        if (!results.isEmpty()) dismissLoadingDialog();
        devices = results;
        deviceListAdapter.notifyDataSetChanged();
    }

    @Override
    public void onError(CrimsonError error) {
        dismissLoadingDialog();
    }
}, null);

// scanFilters
final ArrayList<ScanFilter> scanFilters = new ArrayList<>();
scanFilters.add(new ScanFilter.Builder().setDeviceAddress("xxx-xxx").build());

// stopScan
CrimsonSDK.stopScan();
```

### Connect 连接

```java
listener = new DeviceListener();
device.setListener(listener);
device.connect(this);

// CrimsonDeviceListener
public abstract class CrimsonDeviceListener {
    // class CrimsonError
    public void onError(CrimsonError error){}
    public void onDeviceInfoReady(DeviceInfo info){} //蓝牙设备信息
    // batteryLevel 区间在0~100, -1表示未知
    public void onBatteryLevelChange(int batteryLevel){} //电量
    // Enum Connectivity
    public void onConnectivityChange(int connectivity){} //连接状态
    
    //******************** NOTE: invoked after startEEG *******************
    // Enum ContactState
    public void onContactStateChange(int state){} //佩戴状态
    // Enum Orientation, NOTE: invoked after startIMU
    public void onOrientationChange(int orientation){} // 佩戴方向
    public void onIMUData(IMU data){} // 陀螺仪数据
    public void onEEGData(EEG data){} // 脑电EEG数据
    public void onBrainWave(BrainWave wave){}    //脑电频域波段数据 
    public void onAttention(float attention){}   //注意力指数
    public void onMeditation(float meditation){} //冥想指数
    public void onBlink(){} // 眨眼事件
}
```

### Pair 配对

```java
void onConnectivityChange(int connectivity) {
    if (connectivity == Connectivity.CONNECTED) {
        pair()
    }
}

func pair() {
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
                if (error.getCode() == 3) {
                    // TODO: 配对失败
                }
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
                if (error.getCode() == 4) {
                    // TODO: 检验配对信息失败，去重新配对
                }
            }
        });
    }
}
```

### Model

```java
// 头环连接状态
public class Connectivity {
    public static final int CONNECTING = 0;
    public static final int CONNECTED = 1;
    public static final int DISCONNECTING = 2;
    public static final int DISCONNECTED = 3;
}
// 佩戴状态，电极与皮肤接触良好
public class ContactState {
    public static final int UNKNOWN = 0;
    public static final int CONTACT = 1;   //佩戴好
    public static final int NO_CONTACT = 2;//未戴好
}
// 佩戴方向，检测是否佩戴反
public class Orientation {
    public static final int UNKNOWN = 0;
    public static final int UPWARD = 1;   //头环戴正
    public static final int DOWNWARD = 2; //头环戴反
}
// EEG, 默认为每秒回调5次
public class EEG {
    private final int sequenceNumber;
    private final double sampleRate;
    private final float[] eegData;
}
// 脑电频域波段能量
public class BrainWave {
    private final double delta;
    private final double theta;
    private final double alpha;
    private final double lowBeta;
    private final double highBeta;
    private final double gamma;
}
```

### StartEEG 开启传输脑电数据

```java
device.startEEGStream(error -> {
    if (error != null) {
        Log.i(TAG,"startEEGStream:" + error.getCode() + ", message=" + error.getMessage());
    } else {
        Log.i(TAG,"startEEGStream success");
    }
});
```

### StartIMU 开启传输陀螺仪数据

```c
// IMU SampleRate 采样频率
public class IMUSampleRate {
    public final static int UNUSED = 0; // stop imu stream
    public final static int SR12_5 = 0x10; // 默认使用
}
```

```java
device.startIMU(error -> {
    if (error != null) {
        Log.i("startIMU:" + error.getCode(), error.getMessage());
    }
});
```

### More

```java
// 设置日志级别，默认为INFO
CrimsonSDK.setLogLevel(CrimsonSDK.LogLevel.ERROR);
public static void setLogLevel(int level)

public void connect(@NonNull Context context)
public void disconnect()
public void pair(CrimsonResponseCallback callback)
public void validatePairInfo(CrimsonResponseCallback callback)

public int startDataStream(CrimsonResponseCallback callback)
public int stopDataStream(CrimsonResponseCallback callback)
public int startIMU(int sampleRate, CrimsonResponseCallback callback)
public int stopIMU(CrimsonResponseCallback callback)

// @param name length should be 4 ~ 18, 修改设备名称成功之后，固件会断开连接
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


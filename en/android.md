# Android

## Download

[Sample](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.3/android/android_example.zip)

[Video](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.0/android/example.mp4)

## Requirement

Android 6.0 or later

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
    api 'tech.brainco:crimsonsdk:1.1.0'
}

// manifest
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.xxx.xxx">
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
</manifest>

// proguard-rules.pro
-dontwarn java.awt.*
-keep class com.sun.jna.* { *; }
-keepclassmembers class * extends com.sun.jna.* { public *; }
```

## Usage

### FAQ

{% page-ref page="faq.md" %}

### Scan

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
// BLEStateReceiver
public class ScanActivity extends BaseActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        CrimsonSDK.setLogLevel(CrimsonSDK.LogLevel.INFO);
        CrimsonSDK.registerBLEStateReceiver(this); //注册蓝牙状态监听
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        CrimsonSDK.unregisterBLEStateReceiver(this);//移除注册蓝牙状态监听
    }
}

// startScan
showLoadingDialog();
CrimsonSDK.startScan(new CrimsonDeviceScanListener() {
    /*
    param: state
    BluetoothAdapter.STATE_OFF:
    @IntDef(prefix = { "STATE_" }, value = {
            STATE_OFF,
            STATE_TURNING_ON,
            STATE_ON,
            STATE_TURNING_OFF,
            STATE_BLE_TURNING_ON,
            STATE_BLE_ON,
            STATE_BLE_TURNING_OFF
    })
    */
    @Override
    public void onBluetoothAdapterStateChange(int state) {
                Log.i(TAG, "BluetoothAdapter state=" + state);
                switch (state) {
                    case BluetoothAdapter.STATE_ON:
                        // restart scan
                        scanDevices();
                        break;
                    case BluetoothAdapter.STATE_OFF:
                        CrimsonSDK.stopScan();
                        break;
                    default:
                        break;
                }
            }

    @Override
    public void onFoundDevices(List<CrimsonDevice> results) {
        if (!results.isEmpty()) dismissLoadingDialog();
        devices = results;
        deviceListAdapter.notifyDataSetChanged();
    }

    @Override
    public void onError(CrimsonError error) {
        dismissLoadingDialog();
        showMessage(error.getMessage());
    }
}, null);

// scanFilters
final ArrayList<ScanFilter> scanFilters = new ArrayList<>();
scanFilters.add(new ScanFilter.Builder().setDeviceAddress("xxx-xxx").build());

// stopScan
CrimsonSDK.stopScan();

// NOTE: startScan and stopScan should use in pairs, stopScan when connect device
```

### Connect

```java
listener = new DeviceListener();
device.setListener(listener);
device.connect(this);

// CrimsonDeviceListener
public abstract class CrimsonDeviceListener {
    // class CrimsonError
    public void onError(CrimsonError error){}
    public void onDeviceInfoReady(DeviceInfo info){}
    // batteryLevel 0~100, -1 unknown
    public void onBatteryLevelChange(int batteryLevel){}
    // Enum Connectivity
    public void onConnectivityChange(int connectivity){}

    //******************** NOTE: invoked after startEEG *******************
    // Enum ContactState
    public void onContactStateChange(int state){}
    // Enum Orientation, NOTE: invoked after startIMU
    public void onOrientationChange(int orientation){}
    public void onIMUData(IMU data){}
    public void onEEGData(EEG data){}
    public void onBrainWave(BrainWave wave){}
    public void onAttention(float attention){}
    public void onMeditation(float meditation){}
    public void onBlink(){} // eye blink
}
```

### Pair

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
                    // TODO: pair failed
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
                    // TODO: validatePairInfo failed
                }
            }
        });
    }
}
```

### Model

```java
public class Connectivity {
    public static final int CONNECTING = 0;
    public static final int CONNECTED = 1;
    public static final int DISCONNECTING = 2;
    public static final int DISCONNECTED = 3;
}

public class ContactState {
    public static final int UNKNOWN = 0;
    public static final int CONTACT = 1;
    public static final int NO_CONTACT = 2;
}

public class Orientation {
    public static final int UNKNOWN = 0;
    public static final int UPWARD = 1;   //Normal
    public static final int DOWNWARD = 2; //UpsideDown
}

public class EEG {
    private final int sequenceNumber;
    private final double sampleRate;
    private final float[] eegData;
}

public class BrainWave {
    private final double delta;
    private final double theta;
    private final double alpha;
    private final double lowBeta;
    private final double highBeta;
    private final double gamma;
}

public class CrimsonError {
    public static final int ERROR_NONE = 0;
    public static final int ERROR_UNKNOWN = -1;
    public static final int ERROR_SCAN_FAILED_INTERNAL = -64;
    public static final int ERROR_SCAN_FEATURE_UNSUPPORTED = -65;
    public static final int ERROR_PERMISSION_DENIED = -128;
    public static final int ERROR_DEVICE_NOT_CONNECTED = -160;
    public static final int ERROR_RESPONSE_TIMEOUT = -1001;

    private int code;
    private String message;

    CrimsonError(int code) {
        this.code = code;
        switch(code) {
            case ERROR_NONE:
                message = "Success";
                break;
            case ERROR_UNKNOWN:
                message = "Unknown error";
                break;
            case ERROR_SCAN_FAILED_INTERNAL:
                message = "Bluetooth LE scan failed internally";
                break;
            case ERROR_SCAN_FEATURE_UNSUPPORTED:
                message = "Bluetooth LE scan feature is not supported for this device";
                break;
            case ERROR_PERMISSION_DENIED:
                message = "Bluetooth or location permissions could have been denied";
                break;
            case ERROR_DEVICE_NOT_CONNECTED:
                message = "Device not connected";
                break;
            case ERROR_RESPONSE_TIMEOUT:
                message = "response timeout";
                break;
            default:
                message = "Unknown error:" + code;
        }
    }
}
```

### StartEEG

```java
device.startEEGStream(error -> {
    if (error != null) {
        Log.i(TAG,"startEEGStream:" + error.getCode() + ", message=" + error.getMessage());
    } else {
        Log.i(TAG,"startEEGStream success");
    }
});
```

### StartIMU

```c
// IMU SampleRate
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

### OTA

```java
public class CrimsonOTA {
    public static String latestVersion;
    public static String url;
    public static String desc;
    public static String descEN;
}

boolean ret = device.isNewFirmwareAvailable();
Log.i(TAG, "isNewFirmwareAvailable=" + ret);
Log.i(TAG, "latestVersion=" + CrimsonOTA.latestVersion);
Log.i(TAG, "desc=" + CrimsonOTA.desc);

device.startDfu(this, new DFUCallback() {
    @Override
    public void onSuccess() {
        Log.i(TAG, "onSuccess");
    }

    @Override
    public void onFailure(Exception e) {
        Log.i(TAG, e.getMessage());
    }

    @Override
    public void onProgress(int progress) {
        Log.i(TAG, "progress=" + progress);
    }
});
```

### More

```java
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

// @param name length should be 4 ~ 18, device will disconnect when name changed
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

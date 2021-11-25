# NodeJS

## Download

[Sample](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.3/node/cmsn-node.zip)

## Requirement

- BLE 4.2 or later
- NodeJS 12.10.x or later
- Mac 10.15 or later
- Windows 10 build 10.0.15063 or later
- Linux

## Integration

package.json

```javascript
"devDependencies": {
    "@mapbox/node-pre-gyp": "^1.0.5",
    "node-gyp": "^8.1.0"
}

"dependencies": {
    "debug": "^4.3.2",
    "cmsn-noble": "3.1.4",
},
```

## Usage

### Init

```javascript
let cmsnSDK;
(async function main() {
  debug.enable("example:info");
  logI("------------- Example Main -------------");
  const useDongle = false; //nRF Dongle
  const logLevel = CMSNLogLevel.enum("info"); //info/error/warn
  cmsnSDK = await CrimsonSDK.init(useDongle, logLevel);

  cmsnSDK.on("error", (e) => logI(e));
  cmsnSDK.on("onAdapterAvailable", async () => await startScan());
  if (cmsnSDK.adapter.available) await startScan();
})();
```

### Dispose

```javascript
// when exit application, disconnect all devices & clean resources
process.on("SIGINT", async () => {
  logI({ message: `SIGINT signal received.` });
  await CrimsonSDK.dispose();
  logI("End program");
  process.exit(0);
});
```

### FAQ

{% page-ref page="faq.md" %}

### Scan

```javascript
async function startScan() {
  const targetDevices = new Map(); // (uuid: string, device)
  /// NOTE: user prompt, switch to pairing mode when connect crimson first time
  logI("Scanning for BLE devices");
  cmsnSDK.startScan(async (device) => {
    logI("found device", device.name);
    if (utils.array_contains(TARGET_DEVICE_NAME_ARRAY, device.name)) {
      targetDevices.set(device.id, device);
      logI(`targetDevices.size=${targetDevices.size}`);
    }

    if (targetDevices.size >= TARGET_DEVICE_COUNT) {
      await cmsnSDK.stopScan();
      await utils.sleep(500);

      for (let device of targetDevices.values()) {
        device.listener = exampleListener;
        await device.connect();
        // data stream listen, default return attention only
        // attention, meditation, socialEngagement
        // device.setDataSubscription(true, false, false);
      }
    }
  });
}
```

### Connect

```javascript
device.listener = exampleListener;
await device.connect();
```

### Disconnect

```javascript
// disconnect device
await device.disconnect();
```

### CrimsonDeviceListener

```javascript
const exampleListener = new CMSNDeviceListener({
  onError: (error) => {
    //CMSNError
    console.error("[ERROR]", error.message);
  },
  onConnectivityChanged: (device, connectivity) => {
    //Connectivity
    console.log({
      message: `[${device.name}] Connectivity changed to: ${CONNECTIVITY(
        connectivity
      )}`,
    });
    if (connectivity == CONNECTIVITY.enum("connected")) {
      device.pair((success, error) => {
        if (success) {
          device.startDataStream();
        } else {
          device.logError(error);
        }
      });
    }
  },
  onDeviceInfoReady: (device, deviceInfo) => {
    //deviceInfo
    console.log(device.name, `Device info is ready:`, deviceInfo);
  },
  onContactStateChanged: (device, contactState) => {
    //ContactState
    console.log(
      device.name,
      `Contact state changed to:`,
      CONTACT_STATE(contactState)
    );
  },
  onOrientationChanged: (device, orientation) => {
    //Orientation
    console.log(
      device.name,
      `Orientation changed to:`,
      ORIENTATION(orientation)
    );
  },
  onIMUData: (device, imu) => {
    //IMUData
    console.log(device.name, `IMU data received:`, imu);
  },
  onEEGData: (device, eeg) => {
    //EEGData
    console.log(device.name, "EEG data received:", eeg);
  },
  onBrainWave: (device, stats) => {
    //BrainWave
    console.log(device.name, "BrainWave data received:", stats);
  },
  onAttention: (device, attention) => {
    //Float
    console.log(device.name, `Attention:`, attention);
  },
  onMeditation: (device, meditation) => {
    //Float
    console.log(device.name, `Meditation:`, meditation);
  },
});
```

### ENUM

```javascript
const CONNECTIVITY = createEnum({
  connecting: 0,
  connected: 1,
  disconnecting: 2,
  disconnected: 3,
});
const CONTACT_STATE = createEnum({
  unknown: 0,
  contact: 1,
  no_contact: 2,
});
const ORIENTATION = createEnum({
  unknown: 0,
  normal: 1,
  upsideDown: 2,
});
```

### StartIMU

```javascript
// IMU SampleRate
const IMU = {
  SAMPLE_RATE: createEnum({
    unused: 0,
    sr125: 0x10,
    sr26: 0x20,
    sr52: 0x30,
    sr104: 0x40,
  }),
};

device.listener = exampleListener;
device.startIMU(IMU.SAMPLE_RATE.enum("sr104"));
```

### More

```javascript
class CMSNDevice
    id,
    name,
    connectivity,
    connect()
    disconnect()
    pair(cb)
    startDataStream(cb)
    stopDataStream(cb)
    startIMU(sampleRate, cb)
    stopIMU(cb)
    getSystemInfo(cb)
    shutdown(cb)
    // @param name length should be 4 ~ 18
    setDeviceName(name, cb)
    // @param {(string|number[])} color e.g. string '#FFAABB' or array [255, 0, 0]
    setLEDColor(color, cb)
    // @param timeSec => seconds to put device into sleep mode, 0 for no sleep
    setSleepIdleTime(timeSec, cb)
    // @param intensity => vibration intensity, 0 ~ 100
    setVibrationIntensity(intensity, cb)
}
```

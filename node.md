# NodeJS

## Download

[下载SDK及Example](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.1/node/node.zip)

## Requirement

* NodeJS 12.18.x or later
* Mac 10.15 or later
* Windows 10 build 10.0.15063 or later

## Integration

package.json

```json
"dependencies": {
    "cmsn-noble": "^3.0.3"
},

"devDependencies": {
"@mapbox/node-pre-gyp": "^1.0.5",
"node-gyp": "^8.1.0"
}
```

### when npm install, will download prebuilt lib if exists
* noble-v3.0.3-electron-v7.3-darwin-x64.tar.gz
* noble-v3.0.3-electron-v7.3-win32-x64.tar.gz
* noble-v3.0.3-node-v72-darwin-x64.tar.gz
* noble-v3.0.3-node-v72-win32-x64.tar.gz
* noble-v3.0.3-node-v83-darwin-x64.tar.gz
* noble-v3.0.3-node-v83-win32-x64.tar.gz

### electron

#### .npmrc

```text
runtime = electron
target = 7.3.3
npm_config_disturl=https://electronjs.org/headers
```

#### electron-builder.yml

```yaml
extraResources:
  - from: "node_modules"
    to: "node_modules"
    filter:
      - "cmsn-noble/**/*"
      - "bindings/**/*"
      - "napi-thread-safe-callback/**/*"
      - "node-addon-api/**/*"
      - "file-uri-to-path/**/*"
```

## Usage

### Init

```js
let cmsnSDK;
(async function main() {
    console.log('------------- Example Main -------------');
    const useDongle = false;
    cmsnSDK = await CrimsonSDK.init(useDongle);
    if (exampleListener.onError) {
        cmsnSDK.on('error', error => exampleListener.onError(error));
    }
})();
```

### Scan 扫描

#### 首次配对新设备时，需要先将头环设置为配对模式--&gt;蓝灯快闪

```js
cmsnSDK.startScan(async device => { 
    console.log(`found device, [${p.name}] ${p.address}`);
});
```

### Connect 连接

```js
await cmsnSDK.stopScan();
await utils.sleep(500);
device.listener = exampleListener;
await device.connect();
```

### Disconnect 断开连接

```js
// disconnect device
await device.disconnect();

// when exit application, disconnect all devices & clean resources
CrimsonSDK.dispose();
```

### CrimsonDeviceListener

```js
const exampleListener = new CMSNDeviceListener({
    onError: error => { //CMSNError
        console.error('[ERROR]', error.message);
    },
    onConnectivityChanged: (device, connectivity) => { //Connectivity
        console.log({ message: `[${device.name}] Connectivity changed to: ${CONNECTIVITY(connectivity)}` });
        if (connectivity == CONNECTIVITY.enum('connected')) {
            // 首次配对新设备时，需要先将头环设置为配对模式-->蓝灯快闪
            device.pair((success, error) => {
                if (success) {
                    // 开启EEG数据流
                    device.startDataStream();
                } else {
                    device.logError(error);
                }
            });
        }
    },
    onDeviceInfoReady: (device, deviceInfo) => { //deviceInfo
        console.log(device.name, `Device info is ready:`, deviceInfo);
    },
    onContactStateChanged: (device, contactState) => { //ContactState
        console.log(device.name, `Contact state changed to:`, CONTACT_STATE(contactState));
    },
    onOrientationChanged: (device, orientation) => { //Orientation
        console.log(device.name, `Orientation changed to:`, ORIENTATION(orientation));
    },
    onIMUData: (device, imu) => { //IMUData
        console.log(device.name, `IMU data received:`, imu);
    },
    onEEGData: (device, eeg) => { //EEGData
        console.log(device.name, "EEG data received:", eeg);
    },
    onBrainWave: (device, stats) => { //BrainWave
        console.log(device.name, "BrainWave data received:", stats);
    },
    onAttention: (device, attention) => { //Float
        console.log(device.name, `Attention:`, attention);
    },
    onMeditation: (device, meditation) => { //Float
        console.log(device.name, `Meditation:`, meditation);
    },
});
```

### ENUM

```js
// 头环连接状态
const CONNECTIVITY = createEnum({
    connecting: 0,
    connected: 1,
    disconnecting: 2,
    disconnected: 3,
});
// 佩戴状态，电极与皮肤接触良好
const CONTACT_STATE = createEnum({
    unknown: 0,
    contact: 1,    //佩戴好
    no_contact: 2, //未戴好
});
// 佩戴方向，检测是否佩戴反
const ORIENTATION = createEnum({
    unknown: 0,
    normal: 1,     //头环戴正
    upsideDown: 2, //头环戴反
});
```

### StartIMU 开启传输陀螺仪数据

```js
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
device.startIMU(IMU.SAMPLE_RATE.enum('sr104'));
```

### More

```js
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

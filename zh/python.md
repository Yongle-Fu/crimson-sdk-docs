# Python

## Download

[下载 SDK 及 Example](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.1/python/python.zip)

## Requirement

- Python 3.0 or later
- Mac 10.15 or later
- Windows 10 build 10.0.15063 or later

## Usage

```text
pip3 install -r requirements.txt
python3 gui.py //or python3 example.py
```

### FAQ

{% page-ref page="faq.md" %}

### Scan 扫描

#### 首次配对新设备时，需要先将头环设置为 _配对_ 模式--&gt;蓝灯快闪

```python
CMSNSDK.start_device_scan(on_found_device)
```

### Connect 连接

```python
print("Stop scanning for more devices")
CMSNSDK.stop_device_scan()

_target_device = device
_target_device.set_listener(DeviceListener())
_target_device.connect()
```

### Disconnect 断开连接

```python
// disconnect device
_target_device.disconnect()
```

### CMSNDeviceListener

```python
class DeviceListener(CMSNDeviceListener):
    def on_connectivity_change(self, connectivity):
        print("Connectivity:" + connectivity.name)
        if connectivity == Connectivity.connected:
            // 首次配对新设备时，需要先将头环设置为配对模式-->蓝灯快闪，然后开始扫描
            _target_device.pair(on_pair_response)
            // 开启EEG数据流
            _target_device.start_eeg_stream()

    //******************** NOTE: invoked after startEEG *******************
    def on_contact_state_change(self, contact_state):
        print("Contact state:" + contact_state.name)

    // NOTE: invoked after startIMU
    def on_orientation_change(self, orientation):
        print("orientation:" + orientation.name)

    def on_imu_data(self, imu_data):
        print("IMU Data:")
        print(imu_data)

    def on_eeg_data(self, eeg_data):
        if eeg_data.signal_type == AFEDataSignalType.lead_off_detection:
            print("Received lead off detection signal, skipping the packet.")
            return
        print("EEG Data:")
        print(eeg_data)

    def on_brain_wave(self, brain_wave):
        print("Alpha:" + str(brain_wave.alpha))

    def on_attention(self, attention):
        print("Attention: " + str(attention))

    def on_meditation(self, meditation):
        print("Meditation: " + str(meditation))
```

### Model

```python
// 头环连接状态
class Connectivity(IntEnum):
    connecting = 0
    connected = 1
    disconnecting = 2
    disconnected = 3

// 佩戴状态，电极与皮肤接触良好
class ContactState(IntEnum):
    unknown = 0
    contact = 1    //佩戴好
    no_contact = 2 //未戴好

// 佩戴方向，检测是否佩戴反
class Orientation(IntEnum):
    unknown = 0
    upward = 1   //头环戴正
    downward = 2 //头环戴反

// EEG
class EEGData:
    sequence_num = None
    sample_rate = None
    eeg_data = None

// 脑电频域波段能量
class BrainWave:
    delta = 0
    theta = 0
    alpha = 0
    low_beta = 0
    high_beta = 0
    gamma = 0

class IMUData:
    acc_data = None
    gyro_data = None
    euler_angle_data = None
    sample_rate = None
```

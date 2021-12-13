# Python

## Download

[Download](https://focus-resource.oss-accelerate-overseas.aliyuncs.com/universal/crimson-sdk-prebuild/1.1.0/python/python.zip)

## Requirement

- Python 3.0 or later
- Mac 10.15 or later
- Windows 10 build 10.0.15063 or later
- BLE 4.2 or later
## Usage

```text
pip3 install -r requirements.txt
python3 gui.py //or python3 example.py
```

### FAQ

{% page-ref page="faq.md" %}

### Scan

```python
CMSNSDK.start_device_scan(on_found_device)
```

### Connect

```python
print("Stop scanning for more devices")
CMSNSDK.stop_device_scan()

_target_device = device
_target_device.set_listener(DeviceListener())
_target_device.connect()
```

### Disconnect

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
            _target_device.pair(on_pair_response)
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
class Connectivity(IntEnum):
    connecting = 0
    connected = 1
    disconnecting = 2
    disconnected = 3

class ContactState(IntEnum):
    unknown = 0
    contact = 1
    no_contact = 2

class Orientation(IntEnum):
    unknown = 0
    upward = 1   //normal
    downward = 2 //upsidedown

// EEG
class EEGData:
    sequence_num = None
    sample_rate = None
    eeg_data = None

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

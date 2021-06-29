# Flutter

## Installation

```yaml
  headband_sdk:
    version: ^1.1.4
    hosted:
      name: headband_sdk
      url: http://dart-pub.brainco.cn:8081
```

## Init

```dart
HeadbandConfig.logLevel = Level.INFO;
HeadbandConfig.availableType = HeadbandAvailableType.crimson;
await HeadbandManager.init();
```

## Scan-扫描

```dart
await HeadbandManager.setScanMode(HeadbandScanMode.crimson);
if (Platform.isAndroid) {
    await [
        Permission.bluetooth,
        Permission.locationWhenInUse,
      ].request();
} else {
    Permission.bluetooth.request();
}
await HeadbandManager.startScan();

HeadbandManager.scanner.onFoundDevices.map((event) => event as List<ScanResult>)
```

## Pair-配对

### 首次配对新设备时，需要先将头环设置为配对模式--&gt;蓝灯快闪

```dart
try {
    await HeadbandManager.stopScan();
    await EasyLoading.show(status: '配对中...');
    await HeadbandManager.bindCrimson(result);
    await EasyLoading.showSuccess('配对成功!');
    await Navigator.of(context).pushReplacement(
        MaterialPageRoute(builder: (context) => CrimsonDeviceScreen()));
} catch (e, _) {
    loggerExample.i('$e');
    await EasyLoading.showError('配对失败');
    await HeadbandManager.startScan(); //restart scan
}
```


## CrimsonDelegate

```dart
enum HeadbandState {
  ///未连接
  disconnected,

  ///连接中
  connecting,

  ///已连接
  connected,

  ///佩戴检测中 -> 先检测电极与皮肤是否接触良好，然后检测是否戴正
  contacting,

  /// 佩戴反
  contactUpsideDown,

  ///佩戴检测通过 -> 佩戴良好 & 佩戴方向正确
  ///数据采集中 -> 持续采集脑电数据进行解析
  contacted,

  ///数据解析正常, 持续工作中
  analyzed
}
// listen streams
HeadbandProxy.instance.onStateChanged
HeadbandProxy.instance.onEEGData
HeadbandProxy.instance.onIMUData
HeadbandProxy.instance.onAttention
HeadbandProxy.instance.onMeditation
HeadbandProxy.instance.onBatteryLevelChanged
```

## More-更多详见SDK




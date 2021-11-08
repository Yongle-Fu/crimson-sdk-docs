# 常见问题说明

## [配对连接](https://www.yuque.com/docs/share/4afe9d08-cf4b-42fb-93da-0ee239830090)

头环**首次连接**到新设备时，必须先切换到**配对模式**使用

- 开关机，关机状态短按电源键开机，开机状态短按电源键关机
- 配对模式，关机状态长按电源键直至头环振动两次，此时 LED 为蓝灯快闪
- 工作模式，开机状态默认模式，此时 LED 为蓝灯慢闪

扫描->扫描成功->连接->连接成功->配对/校验配对信息-->配对/校验配对信息成功->StartEEG

```java
if 存在配对记录
    scan->connect->validatePairInfo->
    if validatePairInfo_success: StartEEG
    else if validatePairInfo_fail_error_code == 4: // 提示去pair
    else //提示retry

else
    // 提示用户切换到配对模式
    // 可过滤掉不处于配对模式的设备
    scan->connect->pair->(if success)->存储配对记录 & StartEEG
```

## [LED 灯光](https://www.yuque.com/docs/share/a0cee022-8f4e-4f06-9221-e05cfec2b608)

```java
// 在连接并配对/校验配对信息成功之后，灯光由软件/app控制，其他情况固件端会进行接管

// 灯光颜色，固件端设置的
colorDisconnected: 0x0102A0 // RGB 1,2,160,
colorUnContact: 0x6E6E64 // RGB 110,110,100 配对成功
蓝灯快闪 //配对模式
蓝灯慢闪 //工作模式

白灯闪烁 //OTA
红灯闪烁 //电量低/充电
黄灯闪烁 //电池温度异常

colorDisbaled: 0x0 // RGB 0,0,0 黑色即不显示灯光
colorContact: 0xffffff // RGB 255,255,255 佩戴通过
```

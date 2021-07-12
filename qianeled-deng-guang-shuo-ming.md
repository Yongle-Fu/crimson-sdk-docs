# 前额LED灯光说明

```java
// 在连接并配对/校验配对信息成功之后，灯光由软件/app控制，其他情况固件端会进行接管

// 灯光颜色，固件端设置的
colorDisconnected: 0x0102A0 // RGB 1,2,160,
colorUnContact: 0x6E6E64 // RGB 110,110,100 配对成功
蓝灯快闪 //配对模式
蓝灯慢闪 //工作模式
白灯闪烁 //OTA模式
红灯闪烁 //电量低/充电

colorDisbaled: 0x0 // RGB 0,0,0 黑色即不显示灯光
colorContact: 0xffffff // RGB 255,255,255 佩戴通过
```


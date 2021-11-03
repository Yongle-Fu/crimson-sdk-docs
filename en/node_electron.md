# Electron

## Download

[Sample](https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/crimson-sdk-prebuild/1.0.3/node/electron.zip)

## Requirement

* BLE 4.2 or later
* NodeJS 12.10.x or later
* Mac 10.15 or later
* Windows 10 build 10.0.15063 or later
* Linux

#### .npmrc

```text
runtime = electron
target = 8.2.5
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


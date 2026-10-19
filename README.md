# build-ipa

用一条命令，将 Xcode 项目打包为带 Ad Hoc 临时签名的 iPhone / iPad IPA。单文件 Python 脚本，无需安装第三方 Python 库或 fastlane。

A single-file Python CLI that builds an iOS Xcode project and packages an ad-hoc-signed IPA. No third-party Python packages or fastlane required.

## 功能

- 支持 `.xcodeproj` 和 `.xcworkspace`，自动发现项目，优先使用工作区。
- 自动选择与项目同名的 Scheme，或唯一 Scheme；也可手动指定。
- 使用 Release 配置和真机 iOS SDK 归档，无需配置构建签名账号。
- 生成标准 `Payload/App.app` IPA 结构。
- 自动清理临时构建文件；构建失败不会覆盖已有 IPA。
- 支持自定义输出路径，支持含空格的路径。

## 环境要求

- macOS 和完整 Xcode（仅安装 Command Line Tools 不够，需要 iOS SDK）。
- Python 3.8 或更新版本。
- `xcodebuild` 与 macOS 自带的 `ditto`、`codesign` 可用。
- 项目本身能够构建，所需依赖已安装，Xcode 首次启动设置已完成。

检查环境：

```bash
python3 --version
xcodebuild -version
```

若 Xcode 开发工具目录未正确选择：

```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

## 下载与使用

```bash
git clone https://github.com/Gitawake/build-ipa.git
cd build-ipa
python3 build-ipa /path/to/YourApp
```

也可以直接下载仓库中的 `build-ipa` 文件，通过 `python3` 运行。无须 `pip install`。

```bash
# 传入项目目录
python3 build-ipa /path/to/YourApp

# 直接传入 Xcode 项目或工作区
python3 build-ipa /path/to/YourApp/YourApp.xcodeproj
python3 build-ipa /path/to/YourApp/YourApp.xcworkspace

# 指定 Scheme 和输出文件
python3 build-ipa /path/to/YourApp --scheme YourApp --output ~/Desktop/YourApp.ipa

# 查看帮助
python3 build-ipa --help
```

如需直接执行：

```bash
chmod +x build-ipa
./build-ipa /path/to/YourApp
```

默认输出：所选 `.xcodeproj` 或 `.xcworkspace` 所在目录下的 `build/ipa/App名称.ipa`。

## 参数

| 参数 | 说明 |
| --- | --- |
| `project` | 必填：项目目录、`.xcodeproj` 或 `.xcworkspace` 路径 |
| `--scheme` | 指定 Scheme；存在多个且无法自动选择时必须提供 |
| `--output` | 指定输出文件，扩展名必须为 `.ipa` |
| `-h`, `--help` | 查看帮助 |

## 安装与适用范围

输出是**带 Ad Hoc 临时签名的 IPA**，需通过侧载工具签名后才能安装到 iPhone / iPad。该脚本不管理证书或描述文件，也不生成可直接上传 App Store 的签名导出包。

项目自身使用 CocoaPods、Swift Package Manager 或其他依赖时，仍需准备相应依赖；“无第三方依赖”指脚本本身。需要特定权限的 App、扩展和嵌套组件，安装时仍取决于签名工具和账号权限。

## 常见问题

- **找不到 `xcodebuild` / 没有 iOS SDK**：安装完整 Xcode，完成首次启动配置并选择开发工具目录。
- **找到多个项目**：直接传入要构建的 `.xcodeproj` 或 `.xcworkspace` 路径。
- **无法确定 Scheme**：使用 `--scheme` 指定；需要在 CI 使用时，在 Xcode 中共享 Scheme。
- **没有唯一 `.app`**：确认 Scheme 构建的是 iOS App，App Target 的 `Skip Install` 为 `NO`。
- **编译失败**：查看 `xcodebuild` 输出，先在 Xcode 中解决项目自身的编译问题。
- **IPA 无法直接安装**：未签名包需要另外签名；检查签名工具的安装要求。

## 验证

已在 LiveBox 项目上完成 Release 真机归档与 IPA 打包，检查 ZIP 完整性、`Payload/LiveBox.app/Info.plist`、版本信息和 App Icon。此验证不代表已覆盖所有 Xcode 项目类型。

## License

MIT，见 [LICENSE](LICENSE)。

## SideStore 兼容性

脚本会在打包前使用 macOS 自带的 `codesign` 添加 Ad Hoc 临时签名，无需 Apple 开发者证书或第三方 Python 库。嵌套 Framework、动态库、App Extension 和子应用按从内到外的顺序签名，并校验签名；已有签名中的 entitlements 会保留。完全未签名构建中的权限不会凭空补齐，依赖特殊权限的项目仍需准备对应配置。

此处的 Ad Hoc 是 `codesign --sign -` 的本地临时签名，不是 Apple 的 Ad Hoc 分发证书。安装时仍需由 SideStore 等侧载工具使用你的账号重新签名。LiveBox 的临时签名测试包已由用户确认可通过 SideStore 安装；其他项目仍需实际验证。

旧版完全未签名的 LiveBox IPA 曾触发 `The app is in an invalid format. SideStore.OperationError 2`。请重新运行构建，下载新产物；旧文件不会自动更新。

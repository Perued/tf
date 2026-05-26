# tf

## iOS TestFlight 自动签名与上传配置

本仓库当前包含示例 `fastlane` 与 GitHub Actions 配置，用于 iOS 应用自动构建、签名、上传到 TestFlight。

### 已添加文件

- `Fastfile`: fastlane lane，用于通过 App Store Connect API Key 和 `match` 进行构建上传。
- `Appfile`: 应用标识和 Apple ID 配置。
- `.github/workflows/ios-testflight.yml`: GitHub Actions workflow，在 `main` 分支 push 时触发。
- `Gemfile`: fastlane 依赖管理。
- `.ruby-version`: 推荐 Ruby 版本。

### 需要填写的项目配置

- 在 `Appfile` 中替换 `app_identifier("com.example.app")` 为你的 App Bundle ID。
- 在 `Appfile` 中替换 `apple_id("your@apple.id@example.com")` 为你的 App Store Connect 登录邮箱。
- 在 `Fastfile` 中将 `build_app(scheme: "App")` 中的 `App` 修改为你的 Xcode scheme。
- 如果你的项目是 workspace，请把 build_app 改成：
	`build_app(workspace: "YourApp.xcworkspace", scheme: "YourScheme")`。

### GitHub Secrets 配置

- `APP_STORE_CONNECT_API_KEY`: App Store Connect API Key JSON 内容。
- `MATCH_GIT_URL`: fastlane match 私有仓库 URL，用于保存证书和描述文件。
- `MATCH_PASSWORD`: fastlane match 仓库加密密码。

如果你不使用 `match`，也可以改为手动导入证书并在 workflow 中使用 `SIGNING_CERT_P12` / `SIGNING_CERT_P12_PASSWORD` / `SIGNING_PROFILE`。

### 快速使用步骤

1. 在 App Store Connect → Users and Access → Keys 创建 API Key，下载 `.json`。
2. 在 GitHub 仓库 Secrets 中增加 `APP_STORE_CONNECT_API_KEY`、`MATCH_GIT_URL`、`MATCH_PASSWORD`。
3. 修改 `Appfile` 与 `Fastfile` 中的 app id、Apple ID、scheme、workspace 配置。
4. 提交代码并 push 到 `main`。
5. 在 Actions 里查看 `.github/workflows/ios-testflight.yml` 的执行情况。

### 直接上传现成 IPA

如果你已经有现成的 `IPA` 文件，可以使用 `upload_ipa` lane 直接上传，而无需在 CI 中重新签名。

1. 将 `ipa` 文件放到仓库中，建议路径为 `ipa/app.ipa`，或在 `IPA_PATH` secret 中指定路径。
2. 在 GitHub Secrets 中添加：
   - `IPA_PATH`：例如 `ipa/app.ipa`
3. workflow 会执行 `fastlane upload_ipa`，使用 `upload_to_testflight(ipa: ipa_path)` 直接上传。

```yaml
      - name: Run Fastlane upload_ipa lane
        env:
          IPA_PATH: ${{ secrets.IPA_PATH }}
        run: fastlane upload_ipa
```

### 注意事项

- `fastlane match` 推荐用于统一管理证书，避免 CI 中的 keychain 配置问题。
- API Key 方式比用户名/密码更稳定，避免 2FA 人工交互。
- 若你使用 Xcode workspace，请在 `Fastfile` 中提供 `workspace:` 参数。
- 如果 runner 报 codesign 权限错误，可补充 `security set-key-partition-list`。
# tf
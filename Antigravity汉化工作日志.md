# Antigravity 本体全局汉化工作日志

日期：2026-05-23  
目标：只汉化 `/Applications/Antigravity.app` 本体，不处理 `Antigravity IDE`。

## 一、最终结果

已完成：

1. Antigravity 本体主界面、菜单栏、设置页、反馈页等主要 UI 汉化。
2. Antigravity 本体自动更新已在应用代码层禁用。
3. 已安装用户级 LaunchAgent，登录/重启后会自动检查并保持汉化版。
4. 已重新写入 `app.asar` 完整性哈希，并完成 ad-hoc 签名。
5. 已生成桌面 skill 技能包：`/Users/lizeju/Desktop/antigravity-zh-cn-maintainer`。

未处理范围：

- 不再修改 `Antigravity IDE`。
- 不处理第三方网页、模型名称、用户已有项目/对话标题等动态业务内容。

## 二、重要路径

Antigravity 本体：

```text
/Applications/Antigravity.app
```

当前汉化维护 payload：

```text
/Users/lizeju/Library/Application Support/Antigravity/zh-cn-patch/
```

开机/登录自动修复 LaunchAgent：

```text
/Users/lizeju/Library/LaunchAgents/com.lizeju.antigravity.zh-cn-repair.plist
```

原始备份：

```text
/Users/lizeju/Documents/Codex/2026-05-23/files-mentioned-by-the-user-2026/antigravity-backup-20260523-2249/app.asar
/Users/lizeju/Documents/Codex/2026-05-23/files-mentioned-by-the-user-2026/antigravity-backup-20260523-2249/Info.plist
```

当前汉化 asar 哈希：

```text
20abffd9d3ff90623f6067ccd24222194e0d6f44ba0aa81b0f02f868f2e673c3
```

## 三、使用的工具和技能

使用的 Codex skill：

- `skill-creator`：用于生成桌面技能包 `antigravity-zh-cn-maintainer`。

使用的工具/命令：

- `rg`：搜索 Antigravity 解包后的 JS 代码和英文文案。
- `sed`：查看关键源码片段。
- `asar pack`：重新打包 Electron `app.asar`。
- `node --check`：检查修改后的 JS 文件语法。
- `shasum -a 256`：计算 `app.asar` 哈希。
- `/usr/libexec/PlistBuddy`：写入 `Info.plist` 里的 `ElectronAsarIntegrity.Resources.app.asar.hash`。
- `codesign --force --deep --sign -`：对修改后的 Antigravity.app 做本机 ad-hoc 签名。
- `codesign --verify --deep --strict --verbose=2`：验证签名有效。
- `osascript`：退出 Antigravity，避免替换运行中文件。
- `launchctl`：加载用户级 LaunchAgent。
- `Computer Use`：打开真实 Antigravity 界面做视觉和可访问性树验证。

## 四、主要修改内容

### 1. 系统语言和用户配置

对 Antigravity 本体设置中文语言偏好：

```text
~/Library/Application Support/Antigravity/User/locale.json
com.google.antigravity AppleLanguages
```

说明：这只能覆盖一小部分系统/原生语言逻辑，不能完成全局汉化，所以继续修改 Electron 资源包。

### 2. app.asar 资源包汉化

解包并修改：

```text
antigravity-asar-work/dist/
```

关键文件：

- `loadingOverlay.js`：启动页 `Loading Antigravity` 改为 `正在加载 Antigravity`。
- `menu.js`：菜单栏 File/Edit/View/Window/Help 等汉化。
- `tray.js`：代理运行状态汉化。
- `ideInstall/wizardHtml.js`：Antigravity 本体内的 IDE 安装向导文案汉化。
- `updater.js`：原有更新菜单文案汉化，后续改为禁用自动更新。
- `preload.js`：加入运行时 DOM 汉化层。

`preload.js` 是覆盖面最大的补丁，包含：

- `zhText`：完整文本映射。
- `zhPartialText`：带数字、模型名、刷新时间、链接 Markdown 的组合文本匹配。
- `zhInlineText`：处理被 React/Markdown 拆开的英文片段。
- `MutationObserver`：监听页面后续动态渲染。
- `Element.prototype.attachShadow` hook：处理 Shadow DOM。
- 属性处理：`aria-label`、`title`、`placeholder`、`alt`。

已覆盖的主要页面：

- 主界面
- 设置
- 账户
- 权限
- 外观
- 模型
- 自定义
- 浏览器
- 应用
- 快捷键
- 提供反馈

### 3. 自动更新禁用

修改了：

```text
antigravity-asar-work/dist/updater.js
antigravity-asar-work/dist/ipcHandlers.js
```

效果：

- 启动后不再自动检查更新。
- 不再定时每小时检查更新。
- 不再自动下载更新。
- 不再退出时自动安装更新。
- `updater:quit-and-install` IPC 被拦截。

原因：官方更新会覆盖 `app.asar` 和签名，导致汉化失效。

### 4. 开机/登录自动保持汉化

已安装：

```text
/Users/lizeju/Library/Application Support/Antigravity/zh-cn-patch/antigravity-zh-cn-repair.sh
/Users/lizeju/Library/LaunchAgents/com.lizeju.antigravity.zh-cn-repair.plist
```

LaunchAgent 设置：

```text
RunAtLoad = true
StartInterval = 1800
```

含义：

- 用户登录时自动检查一次。
- 此后每 30 分钟检查一次。
- 如果发现 `/Applications/Antigravity.app/Contents/Resources/app.asar` 哈希不是当前汉化版，就自动恢复汉化包、恢复匹配的 `Info.plist` 并重新签名。

日志：

```text
/Users/lizeju/Library/Application Support/Antigravity/zh-cn-patch/repair.log
```

当前验证结果：

```text
Chinese patch already active.
```

## 五、验证记录

已执行：

```bash
node --check antigravity-asar-work/dist/preload.js
node --check antigravity-asar-work/dist/updater.js
node --check antigravity-asar-work/dist/ipcHandlers.js
codesign --verify --deep --strict --verbose=2 /Applications/Antigravity.app
launchctl print gui/501/com.lizeju.antigravity.zh-cn-repair
```

结果：

- JS 语法检查通过。
- `codesign` 显示 `valid on disk` 和 `satisfies its Designated Requirement`。
- LaunchAgent 已加载，`runs = 1`，退出码为 `0`。
- Antigravity 可正常打开，启动页、菜单栏、主界面、设置页主要内容均显示中文。

## 六、风险和注意事项

1. 这是本地补丁，不是官方中文包。
2. 修改 app 包会破坏 Google 原始签名，因此已改为本机 ad-hoc 签名。
3. 如果手动安装新版 Antigravity，可能覆盖当前汉化，需要重新打补丁。
4. 自动更新已禁用，目的是避免汉化被自动覆盖。
5. 如果后续 Antigravity 页面结构大改，`preload.js` 的词表和 DOM 规则可能需要补充。

## 七、恢复官方原版

如需恢复：

1. 退出 Antigravity。
2. 用备份文件覆盖：

```bash
cp /Users/lizeju/Documents/Codex/2026-05-23/files-mentioned-by-the-user-2026/antigravity-backup-20260523-2249/app.asar /Applications/Antigravity.app/Contents/Resources/app.asar
cp /Users/lizeju/Documents/Codex/2026-05-23/files-mentioned-by-the-user-2026/antigravity-backup-20260523-2249/Info.plist /Applications/Antigravity.app/Contents/Info.plist
codesign --force --deep --sign - /Applications/Antigravity.app
```

3. 如不再需要自动修复：

```bash
launchctl bootout gui/501 /Users/lizeju/Library/LaunchAgents/com.lizeju.antigravity.zh-cn-repair.plist
```

## 八、桌面技能包

技能包路径：

```text
/Users/lizeju/Desktop/antigravity-zh-cn-maintainer
```

用途：

- 给后续 Codex 复用这次 Antigravity 汉化流程。
- 记录只处理 Antigravity 本体、不处理 Antigravity IDE 的边界。
- 提供重新应用汉化、禁用自动更新、安装 LaunchAgent、验证签名和哈希的流程。

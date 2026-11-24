# WeChat Sentinel
实时监控微信聊条记录 在群聊中找到需要的信息（半成品）

## 添加监控

双击 `Start-Sentinel-Floating.cmd`，或在 VS Code 打开 `backend` 后按 F5。使用便携版时，解压整包并打开 `WeChatSentinel.exe`。

1. 打开目标微信群。在悬浮窗点「添加微信窗口」，倒计时内把鼠标移到群聊窗口内并停住。
2. 核对群名。键入关键词后点「添加」或按 Enter，可以用逗号、顿号一次输入多个。选择「任一关键词命中」或「全部关键词命中」，点「确认并开始」。
3. 重复添加其他群。让各群使用独立聊天窗口并排保持可见，最多同时监控四个窗口。悬浮窗会显示每个窗口是否可采集。
4. 监控中点「添加并保存」可使新关键词生效，移除关键词或切换匹配方式也会自动保存。上方「已保存」显示写入成功的关键词数。关闭悬浮窗会停止采集。下次打开时保留群配置和关键词，需要重新选中微信窗口。

桌面锁定、窗口重叠或最小化会让相应群显示受限。读取依赖该微信版本向 Windows 暴露的控件。更多信息见 [兼容性与验证](docs/uia-compatibility.md)。

## 组合关键词与命中清理

在悬浮窗选择「任一完整关键词组」，输入一组如「报名、周四」并添加，再输入下一组如「组会、地点」。只有同一条消息集齐任意一组的所有词才收录。也可以在网页「匹配规则」里用每行一组的方式设置，并用「测试规则」预览。

在「命中记录」点一行的「删除」并确认，可从本地库移除该条记录及相应命中和通知。群聊内容留在微信。

监控开始后可点「群窗置顶」，帮助已选群窗在普通窗口上方保持可见。旁边的「面板置顶」只控制 Sentinel 悬浮窗。暂停、关闭或改绑时会恢复本程序设置的群窗置顶。OCR 仍需手选区域可见，其他置顶窗口盖住选区、微信最小化或锁屏时会暂停。

## OCR 速度与历史清空

本地 OCR 默认每秒尝试截取一次已授权选区，模型常驻并复用未变气泡的识别结果。命中页第一页自动刷新。可以在「诊断与设置」调整 OCR 间隔。选区被遮挡、移出屏幕、最小化或锁屏时仍会暂停，快速连续滚动也可能让消息在两次画面之间滑出选区。

在「命中记录」或「历史记录」点击「清空本地历史」并确认，会清空本机消息、命中、通知日志和导入批次。群配置、关键词规则和采集状态会保留。请先完成或取消正在运行的导入。

## Windows 源码安装与运行

在项目根目录打开 PowerShell。需要 Python 3.11 或更新版本与 Node.js 20.19 或更新版本。便携包已包含运行环境，可直接解压启动。

```powershell
Push-Location backend
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.lock -e '.[dev,package]'
Pop-Location
npm.cmd --prefix frontend ci
npm.cmd --prefix frontend run build
.\Start-Sentinel-Floating.cmd
```

完整浏览器控制台可用 `.\scripts\run.ps1` 启动，默认地址为 `http://127.0.0.1:8765`。运行 `.\scripts\diagnose.ps1` 或 `WeChatSentinel.exe --diagnose` 可检查 Windows UI Automation 的可用性，诊断不会输出聊天正文。

## 本地 OCR 原型

微信 4.2.0.52 的真机诊断未读到消息正文控件，可用独立 OCR 原型试读授权群的可见文字。解压 OCR 便携版后，双击其中的 `WeChatSentinel.exe` 或 `Start-Sentinel-OCR.cmd`。OCR 窗口标题带有「本地 OCR」，并提供「框选微信窗口」按钮。源码环境使用 Python 3.13 x64，先在项目根目录安装可选依赖，再运行 `Start-Sentinel-OCR.cmd`。

```powershell
.\backend\.venv\Scripts\python.exe -m pip install --only-binary=:all: -r backend/requirements-ocr.lock
.\Start-Sentinel-OCR.cmd
```

1. 打开一个有权处理的独立群聊窗口，滚到最新消息，确保能看到至少一个完整的文本气泡。把悬浮窗放在群窗旁边。
2. 点「框选微信窗口」，把鼠标移到群窗内。先拖框圈住一行群名和括号里的成员数，避开按钮图标，再圈住消息列表，留出气泡边缘并避开输入框、侧栏和聊天背景图。按 Esc 可取消。
3. 核对识别出的群名，添加关键词并选「任一」或「全部」。点「确认并开始」，等状态变成「可采集」再发测试消息。监控中新增关键词时，点「添加并保存」，等提示「已保存并生效」后再用新消息测试。
4. 「查看命中」打开本地后台。原型一次绑定一个群，窗口移动后仍核对标题，调大小时需重新框选。暂停后可重新选窗。重启时保留关键词和命中记录，需要重新授权窗口和选区。

模型随 OCR 便携包提供，运行时在本机识别。图片只在内存中短暂处理，数据库只保存匹配的文本。截图、未命中正文、推断出的发送者和原消息时间均不入库。窗口被遮挡、最小化、锁屏、标题不匹配或消息断档时，采集会受限并在恢复后重建基线。普通主题下的完整文字气泡阅读效果较好，很长的消息、主题变化和快速滚动可能漏采。

## 测试与打包

```powershell
Push-Location backend
.\.venv\Scripts\python.exe -m pytest -q
Pop-Location
npm.cmd --prefix frontend test
.\scripts\build.ps1 -SkipInstall
```

打包结果位于 `build/portable/WeChatSentinel`。复制或压缩整个目录交付，解压后运行里面的 `WeChatSentinel.exe`。若打包 OCR 原型，构建时加 `-IncludeOcr` 并用独立的 `-OutputDirectory`。可用 `WeChatSentinel.exe --ocr-self-test` 在离线环境诊断模型是否齐全。

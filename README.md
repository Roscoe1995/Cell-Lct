# Cell-lct

把参考图重建为 Adobe Illustrator 中可继续修改的矢量路径和真实文本，并追加到用户已经打开的画板。

`Cell-lct` 先记录文字，再从去字后的工作图识别路径；返回的 SVG 只解析一次，随后按原图层级分批写入 Illustrator。已有画板内容保持不变，最终同时保留可编辑 AI 文件和导出的 PNG。

## 你会得到

- Illustrator 原生可编辑路径；
- 恢复为真实 SVG 文本的文字内容；
- 保留复合路径、镂空和原图绘制顺序；
- 可续画的几何缓存与阶段结果；
- 完成后的 AI 文件和 PNG 预览。

## 选择安装版本

- **最新 `main`**：包含可选的 [cell_no_ai](https://github.com/yrui-cmd/cell_no_ai) 隐藏水印处理衔接，适合使用当前完整流程。
- **固定 `v0.2.1`**：经过锁定和发行验证的 Windows 稳定版，不包含后续接入的新功能。

最新流程会在去字后询问是否使用 `cell_no_ai`。选择使用时，必须先接收处理结果再继续矢量识别；选择跳过时直接使用已核对的去字图。安装会同步该独立 Skill，但安装依赖不等于授权付费处理。第三方处理效果以服务返回和后续验证为准。

从最新 `main` 安装：

```powershell
git clone --branch main --depth 1 https://github.com/yrui-cmd/cell-lct.git
Set-Location .\cell-lct
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

下方的固定 Tag、校验文件和环境契约对应 `v0.2.1` 稳定版。

## 稳定版保证

- 固定源码版本：Git Tag `v0.2.1`。
- 固定 Python 依赖：`requirements.lock`。
- 固定运行契约：`runtime-lock.json`。
- 一键安装与诊断：`setup.ps1`、`doctor.ps1`。
- Windows 自动端到端测试，另提供 Illustrator 2026 人工触发真机测试。
- Release ZIP 配套独立 SHA256 文件。
- 不提供 Marketplace 安装入口；从固定 Tag 或 Release ZIP 安装。
- API Key 不随项目分发，只通过当前 Windows 账户的 DPAPI 加密保存。
- 单张图片预计消耗超过 1 额度时，必须在上传 API 前取得用户确认；拒绝时不上传。确认后处理与下载不再重复询问。

## 环境要求

- Windows 10/11 x64
- Codex Desktop，且当前任务具备内置 Image 2 图片编辑能力
- Adobe Illustrator 2026（30.x）
- PowerShell 5.1 或更高版本
- Python 3.11–3.14

## 从固定 Tag 一键安装

```powershell
git clone --branch v0.2.1 --depth 1 https://github.com/yrui-cmd/cell-lct.git
Set-Location .\cell-lct
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

首次安装会安装锁定依赖、复制 Skill、在终端中安全提示录入 API Key，并运行诊断。API Key 不应写进命令、仓库、截图或聊天记录。

如果已经存在同名 Skill，并确认要替换：

```powershell
powershell -ExecutionPolicy Bypass -File .\setup.ps1 -Force
```

安装后重启 Codex，并新建任务。先由用户打开 Illustrator 2026 和目标文档，然后发送：

```text
使用 $cell-lct，根据我上传的内容或图片在当前 Illustrator 画板中作图，保留全部已有内容。
```

## 从 Release ZIP 安装

下载同一版本的两个文件：

- `cell-lct-v0.2.1.zip`
- `cell-lct-v0.2.1.zip.sha256`

验证后解压并运行 `setup.ps1`：

```powershell
$zip = '.\cell-lct-v0.2.1.zip'
$expected = ((Get-Content "$zip.sha256") -split '\s+')[0]
$actual = (Get-FileHash $zip -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw 'SHA256 校验失败，停止安装。' }
Expand-Archive $zip -DestinationPath .\cell-lct-v0.2.1
Set-Location .\cell-lct-v0.2.1\cell-lct-v0.2.1
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

## 诊断

```powershell
.\doctor.ps1
.\doctor.ps1 -VerifyApi -RequireIllustratorOpen
.\doctor.ps1 -Json
```

诊断只检查 Illustrator 注册和进程状态，不会启动、重启、聚焦、最大化或关闭 Illustrator。

## 工作方式

- 先记录参考图文字的内容、位置、尺寸、字体、字重、颜色、旋转、对齐和层级。
- Image 2 只清除文字，保留箭头、框、坐标轴、热图、图例、科研主体和原布局。
- 去字后可选调用 `cell_no_ai`；选择使用时接收处理图片后再进入矢量识别，跳过时使用去字图。矢量返回后先在 Master SVG 中合并真实可编辑 `<text>`。
- SVG 只解析一次并建立几何缓存，全程复用一个 Illustrator 连接。
- 普通批次为 20–50 条路径，复杂路径可单独处理。
- 不删除、不隐藏、不替换已有画板内容；PNG 只在结束时导出。

## 测试与发行

```powershell
.\tests\test-package.ps1
.\tests\test-windows-e2e.ps1
.\build-release.ps1
```

Illustrator 真机测试会向当前打开的文档写入测试图，只能在一次性测试文档中显式运行：

```powershell
.\tests\test-illustrator-e2e.ps1 -ConfirmDisposableOpenDocument
```

生成的 ZIP 与 SHA256 位于 `dist`。CI 使用 Windows runner 执行离线绘制链路测试；Illustrator 真机链路仅在带 Illustrator 2026 的自托管 Windows runner 上人工触发。

## 可复现性边界

仓库可以固定 Skill、脚本、依赖、测试和发行文件，但无法把 Codex 内置 Image 2 或 Adobe Illustrator 本体打包进去。另一台电脑要获得一致流程，必须满足 `runtime-lock.json` 中的环境契约，并自行配置 DPAPI API Key。

完整插件源码位于 `plugins/cell-lct`，安装器只把其中的 `cell-lct` Skill 部署到用户的 Codex Skills 目录。原版 Cell-lct 与 Cell-lct 可以并存。

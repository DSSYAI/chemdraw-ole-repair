# ChemDraw ↔ Word OLE 集成修复

ChemDraw 结构复制/拖进 Word 后变成图片、无法双击编辑，或插入对象时报
"用于创建此对象的程序是 ChemDraw，您的计算机尚未安装此程序或此程序无响应" / "Class not licensed"
—— 这个脚本包用来**先诊断、再一键修好**。

作者实测环境：Windows ＋ ChemDraw 20.0 ＋ Word。原理是把 ChemDraw 重新注册为 OLE 服务器（`/regserver`），
并把两个 CLSID 的 `LocalServer32` 指回本机 ChemDraw 的真实路径。

## 它怎么工作

| 文件 | 作用 |
|---|---|
| `诊断.bat` | **只读**检查，不改任何东西；在桌面生成《ChemDraw诊断报告.txt》，给出"可修复 / OLE 正常"的结论，并检查默认打印机 |
| `修复.bat` | 按诊断结果重注册 OLE 服务器并改 CLSID 指向；会请求管理员权限 |

两个脚本都**不含任何硬编码路径**（ChemDraw 安装位置由脚本自己探测），也不需要联网。

## 用法（按顺序）

1. **双击 `诊断.bat`**（不需要管理员，只读检查）
   会在桌面生成《ChemDraw诊断报告.txt》。看"结论"和"[5] 默认打印机"：
   - 结论为「可修复」（报 `80040112`，或指向的文件打不开）→ 走第 2 步
   - 结论为「OLE 正常」→ 病因不在注册表，请把报告发出来再议
   - [5] 显示没有默认打印机 → 先把默认打印机设成 `Microsoft Print to PDF` 再重试

2. **运行 `修复.bat`**：运行前关闭已打开的 ChemDraw 窗口，建议**右键 → 以管理员身份运行**
   （直接双击会请求提权，点"是"；若窗口一闪而过，就改用右键管理员运行）
   - 显示「修复成功」→ 去 Word 里试 `插入 → 对象 → CS ChemDraw Drawing`，或复制粘贴后双击
   - 显示「本来就正常」→ 没病可修，看诊断报告找真凶
   - 显示「修复后仍失败」→ 把桌面那份报告发出来

## 手动等价操作（第二步在管理员命令行里做）

把 `你的路径` 换成诊断报告 `[1]` 里给出的真实路径：

```bat
"你的路径\ChemDraw.exe" /regserver

reg add "HKCR\CLSID\{41BA6D21-A02E-11CE-8FD9-0020AFD1F20C}\LocalServer32" /ve /d "\"你的路径\ChemDraw.exe\"" /f
reg add "HKCR\CLSID\{8E56081F-2AC3-47D2-9DB6-2D4C4D86806A}\LocalServer32" /ve /d "\"你的路径\ChemDraw.exe\" /Automation" /f
```

> 改动只发生在 `HKEY_CLASSES_ROOT\CLSID` 下这两个键的 `LocalServer32` 值上，不碰许可与激活状态。

## 许可

MIT，见 [LICENSE](LICENSE)。
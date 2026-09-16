# ClashMetaForAndroid Mod

给 [MetaCubeX/ClashMetaForAndroid](https://github.com/MetaCubeX/ClashMetaForAndroid) 加一个小改动：把「访问控制模式」和「访问控制应用包列表」从 **设置 → 网络** 提到 **主页面「日志」的上方**。其余功能一律不动。

本仓库**只有补丁和云端构建脚本**，不保存上游源码。每次构建都实时拉取上游最新 release 标签、打补丁、在 GitHub Actions 里签名编译，所以你不需要在本地安装 Android SDK / Go / NDK。

## 主页面效果

| 新增项 | 行为 |
| --- | --- |
| 访问控制模式 | 副标题显示当前模式（允许所有应用 / 仅允许已选择的应用 / 不允许已选择的应用），点击弹出三选一对话框。Clash 运行中该行变暗，点击提示「选项在 Clash 运行时不可用」（与设置页一致，需先停止 Clash 再改）。 |
| 访问控制应用包列表 | 副标题显示「已选 N 个应用」，点击进入原有的应用勾选页；运行中也可进入，该页会在选择变化后自动重启服务。 |

设置 → 网络 里的原有两项保留，行为完全不变。

## 安装

1. 打开 [Releases](../../releases)，下载 `v*-mod.*` 里的 APK（64 位手机可下 `arm64-v8a`，不确定就用 `universal`）。
2. **首次安装需要先卸载官方版**：本仓库的 APK 包名与官方一致（`com.github.metacubex.clash.meta`），但签名不同，Android 不允许直接覆盖。
3. 之后本仓库发布的新版本可以直接覆盖安装（签名固定不变）。

## 仓库结构

| 路径 | 说明 |
| --- | --- |
| `patches/0001-home-access-control.patch` | 唯一的改动，针对上游源码 |
| `patch-revision.txt` | 补丁版本号；改动补丁后 +1，会让 Release 标签变成 `vX.Y.Z-mod.N+1` |
| `.github/workflows/build-mod.yml` | 解析上游版本 → 校验并应用补丁 → 编译 → 发布 APK |
| `.github/workflows/keygen.yml`（在私有密钥仓库） | 一次性生成签名密钥 |

## 自动同步上游

工作流每天北京时间 02:00 跑一次，也可以手动在 Actions 页面点 `Run workflow`（可指定任意上游 tag 或分支）：

1. 取上游最新 `vX.Y.Z` 标签；
2. 若 `vX.Y.Z-mod.N` 这个 Release 已存在就跳过（上游没更新时不会白跑一次完整编译）；
3. 否则拉取上游源码 → `git apply --3way` 打补丁 → 把 `versionName`/`versionCode` 按 tag 对齐（上游 tag 里记录的版本号比 tag 本身低一位，官方 release 也是在编译前改写版本号）→ 编译 `app:assembleMetaRelease` → 用固定密钥签名 → 发布 Release。

如果上游改到了补丁涉及的代码导致打不上，工作流不会硬来：`Patch applies` 任务会失败并自动开一个标题以「补丁冲突」开头的 issue。修复方式：

1. 按新上游代码调整 `patches/0001-home-access-control.patch`；
2. 把 `patch-revision.txt` 加 1；
3. 提交后工作流会自动重新构建并发布，关闭 issue。

## 签名密钥

密钥（JKS）保存为私有仓库 `Kano-u/ClashMetaForAndroid-Mod-Keys` 的 Actions 密钥材料和仓库文件，公开仓库里只保存 4 个 Secrets：`KEYSTORE_B64`、`KEYSTORE_PASSWORD`、`KEY_ALIAS`、`KEY_PASSWORD`。

**不要**重新生成密钥：换密钥等于换签名，之后新构建又必须卸载重装。想换的话记得旧密钥和新密钥的 APK 无法互相覆盖。


# 通枢工业通信套件（NetworkKits）

本仓库是**汇总/父仓库**，通过 git submodule 把三个独立子仓库组合在一起，并负责统一版本打 tag 与 CI 自动发布。

## 子仓库结构

| 目录 | 仓库 | 说明 |
|---|---|---|
| `NetworkComponentWeb/` | https://github.com/bigbigbird897/NetworkComponentWeb | Vue 控制台前端（静态站点） |
| `NetworkComponent/` | https://github.com/bigbigbird897/NetworkComponent | ASP.NET Core 后端接口（跨平台） |
| `NetworkComponentWPF/` | https://github.com/bigbigbird897/NetworkComponentWPF | Windows 桌面壳（WebView2 + 托盘 + 拉起后端） |

子仓库之间的相对路径（WPF csproj 里 `..\..\` 回到本仓库根，再进入两个子目录）因此保持不变。

## 首次克隆（带子模块）

```bash
git clone --recurse-submodules git@github.com:bigbigbird897/NetworkKits.git
```

如果已经克隆了父仓库但没带子模块：

```bash
git submodule update --init --recursive
```

## 日常开发与发版

子模块的提交要在各自目录内进行并 push，再回到父仓库更新指针：

```bash
# 1. 在子仓库内提交、推送（以 Web 为例）
cd NetworkComponentWeb
git add -A && git commit -m "..."
git push

# 2. 回到父仓库，更新子模块指针
cd ..
git add NetworkComponentWeb
git commit -m "chore: bump NetworkComponentWeb"
```

### 打 tag 触发自动发布

在父仓库打一个 `v*` 形式的 tag 并推送，即触发 GitHub Actions，按**该 tag 所锁定的三个子模块提交**自动打包：

```bash
git tag -a v1.0.0 -m "release v1.0.0"
git push origin v1.0.0
```

> 因为子模块指针记录在父仓库的这次提交上，所以打 tag 前请先把子模块需要的提交都 push、并在父仓库里 `git add <submodule>` 更新指针后再打 tag。

CI 产物会作为该 tag 的 GitHub Release 资产发布：
- `web-dist.zip`：前端静态文件（`npm run build` 产物）
- `backend-linux-x64.tar.gz`：后端 Linux 自包含可执行
- `backend-win-x64.zip`：后端 Windows 自包含可执行
- `NetworkComponentWPF-win-x64.zip`：WPF 桌面壳（含 WebView2 嵌入的控制台与后端）

工作流见 `.github/workflows/release.yml`。

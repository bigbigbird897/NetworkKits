
# 工业通信套件（NetworkKits）

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

CI 不直接用父仓库里的 submodule 指针，而是读根目录 **`repos.json`**，按其中指定的子仓库 tag 检出对应代码打包：

```json
{
  "NetworkComponentWeb": "v1.0.0",
  "NetworkComponent": "v1.0.0",
  "NetworkComponentWPF": "v1.0.0"
}
```

发版步骤：

```bash
# 1) 给要发版的子仓库打 tag 并推送（版本号自取）
cd NetworkComponentWeb
git tag -a v1.1.0 -m v1.1.0 && git push origin v1.1.0
cd ..

# 2) 改父仓库 repos.json，把对应项指到上面的 tag
# 3) 提交父仓库并打父 tag（v*），推送即触发打包
git add repos.json
git commit -m "release: web v1.1.0"
git tag -a v1.1.0 -m v1.1.0
git push origin main --tags
```

> 这样三个子仓库可以独立演进、各自打 tag，发版时只需在 `repos.json` 指定每个用哪个 tag，不必同步升级。

CI 产物会作为该 tag 的 GitHub Release 资产发布：
- `web-dist.zip`：前端静态文件（`npm run build` 产物）
- `backend-linux-x64.tar.gz`：后端 Linux 自包含可执行
- `backend-win-x64.zip`：后端 Windows 自包含可执行
- `NetworkComponentWPF-win-x64.zip`：WPF 桌面壳（含 WebView2 嵌入的控制台与后端）

工作流见 `.github/workflows/release.yml`。

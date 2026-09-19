# Plane 离线部署包 · GitHub Actions 云端打包（无需本地 Docker）

> 思路：本机（Windows，无 Docker、连不上 Docker Hub）不方便拉镜像。
> 改由 **GitHub 云端 Actions** 完成 `docker pull → docker save → gzip → 分片`，
> 产物发布为 GitHub Release 资产，浏览器直接下载回本机，再拷入内网服务器部署。

---

## 一、一次性准备（约 5 分钟，全网页操作）

1. 登录 [github.com](https://github.com)，右上角 `+` → **New repository**
2. 仓库名：`plane-offline-builder`，选择 **Public**，勾选 Add a README（可跳过），点击 **Create repository**
3. 进入仓库 → **Add file → Create new file**
4. 文件路径输入：`.github/workflows/build-offline.yml`
5. 内容：把本目录下 `build-offline.yml` 的内容**全文粘贴**进去
6. 页面下方 **Commit new file** 提交

## 二、触发打包（每次升级/重打都执行）

1. 进入仓库 **Actions** 页签 → 左侧选择 **Build Plane Offline Package**
2. 点右侧 **Run workflow** → 输入版本号（默认 `v1.4.2`）→ **Run workflow**
3. 等待几分钟，任务绿色勾号即完成（云端自动完成拉镜像/打包/发布）

## 三、下载产物

1. 进入仓库 **Releases** 页签，找到 `plane-offline-v1.4.2`（或对应版本）
2. 下载全部 `plane-offline-images.tar.gz.part00 / part01 / ...` 以及 `SHA256SUMS`
3. 放到同一个本地文件夹，合并并校验：

```bash
cat plane-offline-images.tar.gz.part* > plane-offline-images.tar.gz
sha256sum -c SHA256SUMS      # 输出 OK 即通过
```

> Windows 下用 Git Bash 执行上述命令。

## 四、内网部署

把 `plane-offline-images.tar.gz` 与以下文件放同一目录（如 `/opt/plane`）：

- `deploy.sh`
- `docker-compose.yml`
- `plane.env.example`

```bash
cd /opt/plane
sudo bash deploy.sh
```

看到 `Plane 部署完成 ✅` 即成功，浏览器访问 `http://<内网IP>:5005`。
详细步骤见 `docs/部署SOP.md`。

---

## 说明

- **为什么要分片**：Plane 离线包约 2-3GB，GitHub Release 单文件上限 2GB，按 900MB 分片规避。
- **公开仓库**：Actions 运行时长免费无限；产物存储不限额；仓库内只有打包脚本，无敏感信息。
- **minio 固定 tag**：云端将 `minio/minio:latest` 重打标为 `minio/minio:plane-frozen`，
  与 `docker-compose.yml` 中引用的 tag 一致，保证离线可复现。
- **升级版本**：改一次版本号 → 重新 Run workflow → 下载新资产 → 内网替换 `plane-offline-images.tar.gz` 后 `bash deploy.sh restart`。

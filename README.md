# sing-box-reF1nd-build

为第三方 fork [`reF1nd/sing-box`](https://github.com/reF1nd/sing-box) 提供 GitHub Actions 自动构建发布通道。

## 工作内容

- 每天 04:00 UTC 通过 `gh api` 拉取上游 tags 列表，过滤出以 `-reF1nd` 结尾的最新 tag。
- 若本仓库 Releases 中尚不存在同名 tag，按上游 `build.yml` 的流程编译 `linux-aarch64` 三个变体并打包为 tar.gz：
  - `sing-box-{version}-linux-arm64.tar.gz`（purego 变体，附带 `libcronet.so`）
  - `sing-box-{version}-linux-arm64-glibc.tar.gz`
  - `sing-box-{version}-linux-arm64-musl.tar.gz`
- 把产物上传到本仓库的 GitHub Releases，tag 名沿用上游（如 `v1.12.0-reF1nd`）。

构建启用了 `with_naive_outbound`，对应需要 cronet-go 的 chromium 工具链；首次跑全量约 25 分钟，后续命中缓存约 10 分钟。

## 触发方式

- **定时**：`0 4 * * *`，由 `.github/workflows/build.yml` 中的 `schedule` 自动触发。
- **手动**：在 Actions 页面选择 `Build` → `Run workflow`，可选填写：
  - `tag`：指定要构建的上游 tag；留空时与定时模式一样自动取最新。
  - `force`：勾选后即使本仓库已存在同名 Release 也会重建并覆盖产物。

## 修改构建矩阵

`.github/workflows/build.yml` 中的 `strategy.matrix.include` 列表里每行对应一个变体，注释或删除即可裁剪：

```yaml
- { os: linux, arch: arm64, variant: purego, naive: true }
- { os: linux, arch: arm64, variant: glibc,  naive: true }
- { os: linux, arch: arm64, variant: musl,   naive: true }
```

未来扩展其它架构（如 `amd64`、`loong64`）时按相同结构追加新行即可，构建步骤无需调整。

## 与上游的关系

- 本仓库不包含 sing-box 源码，构建时 `actions/checkout` 直接拉取 `reF1nd/sing-box` 在指定 tag 的快照。
- ldflags 注入路径写死为 `github.com/sagernet/sing-box/constant.Version`，依赖上游 fork 保持原模块路径。如未来 reF1nd 改动模块路径，需要同步修改工作流。

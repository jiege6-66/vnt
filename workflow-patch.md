# GitHub Actions 工作流修改补丁

## 需要修改的文件：`.github/workflows/rust.yml`

### 修改 1：添加 Windows ARM64 构建目标

在第 75 行 `FEATURES: ring-cipher,wss` 后添加以下内容：

```yaml
          - TARGET: aarch64-pc-windows-msvc # Windows ARM64 support
            OS: windows-latest
            FEATURES: ring-cipher,wss
```

完整的构建矩阵应该是：
```yaml
          - TARGET: x86_64-pc-windows-msvc # tested on a windows machine
            OS: windows-latest
            FEATURES: ring-cipher,wss
          - TARGET: aarch64-pc-windows-msvc # Windows ARM64 support
            OS: windows-latest
            FEATURES: ring-cipher,wss
          - TARGET: mipsel-unknown-linux-musl # openwrt
```

### 修改 2：添加 ARM64 编译配置

在第 206 行 `rustflags = ["-C", "target-feature=+crt-static","-C", "strip=symbols"]` 后添加：

```yaml
          [target.aarch64-pc-windows-msvc]
          rustflags = ["-C", "target-feature=+crt-static","-C", "strip=symbols"]      
```

### 修改 3 & 4：更新 Rust 版本

找到两处 `rustup install 1.80` 和 `rustup default 1.80`，将它们都改为：

```yaml
              rustup install 1.82
              rustup default 1.82
```

这两处分别在：
- 第 153-154 行
- 第 167-168 行

## 应用方法

1. 打开 `.github/workflows/rust.yml` 文件
2. 按照上述说明进行修改
3. 保存文件
4. 提交并推送更改

## 验证

修改完成后，创建一个新的 release tag 来触发构建：

```bash
git add .github/workflows/rust.yml
git commit -m "Add Windows ARM64 support to GitHub Actions workflow"
git push origin main
git tag v1.2.17-arm64-final
git push origin v1.2.17-arm64-final
```

构建成功后，您将在 Releases 页面看到 Windows ARM64 的二进制文件。

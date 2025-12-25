# qishui-decrypt

**汽水音乐（字节跳动系列 CENC）纯前端解密方案**

这是一个基于 Web Crypto API 的纯 HTML/JavaScript 工具，用于在浏览器端解密受 CENC (Common Encryption) 保护的 MP4 音视频文件。无需安装 FFmpeg，无需后端服务器，所有处理均在本地完成。

## ✨ 特性

- **纯前端运行**：基于 `index.html` 单文件，无服务器依赖，保护隐私。
- **无需 FFmpeg**：通过直接操作二进制数据流（ArrayBuffer）进行解密和容器重组。
- **智能格式提取**：
  - **FLAC**：检测到 FLAC 编码时，自动提取并封装为标准 `.flac` 文件。
  - **AAC/其他**：保持 `.mp4` / `.m4a` 容器格式，确保最大兼容性。
- **多种输入源**：支持在线 URL 下载（需 CORS 支持）或本地文件上传。
- **密钥支持**：支持输入 `spade_a` 字符串自动计算密钥，或直接输入 Hex Key。

## 🚀 使用方法

### 1. 运行环境要求 (重要)

由于本项目使用了浏览器原生的 **Web Crypto API**，该 API 出于安全考虑，通常要求在 **安全上下文 (Secure Context)** 中运行。

这意味着您不能直接双击 `index.html` 打开（`file://` 协议在某些浏览器中会限制加密 API）。您需要通过以下方式之一运行：

*   **本地服务器 (localhost/127.0.0.1)**：
    *   使用 VS Code 的 "Live Server" 插件。
    *   或者使用 Python: `python -m http.server 8000`
    *   或者使用 Node.js: `npx http-server`
*   **HTTPS 域名**：如果您将文件部署到网络，必须使用 HTTPS 协议。

### 2. 操作步骤

1.  打开网页。
2.  **输入密钥**：
    *   在 "输入密钥" 框中粘贴 `spade_a` 字符串（通常在 API 响应中获取）。
    *   工具会自动计算出 Hex Key。
    *   或者，如果您已有 32位 16进制密钥，可直接粘贴。
3.  **选择文件**：
    *   **在线 URL**：输入音频文件的下载地址（注意：目标服务器必须允许跨域 CORS，或者您需要安装跨域浏览器插件）。
    *   **本地文件**：直接上传已下载的加密 `.m4a` / `.mp4` 文件。
4.  点击 **"开始解密并下载"**。
5.  解密完成后，浏览器会自动开始下载处理好的文件。

## 🛠️ 技术原理

1.  **MP4 解析**：手动解析 MP4 Atom (Box) 结构，提取 `moov` (元数据) 和 `mdat` (加密媒体数据)。
2.  **密钥派生**：实现了 `spade_a` 混淆算法的逆向逻辑，从字符串中还原 AES Key。
3.  **解密**：解析 `senc` (Sample Encryption) 箱子获取 IV (初始化向量)，使用 `window.crypto.subtle.decrypt` (AES-CTR 模式) 对每个样本进行解密。
4.  **容器重组**：
    *   对于 FLAC：从 `stsd` -> `dfLa` 箱子中提取 StreamInfo，结合解密后的帧数据，拼装成标准的 FLAC 文件头。
    *   对于 AAC：移除加密标识 (`enca` -> `mp4a`)，重建 `stco` (Chunk Offsets)，生成干净的 MP4 容器。

## 🤝 致谢

本项目的核心算法（`spade_a` 解密逻辑）来源于开源社区的贡献：

*   **Algorithm Credit**: [jixunmoe](https://github.com/jixunmoe)
*   **Source Discussion**: [export_music Issue #1](https://github.com/592767809/export_music/issues/1#issue-1816086855)

感谢前辈们的逆向分析与分享。

## ⚠️ 免责声明 (Disclaimer)

1.  **仅供学习交流**：本项目仅用于学习浏览器 Web Crypto API、二进制文件解析及加密技术原理。
2.  **版权保护**：请勿使用本工具传播盗版音乐或用于商业用途。请支持正版音乐，尊重创作者和版权方的合法权益。
3.  **自行承担风险**：使用本工具产生的任何法律后果由使用者自行承担，作者不对任何滥用行为负责。
4.  本项目与汽水音乐及字节跳动公司无关。

## 📄 License

[MIT License](LICENSE)

---
marp: true
theme: default
paginate: true
backgroundColor: #1E2761
color: #FFFFFF
style: |
  section {
    font-family: 'Noto Sans CJK TC', 'Arial', sans-serif;
    padding: 40px;
  }
  h1 {
    color: #CADCFC;
    text-align: center;
  }
  h2 {
    color: #CADCFC;
    border-bottom: 2px solid #CADCFC;
  }
  li {
    font-size: 28px;
    margin-bottom: 10px;
  }
  footer {
    color: #CADCFC;
  }
  .hands-on {
    background-color: #F96167;
    color: white;
    text-align: center;
  }
  .hands-on h2 {
    color: white;
    border-bottom-color: white;
  }
  code {
    background: #212121;
    color: #F5F5F5;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: 'Courier New', Courier, monospace;
  }
---

# 《Snap 軟體打包入門》工作坊

COSCUP 2026 x UbuCon Asia  
林博仁  
https://brlin.gitlab.io/snap-packaging-101/

---

## 內容大綱

- Snap 是什麼？
- Snap 軟體包解析
- 安裝 Snapcraft
- Build Providers (LXD/Multipass)
- 實作環節：GNU Hello
- Snapcraft 專案配置 (snapcraft.yaml)
- 打包生命週期 (PULL -> PACK)
- 部署與發布至 Snap Store

---

## Snap 是什麼？

- **基於作業系統虛擬化技術**
- 透過 Ubuntu Core 提供跨平台相容性  
  <small>※需支援 `systemd` 與 `snapd`</small>

---

## Snap 軟體包解析

- **高度壓縮的 SquashFS 檔案系統**
- 掛載至 `/snap/軟體包名稱/current`
- 使用 **Linux Namespace** 技術實現隔離
- 在最小化的 Ubuntu Core 執行環境執行

---

## 打包對象的選擇

- **推薦打包**：
    - 命令列界面 (CLI) 軟體
    - 網路服務
- **斟酌**：
    - 圖形界面軟體 (GUI) 軟體
- **應避免**：
    - 需要存取非標準檔案系統路徑或太底層的系統資源

---

## 安裝 Snapcraft

- **Linux**: `snap install --classic snapcraft`
- **macOS**: 使用 Homebrew (`brew install`)
- **Windows**: 使用 WSL 2 進行操作

---

## Snapcraft Build Providers

- **LXD**: 基於 LXC 容器的打包環境 (Linux 預設)
- **Multipass**: 基於虛擬機的打包環境
- **macOS/Windows**: 僅支援 Multipass
- Multipass build provider 可透過環境變數客製化虛擬機的 CPU 核心樹與主記憶體大小：
  - `SNAPCRAFT_BUILD_ENVIRONMENT_CPU`
  - `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY`

---

## 實作環節：GNU Hello

- 使用 **GNU Hello** 演示程式進行實作
- 下載 GNU Hello 原始碼封存檔 (`.tar.gz`)  
  https://ftp.gnu.org/gnu/hello/
- `mkdir hello-snap`
- `snapcraft init`

---

## Snapcraft 頂層設定


```yaml
name: hello-coscup2025
base: core24
version: 2.12.3
summary: Demonstration snap for COSCUP 2025
description: |
  This is a demonstration snap for COSCUP 2025. It is used to show how to create a snap package.

grade: stable
confinement: strict
```

---

## Snap 打包生命週期

1. **PULL**: 下載並解壓縮零件內容
2. **BUILD**: 編譯軟體並安裝到 build 目錄
3. **STAGE**: 將執行所需檔案移至 stage 目錄
4. **PRIME**: 排除不必要的檔案與加入詮釋資料
5. **PACK**: 封裝成 SquashFS 檔案 (`.snap`)

---

## PULL 步驟

```bash
snapcraft pull --verbose --shell-after
```

## BUILD 步驟

```bash
snapcraft build --verbose --shell-after
```

## STAGE 步驟

```bash
snapcraft stage --verbose --shell-after
```

## PRIME 步驟

```bash
snapcraft prime --verbose --shell-after
```

## PACK 步驟

```bash
snapcraft pack
```

## 安裝打包好的 snap

```bash
snap install --dangerous --devmode ./hello_*snap
```

## 設定 Snap 應用

```yaml
apps:
  hello-coscup2025:
    command: usr/local/bin/hello
    plugs: []

  traditional:
    command: usr/local/bin/hello --traditional
    plugs: []
```

---

## 設定 Layout

```yaml
layout:
  /usr/local/share/locale:
    symlink: $SNAP/usr/local/share/locale
```

---

# 謝謝聆聽
## Q & A

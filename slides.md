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

## 應用程式定義與別名

- **apps 映射**：定義軟體提供的可執行命令
- `command`: 指定軟體在系統中的實際路徑
- **Alias**: 設定慣用的命令名稱 (如 `hello`)
- 允許在不同的頻道 (`edge`, `beta`, `stable`) 之間切換

---

## 處理路徑與佈局 (Layouts)

- **問題**：軟體路徑寫死在程式中導致找不到檔案
- **解決方案**：使用 **Layouts** 功能
- 透過 Symlink 將資料掛載到預期路徑
- **範例**：將 `$SNAP/usr/local/share/locale` 掛載到 `/usr/local/share/locale`

---

<section class="hands-on">

# 🛠️ Hands-on Lab 4
## Publishing to the Store

1. **Login**: `snapcraft login`
2. **Upload**: `snapcraft upload <your_snap_file>`
3. **Register**: Log in to Snap Store dashboard
4. **Promote**: `snapcraft promote --from-channel edge --to-channel beta <your_snap_file>`

</section>

---

## 上傳至 Snap Store

1. 登入 Ubuntu One 帳號 (`snapcraft login`)
2. 上傳軟體包 (`snapcraft upload`)
3. 在商店後台進行 **Register** 與 **Promote**
4. 從 `edge` 頻道逐步提升至 `stable` 頻道

---

## 總結

- Snap 提供了強大的隔離與跨平台打包能力
- 理解 **Build Lifecycle** 是打包成功的關鍵
- 利用 **Layouts** 解決路徑依賴問題
- **祝大家打包順利！**

---

# 謝謝聆聽
## Q & A

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

- **LXD**: 基於 LXC 容器的虛擬化 (Linux 預設)
- **Multipass**: 基於 KVM 虛擬機的虛擬化
- **macOS/Windows**: 僅支援 Multipass
- Multipass build provider 可透過環境變數客製化 CPU 與 Memory：
  - `SNAPCRAFT_BUILD_ENVIRONMENT_CPU`
  - `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY`

---

## 實作環節：GNU Hello

- 使用 **GNU Hello** 演示程式進行實作
- 下載 GNU Hello 原始碼封存檔 (`.tar.gz`)
- 建立 `hello-snap` 工作目錄
- 執行 `snapcraft init` 初始化專案

---

<section class="hands-on">

# 🛠️ Hands-on Lab 1
## Setup & Initialization

1. `git clone https://gitlab.com/brlin/snap-packaging-101.git`
2. `cd snap-packaging-101`
3. `git submodule init && git submodule update`
4. `mkdir hello-snap && cd hello-snap`
5. `snapcraft init`

</section>

---

## Snapcraft 頂層設定

- `name`: 軟體包唯一識別名稱
- `base`: 使用的 Base Snap (例如 `core24`)
- `version`: 軟體版本號 (建議用字串)
- `summary`: 簡短的軟體描述 (限制 78 字元)
- `confinement`: 權限限制模式 (`strict`, `devmode`, `classic`)

---

## Snap 打包生命週期

1. **PULL**: 下載並解壓縮零件內容
2. **BUILD**: 編譯軟體並安裝到 build 目錄
3. **STAGE**: 將執行所需檔案移至 stage 目錄
4. **PRIME**: 排除不必要的檔案與加入詮釋資料
5. **PACK**: 封裝成 SquashFS 檔案 (`.snap`)

---

<section class="hands-on">

# 🛠️ Hands-on Lab 2
## The Build Lifecycle

**Step 1: Pull & Build**
`snapcraft pull --verbose`
`snapcraft build --verbose --shell-after` (Run `exit` when prompted)
`snapcraft build --verbose`

**Step 2: Stage & Prime**
`snapcraft stage --verbose --shell-after` (Run `exit`)
`snapcraft prime --verbose --shell-after` (Run `exit`)

**Step 3: Pack**
`snapcraft pack`

</section>

---

## Build & Stage (建構與階段準備)

- **BUILD**: 執行 `configure`, `make`, `make install`
- **STAGE**: 準備執行環境，提供依賴資源
- 可使用 **Build Plugin** (如 `autotools`, `python`, `go`)
- 利用 `organize` 鍵搬移檔案到指定路徑

---

## Prime & Pack (精簡與封裝)

- **PRIME**: 排除不必要的檔案以減小檔案體積
- 排除方式：在 `parts.name.prime` 中使用 `"-"` 前符
- **PACK**: 將 Prime 目錄內容封裝成最終 Snap 檔
- 使用 `snapcraft pack` 執行

---

<section class="hands-on">

# 🛠️ Hands-on Lab 3

## Deployment & Testing

**Step 1: Run Application**
`sudo snap install --dangerous --devmode ./hello_...snap`
`hello` (Test the command)

**Step 2: Define Apps & Aliases**
*Edit `snapcraft.yaml` to add `apps` and `alias`*
`sudo snap install --dangerous ./hello_...snap`
`hello` (Test via alias)

**Step 3: Fix Layouts (Relocatable)**
*Edit `snapcraft.yaml` to add `layout` for locales*
`sudo snap install --dangerous ./hello_...snap`
`snap run hello` (Test language support)

</section>

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

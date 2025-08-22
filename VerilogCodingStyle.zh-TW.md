
# lowRISC Verilog 程式碼風格指南（中文版）

## 基本原則

### 總覽

Verilog 是 lowRISC Comportable IP 設計的主要邏輯描述語言。由於 Verilog/SystemVerilog 語言彈性大，不同工程師可能寫出風格迥異的程式碼，這會導致協作困難與審查效率下降。

此風格指南的目標是：

- 統一硬體設計的程式風格
- 推廣業界最佳實踐
- 提升程式碼的可重用性與共享性

此指南涵蓋 Verilog-2001 與 SystemVerilog-2017，包含可綜合（synthesizable）及測試平台（testbench）程式碼。

### 術語定義

- **must（必須）**：強制規定，違反即為錯誤。
- **recommended（建議）**：最佳實踐，非強制，但偏離時需充分理解風險。
- **may（可以）**：可選項，取決於具體情境。
- **can（能夠）**：從語法/工具角度是可行的。

### 預設格式：接近 C/C++

除非特別指明，請遵循 [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) 的原則，包括：

- 2 空白縮排（不要使用 Tab）
- 每行最大 100 字元（建議），軟限制
- 大括號風格與 C++ 相同（例如，函數定義的左大括號與函數名稱同行）

---


## 命名規範

- 所有識別符號必須使用小寫字母與底線（`snake_case`）。
- 模組名稱、介面名稱應具有描述性，例如 `fifo_sync` 或 `arbiter_fixed`。
- 輸入輸出埠應明確反映其功能，例如：
  - `clk_i`：輸入時鐘
  - `rst_ni`：低有效非同步重設（_ni 表示 negative logic input）
  - `req_i`、`gnt_o`：請求與授權訊號
- 類型定義加 `_t` 後綴，例如 `state_e`、`fsm_state_t`。

## 模組與介面

- 每個模組必須位於一個單獨的 `.sv` 檔案中，且檔名需與模組名稱一致。
- 使用 SystemVerilog 介面（interface）封裝複雜的匯流排或握手訊號。
- 模組埠順序建議如下排列（但可依邏輯群組微調）：
  1. 時鐘與重設（`clk_i`, `rst_ni`）
  2. 控制信號（如使能、啟動）
  3. 資料通路（`data_i`, `data_o`）
  4. 狀態回報或握手信號（`valid`, `ready`, `gnt` 等）

## 註解風格

- 每個模組需附上簡要註解描述其功能。
- 使用 `//` 單行註解，保持簡潔清楚。
- 用 `/** ... */` 多行註解於必要時補充背景說明或介面協定。

## 時序與時鐘結構

- 僅使用正緣觸發（`posedge clk_i`），除非有特殊理由。
- 所有重設必須為非同步輸入、同步釋放。
- 重設信號應命名為 `rst_ni`（低有效），並放置於邏輯區塊最前面。

### 建議的時序區塊格式：

```systemverilog
always_ff @(posedge clk_i or negedge rst_ni) begin
  if (!rst_ni) begin
    // reset logic
  end else begin
    // sequential logic
  end
end
```

## 多時鐘域與時鐘交叉（CDC）

- 時鐘交叉需使用同步器或 FIFO。
- 禁止使用直接跨時鐘的訊號傳遞。
- 對於握手信號，使用 `valid-ready` 或 `req-ack` 協定，並遵守雙邊時鐘穩定原則。
- 使用像 `prim_fifo_async`、`prim_flop_2sync` 等已驗證元件進行 CDC。

## 可綜合性設計原則

- 僅使用可綜合 SystemVerilog 結構。
- 避免使用 `#delay`、`initial`、動態陣列等不可綜合語法。
- 對於多位元訊號，使用參數化定義訊號寬度（如 `parameter int DW = 32;`）

---

若需進一步細節，建議參考原始 lowRISC style guide 或補充如 lint 工具（Verible、VCS）配置與建議。

## 模組階層與封裝

- 每個模組應專注於單一功能，避免過於龐大或過於複雜。
- 建議使用參數（`parameter`）而非巨集（`define`）進行可配置設計。
- 子模組間的介面應盡量使用 interface 封裝傳遞訊號。

## 巨集（Macro）與參數使用

- **must**：避免過度使用 `define` 巨集，會降低可讀性與除錯能力。
- **recommended**：使用 `parameter` / `localparam` 進行寬度、深度等設計參數設定。
- 使用 `localparam` 封裝內部僅供模組使用的常數。

## 專案結構與檔案命名

- 每個 `.sv` 檔案應只定義一個模組。
- 檔名與模組名稱一致，例如 `fifo_sync.sv` 定義 `fifo_sync` 模組。
- 相關測試檔案應置於 `dv/` 或 `tb/` 子目錄中。

## 測試平台（Testbench）風格

- Testbench 必須使用 SystemVerilog。
- 使用 UVM（Universal Verification Methodology）或輕量級手寫 testbench。
- 優先使用斷言（`assert`）與覆蓋率（`cover`）檢查功能。
- Testbench 使用 `logic` 而非 `wire`/`reg`。

## 建構與驗證工具支援

- **Lint 工具**：推薦使用 [Verible](https://github.com/chipsalliance/verible) 進行語法檢查與自動格式化。
- **Simulation**：支援 VCS、Xcelium、Verilator。
- **Format 工具**：所有程式碼應通過 `verible-verilog-format` 檢查。

## 版本控制與協作

- 所有程式碼應通過 review，並遵守 Git 分支與 commit 訊息格式。
- 推薦的 commit 格式：
  ```
  [module] 簡要說明變更內容
  ```

  範例：
  ```
  [fifo_sync] 修正空狀態判斷邏輯錯誤
  ```

---

# 附錄：推薦工具與參考資源

- **Lint / 格式化工具**
  - [Verible](https://github.com/chipsalliance/verible)
  - `verible-verilog-lint`, `verible-verilog-format`
- **Simulation**
  - [Verilator](https://www.veripool.org/verilator/)
  - Synopsys VCS / Cadence Xcelium
- **風格參考**
  - [Google SystemVerilog Style Guide](https://google.github.io/styleguide/verilog/)

---

感謝閱讀本風格指南。如有建議或想提交貢獻，請透過 lowRISC GitHub 專案提出 Pull Request 或 Issue。

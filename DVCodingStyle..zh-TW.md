# lowRISC SystemVerilog 設計驗證風格指南（正體中文版）

本文件為 [lowRISC Verilog 程式碼風格指南](https://github.com/lowRISC/style-guides/blob/master/VerilogCodingStyle.md) 的補充，專注於依循 [UVM 方法論](https://www.accellera.org/images//downloads/standards/uvm/uvm_users_guide_1.2.pdf) 撰寫 SystemVerilog 設計驗證（Design Verification, DV）相關程式碼。

---

## 目錄

- [簡介](#簡介)
- [命名與風格](#命名與風格)
- [目錄結構與套件](#目錄結構與套件)
- [UVM 準則](#uvm-準則)
- [SystemVerilog 語言特性](#systemverilog-語言特性)
- [SystemVerilog Assertion（SVA）風格](#systemverilog-assertion（sva）風格)

---

## 簡介

本指南定義了 Comportable 設計驗證程式碼的風格。其目標是，在使用 UVM 方法論建構測試平台時，透過一致的撰寫風格，讓驗證程式碼具備統一性、可重用性與可移植性。

本文件假設你已具備 UVM 基礎知識，程式碼片段將精簡呈現重點。

---

## 命名與風格

請參閱 [lowRISC 命名規範](https://github.com/lowRISC/style-guides/blob/master/VerilogCodingStyle.md#naming)，以下為針對驗證環境常見元件的命名建議：

| 測試平台元件                | 命名風格                                      |
| -------------------------- | --------------------------------------------- |
| Virtual Interface          | `virtual <if_name>_if <if_name>_vif;`         |
| 驗證環境（Environment）     | `<dut>_env env;`                              |
| 環境層設定物件              | `rand <dut>_env_cfg cfg;`                     |
| 環境層 coverage 物件        | `<dut>_env_cov cov;`                          |
| Agent                      | `<dut>_agent m_<dut>_agent;`                  |
| Agent 層設定物件           | `rand <dut>_agent_cfg m_<dut>_agent_cfg;`     |
| Agent coverage 物件        | `<dut>_agent_cov cov;`                        |
| Driver                     | `<dut>_driver driver;`                        |
| Monitor                    | `<dut>_monitor monitor;`                      |
| Scoreboard                 | `<dut>_scoreboard scoreboard;`                |
| Virtual Sequencer          | `<dut>_virtual_sequencer virtual_sequencer;`  |
| Sequencer                  | `<dut>_sequencer <dut>_sequencer;`            |

**命名準則補充：**

1. 測試平台頂層模組檔案命名為 `tb.sv`，模組名為 `tb`，DUT 實例名為 `dut`。
2. `type_id::create()` 中 `name` 參數必須與變數名稱一致，以利除錯。
3. 套件定義的類型應加上唯一前綴，避免名稱衝突。
4. 所有套件變數皆需明確指定型別。
5. 類別物件實例名稱建議加上 `m_` 前綴。
6. 多個實例時可加上具意義的名稱，例如 `m_my_cat`、`m_my_dog`。
7. 陣列命名建議使用複數，如 `m_my_animals[NUM_ANIMALS]`。

---

（後續將繼續擴充到整份翻譯內容，包含目錄結構、UVM 準則、SystemVerilog 特性、Assertions 等）


## 目錄結構與套件

每個類別應使用獨立檔案，且檔名與類別名稱相符。類別檔案需透過 `\`include` 引入套件（package）檔案中。

### UVM Agent 目錄結構

- 每個 Agent 的驗證碼應集中於 `<block>_agent/` 目錄。
- 所有 Agent 可集中於共用區域，以提升可重用性。

```txt
<block>_agent/
  <block>_if.sv
  <block>_item.sv
  <block>_agent.sv
  <block>_agent_pkg.sv
  <block>_agent_cfg.sv
  <block>_agent_cov.sv
  <block>_driver.sv
  <block>_host_driver.sv
  <block>_device_driver.sv
  <block>_monitor.sv
  <block>_sequencer.sv
  seq_lib/
    <block>_base_seq.sv
```

### UVM Environment 目錄結構

- 所有 UVM 環境、測試、頂層 testbench 的驗證碼應位於 `<dut>/dv/` 下的子目錄：

```txt
<dut>/dv/
  env/
    <dut>_env.sv
    <dut>_env_pkg.sv
    <dut>_env_cfg.sv
    <dut>_env_cov.sv
    <dut>_scoreboard.sv
    <dut>_virtual_sequencer.sv
    seq_lib/
      <dut>_base_vseq.sv
      <dut>_sanity_vseq.sv
  tb/
    tb.sv
  tests/
    <dut>_base_test.sv
    <dut>_test_pkg.sv
```

- `<block>_agent_pkg.sv`、`<dut>_env_pkg.sv`、`<dut>_test_pkg.sv`、`tb.sv` 中須包含：

```systemverilog
`include "dv_macros.svh"
`include "uvm_macros.svh"
import uvm_pkg::*;
```

---

（接下來會繼續翻譯 UVM 準則與所有內容直到結尾）

## UVM 準則

始終使用 `run_test()` 且不指定測試名稱，這樣可以透過 `+UVM_TESTNAME=...` 參數在命令列中覆寫測試類別，無需重新編譯，亦便於回歸測試（regression run）執行。

### 類別定義（Class Definitions）

1.  宣告變數後，使用 `uvm_component_utils` 或 `uvm_object_utils` 宏註冊至 factory。
2.  欲啟用自動欄位操作，使用 `uvm_field_*` 宏包住欄位定義，並指定旗標為 `UVM_DEFAULT`。
    - 範例：

      ```systemverilog
      `uvm_field_int(rx_delay, UVM_DEFAULT | UVM_NOCOMPARE | UVM_NOPRINT)
      ```

3.  註冊完 factory 後需定義建構子，使用 `uvm_object_new` 或 `uvm_component_new` 宏。
4.  若手動定義建構子，第一行需為 `super.new(...)`。
5.  建構子僅能接收 `name` 和（若為 component）`parent` 參數。
6.  所有物件應在 `build_phase()` 或序列的 `body()` 開頭實例化，方便 factory override。
7.  若繼承自其他類別，覆寫的每個階段方法中必須呼叫 `super.<phase>_phase()`。
8.  transaction 類別：
    - 繼承自 `uvm_sequence_item`
    - 僅包含 interface 資料，不含傳輸機制
    - 必須可合法隨機化

### `new()` 函式定義

- 若繼承自 `uvm_object`，則 `name` 參數預設為空字串 `""`
- 若繼承自 `uvm_component`，不得有預設值

👍 正確範例：

```systemverilog
function new (string name = "");
  super.new(name);
endfunction

function new(string name, uvm_component parent);
  super.new(name, parent);
endfunction
```

👎 錯誤範例：

```systemverilog
function new(string name = "my_seq");  // 錯：名稱預設不為 ""
  super.new(name);
endfunction
```

---

（接下來將持續翻譯 UVM Scoreboard、Agent、Driver、Macro 使用等規範）

### `uvm_scoreboard` 使用

使用 `uvm_tlm_analysis_fifo` 儲存需延遲處理的 transaction。其無界限大小、不阻塞寫入，適合在 Scoreboard 等待資料比對時使用。

---

### `uvm_agent` 使用

1. Interface agent 應只包含 sequencer、driver、monitor 與 config。
2. 每個 DUT interface 應配置一個 agent，並透過 `is_active` 控制是否啟動 sequencer/driver。
3. 若 DUT 有多種模式（如 host/device），需設計兩個 driver，並在 `build_phase` 做選擇。

---

### `uvm_driver` 使用

1. driver 僅連接一個 sequencer 與一個 interface，**不可含 analysis port**。
2. 不得在 driver 隨機化從 sequencer 接收到的 transaction。
3. 若 clock cycle 無有效 transaction，driver 控制之 bus 輸出應為 `X`。

---

### 宏（Macro）使用準則

**UVM 提供大量宏簡化流程，亦有 `dv_macros.svh` 補充自訂宏**

#### UVM 宏使用

- 所有 `\`uvm_` 開頭的宏皆不需結尾分號。
- `uvm_object_*` / `uvm_component_*`：註冊類別至 factory。
- `uvm_field_*`：用於 sequence item 欄位自動化。

> ⚠ 不得在 `uvm_component` 派生類中使用 `uvm_field_*`

##### 正確做法：

```systemverilog
start_item(item);
assert(item.randomize());
finish_item(item);
```

##### 錯誤做法（已棄用）：

```systemverilog
`uvm_do(item)
```

---

（下一階段將補上 dv_macros.svh 宏、Factory 使用、Configuration、Sequence/Item、Assertion、Logging、SVA 等章節）

### `dv_macros.svh` 宏使用建議

1. **UVM 簡化印出巨集**：`gfn`, `gtn`, `gn`, `gmv` 簡化 `get_name` 等調用
2. **downcast**：`downcast(ext, base)` 將基底類別轉型為擴充類型
3. **new 宏**：`uvm_object_new`, `uvm_component_new` 包裝 constructor
4. **建立物件**：`uvm_create_obj`, `uvm_create_comp` 等價於 `type_id::create(...)`
5. **檢查巨集**：`DV_CHECK_*`、`DV_CHECK_*_FATAL` 可快速比對並報錯
6. **隨機化**：使用 `DV_CHECK_RANDOMIZE_FATAL` 等宏包裝 randomize 呼叫
7. **結尾檢查列印**：`DV_EOT_PRINT_*` 用於 scoreboard 中 TLM FIFO 清空檢查
8. **隔離等待（Spinwait）**：`DV_SPINWAIT(...)` 執行隔離式等待與錯誤偵測

---

## Factory 使用

1. 所有類別需註冊至 factory，使用 `uvm_object_utils` 或 `uvm_component_utils`
2. 建立物件應透過 `type_id::create(...)`，勿直接 `new()` 除非是 covergroup / TLM
3. 若需建立陣列，使用 `$sformatf(...)` 為每個物件命名
4. 對於 component，第二參數為 `this` 指定父層
5. 物件命名與 handle 名稱一致可避免錯誤訊息混淆

```systemverilog
m_agent = agent_type::type_id::create("m_agent", this);
```

6. 使用 `gfn` 補足 hierarchy 名稱：

```systemverilog
m_txn = txn_type::type_id::create({`gfn, ".m_txn"});
```

7. Override 使用範例如下：

```systemverilog
factory.set_type_override_by_type(base::get_type(), impl::get_type());
factory.set_inst_override_by_type("env.agent.driver", base::get_type(), impl::get_type());
```

---

## Configuration 機制

1. 僅使用 `uvm_config_db`，**禁止使用** `uvm_resource_db` 或 `set/get_config_*`
2. 每個 agent/env 應有對應 config object，並包入其他 config
3. 僅傳遞完整 object，不建議傳遞基本型別（避免命名衝突）
4. 所有 `get()` 呼叫必須檢查成功與否：

```systemverilog
if (!uvm_config_db#(...)::get(...)) `uvm_fatal(...)
```

5. 不可在 `set()` 中使用 field wildcard；inst_name 可視情況使用 `*`

---

（下一批翻譯將涵蓋 Sequences, Objection, Logging, DPI, SystemVerilog 語法與 Assertion 章節）

---

## Sequences 與 Sequence Items

### Sequences 使用準則

1. 儘量撰寫通用（generic）sequence，避免 directed test。
2. 避免使用絕對時間延遲（如 `#10ns`），鼓勵以 clock 為單位的延遲。
3. 功能邏輯應放在 `body()`，其他初始化應放 `pre_start()` / `post_start()`。
4. 建立 sequence 後須先 `randomize()` 才能啟動。
5. 使用 virtual sequence 協調多 agent/sequencer。
6. virtual sequence 可啟動於 null sequencer 或 virtual sequencer。

### Sequence Item 使用

建議（非強制）為每個 item 撰寫簡易測試來驗證 constraint 是否有效。

---

## Objection 控制與測試結束協調

原則：當 stimulus 尚未結束、或 DUT 尚未回應時，**不得結束 phase**

1. 用 `phase.raise_objection()` / `drop_objection()` 控制測試存活時間
2. 監控元件應實作 watchdog：觀察 bus 活動情形，自動控制 objection
3. scoreboard 不應自行使用 objection，應依據 FIFO 狀態推斷測試結束

### 建議格式

```systemverilog
phase.raise_objection(this, $sformatf("%s raised", `gfn));
...
phase.drop_objection(this, $sformatf("%s dropped", `gfn));
```

4. 建議所有 testbench 設定 timeout：

```systemverilog
uvm_root::set_timeout(1ms, 1);
```

---

## Logging 與列印訊息規範

1. 限制印出至 console，避免大量非必要輸出
2. 關鍵資料寫入 log 檔；console 僅印錯誤或重大狀態
3. 善用 `uvm_info` 並分級：

| Verbosity | 用途                         |
| --------- | ---------------------------- |
| UVM_LOW   | component 初始化，reset 等   |
| UVM_MEDIUM| CSR 操作、重要狀態事件       |
| UVM_HIGH  | DUT/RefModel 交易狀況        |
| UVM_DEBUG | 詳細除錯訊息（CSR 排除等）   |

4. 嚴禁使用 `$display`，請使用 `uvm_info`, `uvm_error`, `uvm_fatal`
5. 禁用 `uvm_warning`，改用 `uvm_error` 或 `uvm_fatal`
6. 第一個參數應使用 `gfn` / `gtn`，避免硬寫 `get_name()`

---

## DPI / C 調用慣例

1. SV ⇄ C 之 DPI 函式命名建議：
   - SV export → `sv_dpi_*`
   - C import → `c_dpi_*`
2. 適用情境：呼叫 C/C++ ref model、emulation、高效函式庫
3. 不建議將 SV 可實作邏輯搬至 C
4. 跨語言呼叫應使用 semaphore 確保 thread safe
5. 儘量僅傳遞基本型別

---

（後續章節將補上 SV 語法細節、randomization、SVA 模組等）

---

## SystemVerilog 語言特性規範

### 函式宣告

- 所有 package 內函式、模組、interface，皆須明確使用 `automatic` 或 `static`。

### 隨機化（Randomization）

1. 所有 constraint 命名應加上 `_c` 作為後綴。
2. 避免在 constraint 中使用迴圈，可使用 `inside` 取代：

```systemverilog
constraint val_c {
  !(a inside {data});
}
```

3. 避免在 constraint 內進行複雜運算。
4. 優先使用 bitmask（如 `&`）而非 `%`。
5. constraint 複雜時可分拆並各自 randomize。
6. 隨機化 object array 時應逐一 randomize。

### Enum 命名

```systemverilog
typedef enum bit {
  UartInterruptFrameErr,
  UartInterruptRxBufferFull
} uart_interrupt_e;
```

- 類型使用 `snake_case`，值為 `UpperCamelCase`。

### Interface / Clocking Block / Modport

1. 不使用 `program` 區塊。
2. clock/reset 必須由 module 或 interface 產生。
3. 使用 clocking block 搭配 modport。
4. 建議 block level interface 可重用於整合/系統測試平台。

### 迴圈（Loop）建議

- 優先使用 `foreach` 迴圈，避免 `for`、`while` 產生錯誤。

### Assert 內不可含副作用

❌ 錯誤範例：

```systemverilog
assert(item.randomize());
```

✅ 正確寫法：使用 `DV_CHECK_RANDOMIZE_FATAL`

---

## SystemVerilog Assertion（SVA）使用準則

1. SVA 模組僅使用 input port，不可驅動訊號。
2. 使用 `ASSERT` 宏而非手寫 `assert property(...)`。
3. 使用 `bind` 綁定至 DUT module，而非 testbench。
4. 不可於 SVA 內使用絕對階層路徑。
5. 若 cross-hierarchy，將 SVA bind 至最接近共同祖先。
6. 所有 bind 語句應寫於獨立 bindfile 中，作為 top module 傳入。
7. bind 時綁定模組名稱而非特定實例名稱。

```systemverilog
bind aes_core sva_aes_checker u_sva(.clk(clk), ...);
```

---

# ✅ 完整翻譯結束

此文件為 lowRISC SystemVerilog 設計驗證風格指南之正體中文完整翻譯版，已涵蓋所有章節，包括命名規範、UVM 方法論、Macro、Objection 控制、DPI、Assertions 及 SVA 綁定技巧。


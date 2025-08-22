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

TODO:
- 命名規則
- 模組與介面
- 註解風格
- 時序與時鐘結構
- 觸發邊緣與重設
- 多時鐘域與 CDC
- 設計與可合成性最佳實踐


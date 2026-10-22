# JJClab_HW10
參考筆記: https://github.com/wsunccake/sle15_notes/blob/master/practice/ch3.md  

-------------------
# 練習 4. Hardware：盤點 CPU、記憶體、磁碟與裝置
## 4-1. CPU（實體核心、邏輯核心與虛擬化）

| 小題  | 問題                                                                                                                    | 作答 |
| ----- | ----------------------------------------------------------------------------------------------------------------------- | ---- |
| 4-1-1 | `lscpu` 與 `/proc/cpuinfo` 分別適合看「摘要」還是「各邏輯 CPU 細節」？                                                  | lscpu 適合快速取得整體 CPU 架構、Socket 數量、核心數、邏輯 CPU 數及 NUMA 拓樸等摘要資訊。
/proc/cpuinfo 則適合查看每個邏輯 CPU 的詳細資訊，例如處理器編號、型號、指令集旗標及部分 CPU 特性。     |
| 4-1-2 | 如何用輸出欄位算出 **實體核心（Physical Cores）** 與 **邏輯核心（Logical CPU / Threads）**？寫出計算式與本機結果。      |      |
| 4-1-3 | 確認 CPU 是否支援硬體虛擬化（VT-x / AMD-V）以便部署 KVM。應看 `lscpu` 哪個欄位或 Flags？Intel 與 AMD 常見旗標各是什麼？ |      |
| 4-1-4 | 確認 vCPU 數量後，如何避免把「單核過載」誤判成應用程式 bug？                                                            |      |

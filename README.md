# JJClab_HW10
參考筆記: https://github.com/wsunccake/sle15_notes/blob/master/practice/ch3.md  

-------------------
# 練習 4. Hardware：盤點 CPU、記憶體、磁碟與裝置
## 4-1. CPU（實體核心、邏輯核心與虛擬化）

| 小題  | 問題                                                                                                                    | 作答 |
| ----- | ----------------------------------------------------------------------------------------------------------------------- | ---- |
| 4-1-1 | `lscpu` 與 `/proc/cpuinfo` 分別適合看「摘要」還是「各邏輯 CPU 細節」？                                                  | lscpu 適合快速查看整體 CPU 架構、Socket 數量、Core 數量及虛擬化資訊；/proc/cpuinfo 則記錄每個邏輯 CPU 的個別資訊，適合查看處理器編號、型號與 CPU flags。    |
| 4-1-2 | 如何用輸出欄位算出 **實體核心（Physical Cores）** 與 **邏輯核心（Logical CPU / Threads）**？寫出計算式與本機結果。      |     本次 VMware 虛擬機顯示 Socket(s)=2、Core(s) per socket=1、Thread(s) per core=1，因此 Guest 可見的核心數為 2x1=2，邏輯 CPU 數為2x1x1=2。由於使用虛擬機，這些數值反映 Guest 的虛擬 CPU 配置，而非 Host 實際物理核心數。 |
| 4-1-3 | 確認 CPU 是否支援硬體虛擬化（VT-x / AMD-V）以便部署 KVM。應看 `lscpu` 哪個欄位或 Flags？Intel 與 AMD 常見旗標各是什麼？ |     可透過 lscpu 的 Virtualization 欄位，以及 /proc/cpuinfo 的 flags 來檢查 CPU 是否提供虛擬化功能。Intel 常見的旗標為 vmx，對應 VT-x；AMD 則為 svm，對應 AMD-V。若系統位於 VMware 虛擬機中，Guest 是否能看到相關旗標取決於 Hypervisor 的設定，因此還需要區分實體 Host 的虛擬化支援能力與 Guest 的可用能力。 |
| 4-1-4 | 確認 vCPU 數量後，如何避免把「單核過載」誤判成應用程式 bug？                                                            |    應先透過 lscpu 確認系統的 vCPU 數量，再使用 top 或 mpstat -P ALL 觀察各個邏輯 CPU 的使用率，而非只看整體平均值。若某個核心長時間滿載，應確認程式是否為單執行緒、是否受到 CPU 資源競爭或虛擬化排程影響，再判斷是否屬於應用程式異常，避免直接將高 CPU 使用率認定為程式錯誤。  |

虛擬機的檢查如下:  
<img width="868" height="93" alt="image" src="https://github.com/user-attachments/assets/ff6bbed4-8c5f-4795-a2c2-889b9e8f309a" />

## 4-2. RAM（應用程式回報記憶體不足）

| 小題  | 問題                                                                          | 作答 |
| ----- | ----------------------------------------------------------------------------- | ---- |
| 4-2-1 | 哪個指令能以人類可讀單位顯示記憶體使用量？寫出完整指令。                      |   使用 free -h 可以以人類可讀的單位顯示系統的記憶體使用情況，包括 RAM 的總容量、已使用量、剩餘量、快取及可用量，同時顯示 Swap 的使用情形。   |
| 4-2-3 | 為什麼 Linux 常看起來 free 很小？應優先看哪個欄位判斷「還能不能再吃記憶體」？ |    Linux 會將部分閒置記憶體用作 Buffer 或 Cache，以提升系統效能，因此 free 較低不一定代表記憶體不足。判斷系統是否還有足夠的記憶體執行新程式時，應優先觀察 available 欄位。  |
| 4-2-4 | 記錄本機 `free -h` 的 available 與 free，並解釋差異                           |   本機 VMware SLES 15 SP7 虛擬機具有約 3.2 GiB RAM。透過 free -h 可觀察到 free 為 1.7 GiB，而 available 為 2.1 GiB。兩者的差異在於 free 只表示完全未使用的實體記憶體，而 available 還考慮部分可回收的快取與其他記憶體，因此較適合用來評估系統能否執行新的應用程式。此外，本機 Swap 總容量為 2.0 GiB，目前使用量為 0 B，顯示本次觀測未出現明顯的 Swap 使用壓力。   |

<img width="661" height="78" alt="image" src="https://github.com/user-attachments/assets/44fb99d4-5c46-4353-a5a0-b3dbe255fbba" />  
<img width="899" height="145" alt="image" src="https://github.com/user-attachments/assets/350a70eb-0c0d-4ace-8143-aecc419943e2" />


### 4-2-2. `free -h` 中 total／used／free／buff/cache／available 各代表什麼？  

|語法|含義|
|---|---|
|total	|系統可使用的實體記憶體總容量|
|used	|系統目前計算為已使用的記憶體容量|
|free	|完全未被使用的記憶體|
|buff/cache	Linux Kernel| 用於緩衝區、檔案快取等的記憶體，其中部分可在需要時回收|
|available	Kernel |估計在不大量使用 Swap 的情況下，還能提供給新應用程式的記憶體容量|

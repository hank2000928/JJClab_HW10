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

## 4-3. Disk
| 指令       | 意義（請填） | 本機觀察摘要 |
| ---------- | ------------ | ------------ |
| `lsblk`    | 查看磁碟、分割區與區塊裝置的階層關係 |透過 lsblk 及 findmnt / 可確認，根目錄 / 使用 /dev/mapper/vg_system-lv_root，檔案系統類型為 XFS。此Logical Volume屬於Volume Group vg_system，其底層儲存裝置為 /dev/sda4，而 /dev/sda4 位於 VMware 的 60 GiB 虛擬磁碟 /dev/sda 中。因此本機根目錄使用 LVM 管理，而非直接使用一般磁碟分割區。|
| `fdisk -l` | 查看磁碟分割資訊，包括容量及分割表   |如下圖|
| `df -h`    | 查看已掛載檔案系統的容量與使用率     |本次透過 df -h 檢查 SLES 虛擬機的磁碟使用情況，發現根目錄 / 的使用率為 22%，/var 為 4%，/tmp 和 /home 均為 3%，/boot 為 13%。所有主要檔案系統均具有充足的可用容量，因此目前沒有接近滿載的現象。|

<img width="899" height="575" alt="image" src="https://github.com/user-attachments/assets/9beb0666-1994-4f52-ac64-96fc5aa4484a" />

<img width="912" height="53" alt="image" src="https://github.com/user-attachments/assets/1c9e3ddd-88d0-4f01-ad03-e6c6df8b514d" />

<img width="924" height="216" alt="image" src="https://github.com/user-attachments/assets/c122055f-7ccc-4e57-ae47-d247d4206e9c" />

## 4-4. NIC 介面（ip / ethtool）

| 小題  | 問題                                                          | 作答 |
| ----- | ------------------------------------------------------------- | ---- |
| 4-4-1 | `ip a` 與 `ifconfig` 在現代 SLES 上如何取捨？|優先使用 ip 指令，因為它是 iproute2 工具集的一部分，能夠查看及管理網路介面、IP 位址、路由及鄰居資訊。ifconfig 屬於較舊的 net-tools 工具集，雖然仍可用於查詢部分網路資訊，但功能與維護方式較不符合現代 Linux 的需求。   |
| 4-4-2 | `ethtool` 適合看哪些網卡層資訊（速率、雙工、驅動、offload）？ | ethtool 是 Linux 用於查詢及管理網路介面底層參數的工具，可查看網卡連結速度、全雙工或半雙工模式、自動協商、連結狀態及網卡驅動程式。|


## 4-5. UMA / NUMA 拓樸

| 小題  | 問題                                                                                                                                              | 作答 |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ---- |
| 4-5-1 | 解釋 **UMA（Uniform Memory Access）** 與 **NUMA（Non-Uniform Memory Access）** 的主要差異。|UMA 為均勻記憶體存取架構，CPU 存取主記憶體的延遲大致相同。NUMA 為非均勻記憶體存取架構，記憶體依 CPU Node 分布，CPU 存取本地記憶體通常比遠端記憶體具有更低的延遲。因此 NUMA 系統需要考慮 CPU 與記憶體之間的親和性，以減少跨 Node 存取所造成的效能損失。  |
| 4-5-2 | 一台雙 Socket 伺服器，如何確認系統共有幾個 **NUMA Node**，以及每個 Node 分別包含哪些 **CPU 邏輯核心** 與多少 **記憶體容量**？寫出指令與本機結果。 |本次使用 lscpu 與 numactl --hardware 檢查 VMware SLES 15 SP7 虛擬機的 NUMA 拓樸。系統共有 1 個 NUMA Node，編號為 Node 0，其中包含邏輯 CPU 0 與 CPU 1。Node 0 的記憶體容量為 3252 MB，查詢當下的剩餘記憶體為 1739 MB。|
| 4-5-3 | 如何顯示每個 Node 的記憶體 **size** 與 **free**？ |可以使用 numactl --hardware 顯示各 NUMA Node 的 CPU 清單、記憶體總容量（Size）與剩餘容量（Free）。也可直接讀取 /sys/devices/system/node/node*/meminfo。本次實驗中，Node 0 的 MemTotal 為 3330856 kB，MemFree 為 1781176 kB，與 numactl 的查詢結果一致。|
| 4-5-4 | 為什麼高效能與資料庫調校常需檢視 NUMA 拓樸？`hwloc-ls`／`lstopo` 適合看什麼？ | 多 Socket 的 HPC 伺服器或資料庫系統中，CPU 與記憶體可能分布於不同 NUMA Node。若行程頻繁存取遠端 Node 的記憶體，可能增加記憶體延遲並影響效能。因此系統調校時，需要分析 CPU 與記憶體的親和性，使行程盡可能使用本地記憶體。hwloc-ls 與 lstopo 能顯示 CPU Socket、Core、Cache、NUMA Node 及其他硬體資源的拓樸，有助於辨識硬體結構與規劃行程配置。     |

<img width="929" height="541" alt="image" src="https://github.com/user-attachments/assets/dde1b621-f118-4515-94f4-2b77dd0d0684" />

## 4-6. DMI：記憶體插槽與主機板（不停機盤點）

| 小題  | 任務                                                                                                                                                                      | 作答 |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- |
| 4-6-1 | 用 `dmidecode` 查出記憶體插槽與規格（Memory），以及主機板／系統廠商（System / Motherboard）。寫出指令，並記錄廠商、型號、現有模組 Size／Type／Speed（能看到的欄位即可）。 |VMware Virtual Platform；虛擬主機板識別為 Intel 440BX；RAM 為 4 GB，類型 DRAM，速度 Unknown|
| 4-6-2 | 用 `dmidecode` 確認主機板是否還有空的記憶體插槽（Slots）。哪個欄位表示「此槽未插模組」？本機空槽數量？  |DMI 列出 64 個 RAM 插槽，其中 1 個安裝、63 個未安裝；非實體擴充能力的證據|
| 4-6-3 | `dmidecode` 讀的是什麼資料？VM 中內容取決於什麼？ |dmidecode 讀取 SMBIOS/DMI 資訊；VM 中內容由虛擬化平台提供，不能直接當作 Host 的實體規格|

## 4-7. PCI：新插網卡系統看不到介面

| 小題  | 任務                                                                                                    | 作答 |
| ----- | ------------------------------------------------------------------------------------------------------- | ---- |
| 4-7-1 | 如何快速確認硬體層面是否成功偵測到這張網卡？寫出指令與相關 `lspci` 列。                                 | 使用 lspci | grep -iE 'ethernet|network' 可以查詢 PCI 裝置列表中的網路控制器。本次VMware虛擬機查詢到Intel Corporation 82545EM Gigabit Ethernet Controller，PCI Bus ID 為 02:01.0，表示 Guest OS 已成功辨識此虛擬網路控制器。|
| 4-7-2 | 如何查看該 PCIe 裝置目前載入的 **核心驅動程式（Kernel driver in use）** 與 **模組（Kernel modules）**？ |使用 lspci -k 可檢查 PCI 裝置的 Kernel driver in use 及 Kernel modules。本次查詢結果顯示，VMware 的 Intel 82545EM 虛擬網卡使用 e1000 驅動程式及核心模組。透過 ethtool -i eth0 再次確認，驅動為 e1000，PCI Bus ID 為 0000:02:01.0，兩者資訊相符。 |

<img width="919" height="453" alt="image" src="https://github.com/user-attachments/assets/13001873-fdb3-491c-95cf-d050797977ab" />

## 4-8. USB Storage：對應區塊裝置名稱

| 小題  | 任務                                                                           | 作答 |
| ----- | ------------------------------------------------------------------------------ | ---- |
| 4-8-1 | 如何查看外接硬碟對應到系統中的**區塊裝置名稱**？寫出指令與本機（或模擬）結果。 | 本次使用 lsusb 查看 USB 裝置，觀察到 VMware Virtual USB Hub、Virtual Mouse 以及 Linux Foundation USB Root Hub，未發現 USB 外接儲存裝置。使用 lsblk 確認目前主要區塊裝置為 /dev/sda（60 GiB VMware 虛擬磁碟）及 /dev/sr0（虛擬光碟機）。 |

### 4-8-2. 空間問題、裝置關係、效能問題，各應先看哪類指令？

|問題需求|推薦方法|判讀重點|
|---|---|---|
|檔案系統空間不足|	df -h|	Use%、Avail
|磁碟與分割區關係|	lsblk、lsblk -f|	裝置、分割區、LVM、掛載點
|USB 裝置偵測|	lsusb、dmesg|	新設備是否被辨識
|磁碟效能異常|	iostat -x、iotop|	I/O 延遲、裝置忙碌程度與占用行程

<img width="931" height="306" alt="image" src="https://github.com/user-attachments/assets/1d50aa46-ce6b-43fa-a8e2-3ecc17855517" />

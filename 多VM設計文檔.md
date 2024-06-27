# 多 VM 設計文檔

當前的 arceos hypervisor 僅支援運行一個虛擬機，然而，如此的 hypervisor 沒有太多現實上的意義，本文檔記錄如何將其增強為能夠同時執行多個虛擬機。

## CPU 虛擬化

### 時鐘中斷
在兩種狀況下 hypervisor 會設定時鐘中斷

1. hypervisor 在上下文切換時設定下一個時鐘中斷，以再次觸發上下文切換。
2. hypervisor 攔截虛擬機的時鐘設定請求，代為向 SBI 請求設定時鐘，並記錄虛擬機設定的時間。

當時鐘中斷發生， trap 進 hypervisor 時，hypervisor 檢查

1. 當前虛擬機時間片是否超時，若是，切換上下文，執行下一個虛擬機
2. 檢查各個虛擬機上一次的時鐘中斷請求時間是否已到，若是，將時鐘中斷注入該虛擬機的 hvip 寄存器。

### 上下文切換
除開 32 個通用寄存器，要實現虛擬機上下文切換，尚需在切換時保存/恢復

- vs 開頭的虛擬機系統寄存器，如 `vsstatus`, `vsie`, `vsepc`, `vscause` 等等，這些寄存器決定虛擬機的系統狀態。
- h 開頭，hypervisor 用於控制虛擬機的寄存器，不同虛擬機應有不同狀態，如 `hgatp` 控制虛擬機物理地址到實體機物理地址的頁表。


## 內存虛擬化
基本沒做虛擬化，就是一大塊內存映射給虛擬機，mmio 設備也是直接恆等映射給虛擬機。

## IO 虛擬化

### 輸入輸出
當僅有單一 VM 的情況下，執行完 arceos-hypervisor 後，console 的輸入輸出就由 VM 接管，但若多個 VM 同時運行，該如何表現？

透過網路取得 shell 或許是一個不錯的方式，就如同在雲端租用虛擬主機時，往往是透過 ssh 來操作虛擬主機，我們倒也不需要實作 ssh 安全性方面的功能，只要能透過網路介面來從虛擬機外部對虛擬機下達指令就行，但這仍舊需要虛擬出網路，評估時間可能不夠，便沒有往這方向繼續做。

#### 實現
最後我採取一個較為簡單的方案，當 hypervisor 啟動之後，在定期時鐘中斷，要進行上下文切換時，hypervisor 會讀取 console 輸入，用戶可手動控制當下獲取到的字符要注入到哪個虛擬機的 buffer 裡。（反引號 \` 用於切換注入對象）

但目前實作仍透過 SBI 來操作 console，不支援虛擬機採用 mmio 來操作 UART ，這大概是無法 Linux 的運行後無法正常輸入的原因。

### virtio
目前沒能虛擬化其他設備，因此虛擬機都是直通設備的，在編譯（arceos apps）階段會修改虛擬機映像，使得多個虛擬機不會操作到同一個設備。

當採用
```
cargo run --bin run_vm -- build
```
編譯虛擬機時，透過傳遞適當的環境變數，`axconfig` 的 build script 會選擇性忽略部分 `virtio-mmio-regions` ，使得虛擬機在偵測 mmio virtio 設備時忽略掉屬於其他虛擬機的設備。

## 配置 Arceos App 為虛擬機

hypervisor 啟動後，會嘗試在 `0x9000_0000` 上找設備樹，找到後會到 `0x9020_0000` 載入映像，接著往 `0xa0000_0000`, `0xb0000_0000` 繼續搜尋。

`run_vm` 有個腳本能夠根據 `run_vm/src/config.rs` 中所定義的虛擬機編譯映像，並生成相應的 qemu 執行指令。

可用以下結構定義一個虛擬機如何編譯及載入 qemu
```rs
pub struct VirtualMachine {
    pub os: &'static str, // ArceOS App 的路徑，例如 apps/fs/shell, apps/net/httpserver
    pub devices: Vec<Device>, // 虛擬機所需設備，qemu 會負責提供
    pub log: Log,         // 日誌等級，編譯用參數
    pub mode: Mode,       // debug 或 release ，編譯用參數，表示是否產生除錯符號、優化程度
}
```


## 實際使用
請見 [MROS/arceos-hypervisor multiple-vm 分支](https://github.com/MROS/arceos-hypervisor/tree/multiple-vm) 。

## 練習 1

### 1. 在 arceos-hypervisor 跑 Linux

RISCV 版的文件讓人去百度雲盤下載 RISC-V 的 Linux ，但我註冊不了百度雲盤，只好試着自己編譯，從 arm 版的文件改一改看能不能動。

無果，只好在群裡請人幫上傳到其他雲端。

### 2.
1. 增加了 HS, VS, VU ，VS 是虛擬機的 supervisor 模式，VU 是虛擬機上的用戶模式，注意此時 U 模式的應用仍可以跑在 HS 上。（猜想：所以 arceos-hypervisor 可以同時執行 App 跟虛擬機？如同 linux-kvm 一樣。）
2. 大概是 type-2 ，源碼看了才能確定。

### 練習 2

`apps/hv/src/main.rs` 中創建了 vcpu ，在 `vm.init_vcpu()` 執行前後打印 vcpu 的內容。
（derive Debug VmCpuRegisters 及其 field 之後可以直接打印整個結構）

比對後可發現，僅有 `virtual_hs_csrs` 裡的 `hgatp` 變了。 `hgatp` 指定了 GVA 到 HPA 的頁表位置。

其在 vcpu.init_page_map() 中被修改。

### 練習 3

`crates/hypercraft/src/arch/riscv/vcp.rs` 中可以看到其處理了時鐘中斷、外部中斷、缺頁異常、SBI call。

其中 SBI call 要再下到 M 模式。

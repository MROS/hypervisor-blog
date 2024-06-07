## 縮寫

暫存器系列
- GPR = General Purpose Registers
- tp = thread pointer = x4 暫存器，指向 進程控制塊/thread local storage。
- sepc = supervisor exception program counter = 陷入 S 模式時，觸發 S 模式的指令的虛擬地址。
- sstatus
    - spp
- hstatus
    - spv = Supervisor Previous Virtualization ， RISC-V 的 V 值標誌著是否處於虛擬態之中，當模式為 VU, VS 時，V 值為 1 ，HS, M 模式的 V 值為 0 ，spv 紀錄了 trap 之前的 V 值為何。
    - spvp = Supervisor Previous Virtual Privilege ，從 V = 1 狀態（即 VU 或 VS 態）trap 進 HS 態時，紀錄 trap 當下是 VU 還是 VS 。若是 VU ， spvp = 0 ，若是 VS ，spvp = 1。可理解為虛擬化版本的 status.spp。

分頁系列
- gpt = guest page table

其他
- hart = hardware threads

## hypercraft 定義之結構

### PerCpu
應對到真實的 CPU （？）

## VM 啓動流程

爲什麼要抽象出 vcpu ？

PerCpu 跟 vcpu 的關係是？

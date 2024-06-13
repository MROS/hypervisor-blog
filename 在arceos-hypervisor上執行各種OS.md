本文件記錄如何在 RISCV 架構上的 arceos-hypervisor 執行 Linux 之外的操作系統。

## 執行 arceos helloworld

ELD 生成時，有許多 `j` 跳轉指令，會直接跳到 ffffffc08020XXXX （沒有 address relocate），然而，riscv 的 hypervisor 會將虛擬機入口載入到 ffffffc090200000，需修改 kernel 的 base address 。

以下是逐步指令

### 先編譯 hypervisor
```sh
make A=apps/hv ARCH=riscv64 HV=y LOG=debug MODE=debug build
```

### 調整 BASE address
修改 modules/axconfig/src/platform/qemu-virt-riscv.toml 進行以下調整：
```sh
phys-memory-base = "0x9000_0000"
kernel-base-paddr = "0x9020_0000"
kernel-base-vaddr = "0xffff_ffc0_9020_0000"
```

### 編譯 helloworld

```sh
make ARCH=riscv64 A=apps/helloworld LOG=debug MODE=debug build
```

### 在 hypervisor 上執行 helloworld

TODO: helloworld 並不依賴 linux.dtb
```sh
qemu-system-riscv64 -m 3G -smp 1 -machine virt -bios default -kernel apps/hv/hv_qemu-virt-riscv.bin -device loader,file=apps/hv/guest/linux/linux.dtb,addr=0x90000000,force-raw=on -device loader,file=apps/helloworld/helloworld_qemu-virt-riscv.bin,addr=0x90200000,force-raw=on -append "rw console=ttyS0" -nographic
```

## 執行 arceos shell

### 調整 BASE address
修改 modules/axconfig/src/platform/qemu-virt-riscv.toml 進行以下調整：
```sh
phys-memory-base = "0x9000_0000"
kernel-base-paddr = "0x9020_0000"
kernel-base-vaddr = "0xffff_ffc0_9020_0000"
```

### 編譯 shell
```sh
make ARCH=riscv64 A=apps/fs/shell LOG=debug MODE=debug build
```

### hypercraft 補丁
若 [此 PR](https://github.com/arceos-hypervisor/hypercraft/pull/11) 尚未被合併，請手動打上補丁。

### 在 hypervisor 上執行 arceos shell
需要給他 file 
```sh
make disk_img
qemu-system-riscv64 -m 3G -smp 1 -machine virt -bios default -drive file=disk.img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0 -kernel apps/hv/hv_qemu-virt-riscv.bin -device loader,file=apps/hv/guest/linux/linux.dtb,addr=0x90000000,force-raw=on -device loader,file=apps/fs/shell/shell_qemu-virt-riscv.bin,addr=0x90200000,force-raw=on -append "rw console=ttyS0" -nographic
```

## 執行 nimbos
### 下載、編譯 nimbos
```sh
# set up cross-compile tools
# download
wget https://musl.cc/riscv64-linux-musl-cross.tgz
# install
tar zxf riscv64-linux-musl-cross.tgz
# exec below command in bash
export PATH=`pwd`/riscv64-linux-musl-cross/bin:$PATH
# OR add below info in ~/.bashrc
# echo PATH=`pwd`/riscv64-linux-musl-cross/bin:$PATH >> ~/.bashrc

# clone nimbos
git clone https://github.com/equation314/nimbos.git
cd nimbos/kernel

# set up rust tools
make env

# 修改 base addr
# 在 kernel/platforms/qemu-virt-riscv.toml 修改以下三屬性爲
# phys-memory-base = "0x9000_0000"
# kernel-base-paddr = "0x9020_0000"
# kernel-base-vaddr = "0xffff_ffc0_9020_0000"

# build nimbos
cd ../user && make ARCH=riscv64 build
cd ../kernel && make build ARCH=riscv64 MODE=debug
```

### 建立必要鏈結
```sh
cd ${ARCEOS-HYPERVISOR}
cd apps/hv/guest/nimbos
ln -s ${NIMBOS-PATH}/kernel/target/riscv64/debug/nimbos # 鏈結 bin 文件到 arceos-hypervisor 目錄下
ln -s ../linux/linux.dtb nimbos.dtb # 復用 Linux 設備樹文件
```

### hypercraft 補丁
若 [此 PR](https://github.com/arceos-hypervisor/hypercraft/pull/11) 尚未被合併，請手動打上補丁。

### 編譯執行
```sh
make ARCH=riscv64 A=apps/hv HV=y LOG=info GUEST=nimbos run
```

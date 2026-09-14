# images/

4 file cần chép vào **phân vùng boot (FAT32)** của thẻ SD khi flash lên BeagleBone Black.

| File | Source (gốc) | Bên trong |
|---|---|---|
| `MLO` | SPL của U-Boot — `board/ti/am335x/`, `arch/arm/cpu/armv7/`, `common/spl/` | Chỉ có code đã biên dịch, **không có device tree** (`CONFIG_SPL_OF_CONTROL` tắt), **không ký** (không ai verify được trên chip GP) |
| `u-boot.img` | Ghép từ 2 thứ đã biên dịch xong: `u-boot-nodtb.bin` (code) + 10 file `.dtb` trong `arch/arm/dts/` (`am335x-evm`, `am335x-bone`, ..., `am335x-boneblack`...) | **Đã nhúng public key** vào riêng device tree của board `am335x-boneblack` bằng `tools/fdt_add_pubkey -a sha256,rsa2048 -k keys -n dev -r conf`, làm **trước** bước đóng gói cuối cùng — không được chạy `make` sau khi nhúng key vì sẽ build lại dtb từ source, xoá mất key |
| `fitImage` | `zImage` (Linux kernel build) + dtb của kernel (`arch/arm/boot/dts/ti/omap/am335x-boneblack.dtb` trong **kernel** source — khác file cùng tên của U-Boot ở trên) | FIT tự viết (`scripts/fit-image.its`): `images/kernel-1` + `images/fdt-1` (mỗi cái kèm hash SHA-256), ký chung cả 2 qua `configurations/conf-1/signature-1` (chống mix-and-match) |
| `extlinux.conf` | Không có source riêng — bản gốc do Yocto tự sinh lúc build image | Vài dòng config cho cơ chế "generic distro boot" của U-Boot (`boot/pxe_utils.c` đọc file này). Bản trong repo đã sửa: `kernel /zImage` + `fdtdir /` → `kernel /fitImage`, bỏ `fdtdir` |

## `unsigned-reference/`

Bản **gốc, chưa ký** của `zImage`/2 device tree — chỉ để đối chiếu trước/sau, không dùng để flash.

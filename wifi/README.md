# Fix wifi initialization issue when the laptop reboots

## Install the latest firmware
```
# ./mt76/firmware has the latest firmware for the NIC
# firmware:       mediatek/WIFI_MT7922_patch_mcu_1_1_hdr.bin
# firmware:       mediatek/WIFI_RAM_CODE_MT7922_1.bin
# firmware:       mediatek/WIFI_MT7961_patch_mcu_1_2_hdr.bin
# firmware:       mediatek/WIFI_RAM_CODE_MT7961_1.bin
cd ./mt76/firmware
cp WIFI_MT7922_patch_mcu_1_1_hdr.bin /lib/firmware/mediatek/
cp WIFI_RAM_CODE_MT7922_1.bin /lib/firmware/mediatek/
cp WIFI_MT7961_patch_mcu_1_2_hdr.bin /lib/firmware/mediatek/
cp WIFI_RAM_CODE_MT7961_1.bin /lib/firmware/mediatek/
```

## Build and deploy kernel module
```zsh
# Apply the patch
cd mt76
git apply -p6 ../mt76-mt7921e-fix-possible-probe-failure-after-reboot.diff

# Build the driver
make -C /lib/modules/$(uname -r)/build M=$(pwd) CONFIG_MT7921_COMMON=m modules

# Check if the kernel modules are present
ls | grep .ko
ls ./mt7921 | grep .ko

# Make sure the following 4 files are present
# * mt76-connac-lib.ko
# * mt76.ko
# * ./mt7921/mt7921-common.ko
# * ./mt7921/mt7921e.ko
xz mt76-connac-lib.ko
xz mt76.ko
xz ./mt7921/mt7921-common.ko
xz ./mt7921/mt7921e.ko

# Move to kernel module directory
sudo mv ./mt76-connac-lib.ko.xz /lib/modules/$(uname -r)/kernel/drivers/net/wireless/mediatek/mt76
sudo mv ./mt76.ko.xz /lib/modules/$(uname -r)/kernel/drivers/net/wireless/mediatek/mt76
sudo mv ./mt7921/mt7921-common.ko.xz /lib/modules/$(uname -r)/kernel/drivers/net/wireless/mediatek/mt76/mt7921
sudo mv ./mt7921/mt7921e.ko.xz /lib/modules/$(uname -r)/kernel/drivers/net/wireless/mediatek/mt76/mt7921

# Deploy
sudo depmod

# Test
sudo modprobe -r mt7921e
sudo modprobe -r mt76_connac_lib
sudo modprobe -r mt76
sudo modprobe mt7921e
```

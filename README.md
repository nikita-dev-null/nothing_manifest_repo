# Манифест сборки ядра для Nothing CMF Phone (1) (mt6878)

Этот репозиторий содержит манифест для утилиты `repo`. Он объединяет официальное GKI ядро от Google и наши кастомные форки для смартфона Nothing CMF Phone (1), позволяя развернуть окружение одной командой.

## ⚖️ Дисклеймер / Disclaimer

Используйте код с осторожностью. Ваше устройство — ваше дело. Автор данного проекта не несет никакой ответственности за повреждение вашего телефона, потерю данных, сгоревшие SD-карты или проблемыс законом, возникшие в результате использования данного программного обеспечения. Вы устанавливаете это ядро на свой страх и риск. Программное обеспечениепредоставляется "КАК ЕСТЬ" (AS IS), без каких-либо явных или подразумеваемых гарантий.

## 🚀 Инструкция по сборке с нуля

### 1. Подготовка рабочего пространства
Создайте чистую папку для проекта и перейдите в неё:
```bash
mkdir cmf_kernel_workspace && cd cmf_kernel_workspace
```

### 2. Инициализация манифеста и скачивание исходников
Утилита `repo` автоматически скачает тяжелую неизменяемую базу у Google, а исходники ядра Nothing и модулей — из форков (содержат изменения в bazel скриптах для успешной сборки):
```bash
repo init -u https://github.com/nikita-dev-null/nothing_manifest_repo
repo sync -c --no-tags
```

### 3. Запуск компиляции
После успешного завершения синхронизации запустите:
```bash
python3 kernel_device_modules-6.1/scripts/gen_build_config.py -p mgk_64_k61 -o ./build.config.legacy -m user --kernel-defconfig-overlays ''

BUILD_CONFIG=build.config.legacy tools/bazel build \
  --override_repository=mgk_info=build/bazel_mgk_rules \
  --override_repository=mgk_internal=build/bazel_mgk_rules/kleaf \
  --nokmi_symbol_list_strict_mode \
  --nokmi_symbol_list_violations_check \
  //kernel_device_modules-6.1:mgk_64_k61.user

python3 /disk/cmf_kernel_workspace/android-kernel/tools/mkbootimg/mkbootimg.py   --kernel /disk/cmf_kernel_workspace/android-kernel/bazel-out/k8-fastbuild/bin/kernel_device_modules-6.1/mgk_64_k61_kernel_aarch64.user/Image.lz4   --header_version 4  -o custom_boot.img

```

### 4. Прошивка
Только ядро
```bash
fastboot flash boot_a custom_boot.img
```
Или все
```bash
fastboot set_active a

fastboot flash boot_a boot.img

fastboot flash dtbo_a dtbo.img

fastboot flash init_boot_a init_boot.img

fastboot flash vendor_boot_a vendor_boot.img

for i in apusys ccu connsys_bt connsys_gnss connsys_wifi dpm gpueb gz lk logo mcf_ota mcupm modem pi_img scp spmfw sspm tee vcp; do fastboot flash "${i}_a" "${i}.img"; done

fastboot flash vbmeta_a vbmeta.img --disable-verity --disable-verification

fastboot flash preloader_a preloader_raw.img

fastboot flash vbmeta_system_a vbmeta_system.img --disable-verity --disable-verification

fastboot flash vbmeta_vendor_a vbmeta_vendor.img --disable-verity --disable-verification

fastboot reboot fastboot

for i in odm_dlkm odm vendor_dlkm product vendor system_dlkm system_ext system; do for s in a b; do fastboot delete-logical-partition "${i}_${s}-cow"; fastboot delete-logical-partition "${i}_${s}"; fastboot create-logical-partition "${i}_${s}" 2; done; done

for i in odm_dlkm odm vendor_dlkm product vendor system_dlkm system_ext system; do fastboot flash "${i}_a" "${i}.img"; done

fastboot set_active a

fastboot reboot

```

## Полезное
[Стоковая прошивка Tetris_B4.1-260415-1709](https://github.com/spike0en/nothing_archive/releases/tag/Tetris_B4.1-260415-1709)

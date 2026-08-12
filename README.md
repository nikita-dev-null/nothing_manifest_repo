# Манифест сборки ядра для Nothing CMF Phone (1) (mt6878)

Этот репозиторий содержит манифест для утилиты `repo`. Он объединяет официальное GKI ядро от Google и наши кастомные форки для смартфона Nothing CMF Phone (1), позволяя развернуть окружение одной командой.

## 🚀 Инструкция по сборке с нуля

### 1. Подготовка рабочего пространства
Создайте чистую папку для проекта и перейдите в неё:
```bash
mkdir cmf_kernel_workspace && cd cmf_kernel_workspace
```

### 2. Инициализация манифеста и скачивание исходников
Утилита `repo` автоматически скачает тяжелую неизменяемую базу у Google, а исходники ядра Nothing и модулей — из наших форков:
```bash
repo init -u https://github.com
repo sync -c --no-tags
```

### 3. Запуск компиляции
После успешного завершения синхронизации скопируйте скрипт автоматической сборки в корень проекта и запустите его:
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


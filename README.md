# Running [NVIDIA® DesignWorks™ Samples](https://github.com/nvpro-samples) on Jetson™ Devices

The DesignWorks sample projects depend on [nvpro_core](https://github.com/nvpro-samples/nvpro_core), which relies on the Vulkan® SDK to be installed. [Jetson Linux 34.x](https://developer.nvidia.com/embedded/jetson-linux) comes with a pre-installed [Vulkan 1.3](https://developer.nvidia.com/embedded/vulkan) library instead. So what's left is installing the [Vulkan include headers](https://github.com/KhronosGroup/Vulkan-Headers) and `glslangValidator`—a tool used to pre-compile shaders. It's part of the [KhronosGroup/glslang](https://github.com/KhronosGroup/glslang) repository and requires [CMake](https://cmake.org/download/) 3.18 or later.

### Upgrading CMake
```
wget https://github.com/Kitware/CMake/releases/download/v3.26.4/cmake-3.26.4-Linux-aarch64.tar.gz -q --show-progress
tar -zxvf cmake-3.26.4-Linux-aarch64.tar.gz
cd cmake-3.26.4-Linux-aarch64/
sudo cp -rf bin/ doc/ share/ /usr/local/
sudo cp -rf man/* /usr/local/man
sync
cd ..
rm -rf cmake-3.26.4-Linux-aarch64.tar.gz
sudo apt remove cmake
```

### Installing glslang
```
git clone https://github.com/KhronosGroup/glslang.git
cd glslang
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$(pwd)/install" ..
make -j4 install
sudo mv install/bin/glslangValidator /usr/bin/
cd ../..
```

### Installing Vulkan Include Headers
```
git clone https://github.com/KhronosGroup/Vulkan-Headers.git
cd Vulkan-Headers/
git checkout -q tags/v1.3.204
cmake -S . -B build/
sudo cmake --install build --prefix /usr/
cd ..
```


## Building DesignWorks Sample Projects
Some sample projects have additional dependencies like [FreeImage](https://freeimage.sourceforge.io/index.html) and [NVML](https://developer.nvidia.com/nvidia-management-library-nvml). Even though the CUDA package contains the necessary include headers, NVML is [not supported on Jetson devices](https://forums.developer.nvidia.com/t/use-nvml-api-on-jetson/248965) which is why `_add_package_NVML()` needs to be removed from the samples' CMake scripts.

### Installing FreeImage
```
wget http://downloads.sourceforge.net/freeimage/FreeImage3180.zip -q --show-progress
unzip -q FreeImage3180.zip
rm -rf FreeImage3180.zip
cd FreeImage
```
Open **Makefile.gnu** and append `-DPNG_ARM_NEON_OPT=0` to the `CFLAGS` variable in order to disable compiler optimizations for Arm® NEON™ processors and prevent *undefined reference* errors.
```
make
make install
cd ..
```
---

### Patching nvpro_core
```
git clone --recursive --shallow-submodules https://github.com/nvpro-samples/nvpro_core.git
cd nvpro_core
git apply ../nvpro_core.patch
cd ..
```

>`-DVK_ENABLE_BETA_EXTENSIONS=OFF` disables some unsupported extensions like `VK_KHR_video_queue`.

### Running [vk_raytrace](https://github.com/nvpro-samples/vk_raytrace)
```
git clone https://github.com/nvpro-samples/vk_raytrace.git
cd vk_raytrace
git apply ../vk_raytrace.patch
cmake -DVK_ENABLE_BETA_EXTENSIONS=OFF -S . -B build/
cd build
make
./../../bin_x64/Release/vk_raytrace_app
```

### Running [vk_mini_path_tracer](https://github.com/nvpro-samples/vk_mini_path_tracer)
```
git clone https://github.com/nvpro-samples/vk_mini_path_tracer.git
cd vk_mini_path_tracer
cmake -DVK_ENABLE_BETA_EXTENSIONS=OFF -S . -B build/
cd build
make
./../../bin_x64/Release/vk_mini_path_tracer_app
```

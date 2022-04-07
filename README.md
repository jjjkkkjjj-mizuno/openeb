# OpenEB

OpenEB is the open source project associated with [Metavision Intelligence](https://www.prophesee.ai/metavision-intelligence/)

It enables anyone to get a better understanding of event-based vision, directly interact with events and build
their own applications or plugins. As a camera manufacturer, ensure your customers benefit from the most advanced 
event-based software suite available by building your own plugin. As a creator, scientist, academic, join and contribute
to the fast-growing event-based vision community.

OpenEB is composed of the 5 Open modules of Metavision Intelligence:
* HAL: Hardware Abstraction Layer to operate any event-based vision device.
* Base: Foundations and common definitions of event-based applications.
* Core: Generic algorithms for visualization, event stream manipulation, applicative pipeline generation.
* Driver: High-level abstraction built on the top of HAL to easily interact with event-based cameras.
* UI: Viewer and display controllers for event-based data.

This document describes how to compile and install the OpenEB codebase.
For further information, refer to our [online documentation](https://docs.prophesee.ai/) where you will find
some [tutorials](https://docs.prophesee.ai/stable/metavision_sdk/tutorials/index.html) to get you started in C++ or Python,
some [samples and applications](https://docs.prophesee.ai/stable/samples.html) to discover how to use
[our API](https://docs.prophesee.ai/stable/api.html) and a more detailed
[description of our modules and plans](https://docs.prophesee.ai/stable/modules.html).


## Compiling on Ubuntu

Currently, we support Ubuntu 18.04 and 20.04.

### Prerequisites

Install the following dependencies:

```bash
sudo apt update
sudo apt -y install apt-utils build-essential software-properties-common wget unzip curl
sudo apt -y install cmake libopencv-dev git
sudo apt -y install libboost-all-dev libusb-1.0-0-dev libeigen3-dev
sudo apt -y install libglew-dev libglfw3-dev
sudo apt -y install libgtest-dev
```

For the Python API, you will need Python and some additional libraries.
If Python is not available on your system, install it
(we support Python 3.6 and 3.7 on Ubuntu 18.04 and Python 3.7 and 3.8 on Ubuntu 20.04).
Then install some extra libraries:

```bash
sudo apt -y install python3-pip python3-distutils
python3 -m pip install pip --upgrade
python3 -m pip install "numpy==1.19.5" "opencv-python>=4.2.0.34" pytest
```

If you want to run tests, then you need to compile **gtest** package (this is optional):

```bash
cd /usr/src/gtest
sudo cmake .
sudo make
sudo make install
```

The Python bindings rely on the [pybind11](https://github.com/pybind) library, specifically version 2.6.0.

*Note* that pybind11 is required only if you want to use the Python bindings of our C++ API.
You can opt out of creating these bindings by passing the argument `-DCOMPILE_PYTHON3_BINDINGS=OFF` at step 3 during compilation (see below).
In that case, you will not need to install pybind11, but you won't be able to use our Python interface.

Unfortunately, there is no pre-compiled version of pybind11 available, so you need to install it manually:

```bash
wget https://github.com/pybind/pybind11/archive/v2.6.0.zip
unzip v2.6.0.zip
cd pybind11-2.6.0/
mkdir build && cd build
cmake .. -DPYBIND11_TEST=OFF
cmake --build .
sudo cmake --build . --target install
```

### Compilation

 1. Retrieve the code `git clone https://github.com/prophesee-ai/openeb.git`
 2. Create and open the build directory in the `openeb` folder (absolute path to this directory is called `OPENEB_SRC_DIR` in next sections): `cd openeb; mkdir build && cd build`
 3. Generate the makefiles using CMake: `cmake .. -DBUILD_TESTING=OFF`
 4. Compile: `cmake --build . --config Release -- -j 4`
 
You can now use OpenEB directly from the build folder.
For this, you will need to update your environment variables using this script
(which you may add to your ~/.bashrc to make it permanent):

```bash
source <OPENEB_SRC_DIR>/build/utils/scripts/setup_env.sh
```

Optionally, you can deploy the OpenEB files in the system paths to use them as 3rd party dependency in some other code
with the following command: `sudo cmake --build . --target install`. In that case, you will also need to update 
`LD_LIBRARY_PATH` with `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib` (If you want to update this path
permanently, you should add the previous command in your ~/.bashrc)

To get started with OpenEB, you can download some [sample recordings](https://docs.prophesee.ai/stable/datasets.html) and stream them with [metavision_viewer](https://docs.prophesee.ai/stable/metavision_sdk/modules/driver/guides/viewer.html#chapter-sdk-driver-samples-viewer). The RAW plugins included in OpenEB will allow you to read the RAW files.
However, if you are planning to use a Prophesee camera, then you need to install **Prophesee plugins** by following this procedure:

 * Go to the [sign-up page for Prophesee Camera Plugins](https://www.prophesee.ai/metavision-intelligence-plugins-download/)
 * Download the `.list` file for your version of Ubuntu and add it to the folder `/etc/apt/sources.list.d`
 * Install Prophesee plugins:

```
sudo apt update
sudo apt install metavision-hal-prophesee-plugins
```

*Note* that the previous command will download the Prophesee Camera Plugins for the latest version of OpenEB. 
If you are using a previous version of OpenEB, specify the version number in your command: `sudo apt -y install 'metavision-hal-prophesee-plugins=x.y.z'`


### Running the test suite (Optional)


Running the test suite is a sure-fire way to ensure you did everything well with your compilation and installation process.

*Note* that the [Prophesee Camera Plugins](https://www.prophesee.ai/metavision-intelligence-plugins-download/) must be installed for most of these tests to run.

 * Download [the files](https://dataset.prophesee.ai/index.php/s/VfhepWI4MPVWh9o) necessary to run the tests.
   Click `Download` on the top right folder. Beware of the size of the obtained archive which weighs around 500 Mb.

 * Extract and put the content of this archive to `<OPENEB_SRC_DIR>/`. For instance, the correct path of sequence `gen31_timer.raw` should be `<OPENEB_SRC_DIR>/datasets/openeb/gen31_timer.raw`.

 * Regenerate the makefiles with the test options on.

  ```bash
  cd <OPENEB_SRC_DIR>/build
  cmake .. -DBUILD_TESTING=ON
  ```

 * Compile again.  `cmake --build . --config Release -- -j 4`

 * Finally, run the test suite:   `ctest --verbose`

## Compiling on Windows

### Prerequisites

Some steps of this procedure don't work on FAT32 and exFAT file system.
Hence, make sure that you are using a NTFS file system before going further.

You must enable the support for long paths:
 * Hit the Windows key, type gpedit.msc and press Enter
 * Navigate to Local Computer Policy > Computer Configuration > Administrative Templates > System > Filesystem
 * Double-click the "Enable Win32 long paths" option, select the "Enabled" option and click "OK"

To compile OpenEB, you will need to install some extra tools:

 * install [cmake](https://cmake.org/)
 * install Microsoft C++ compiler (64-bit). You can choose one of the following solutions:
    * For building only, you can install MS Build Tools (free, part of Windows 10 SDK package)
      * Download and run ["Build tools for Visual Studio 2019" installer](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
      * Select "C++ build tools", make sure Windows 10 SDK is checked, and add English Language Pack
    * For development, you can also download and run [Visual Studio Installer](https://visualstudio.microsoft.com/fr/downloads/)    
 * install [vcpkg](https://github.com/microsoft/vcpkg) that will be used for installing dependencies:
    * download and extract [vcpkg version 2020.11-1](https://github.com/microsoft/vcpkg/archive/refs/tags/2020.11-1.zip)
    * patch a configuration file to fix a known issue with this version of vcpkg. In the file `C:\vcpkg-2020.11-1\scripts\cmake\vcpkg_acquire_msys.cmake`,
      on line 81, add `"https://mirrors.zju.edu.cn/msys2/"` (this is adding a new mirror server). This can be achieved with the following command
      (to adapt to the path in which you extracted vcpkg): 
      ```bash
      powershell -command "& {&'Invoke-WebRequest' -OutFile C:\vcpkg-2020.11-1\scripts\cmake\vcpkg_acquire_msys.cmake -Uri https://files.prophesee.ai/share/dists/public/vcpkg/vcpkg_acquire_msys.cmake}"
      ```
    * `cd <VCPKG_SRC_DIR>`
    * `bootstrap-vcpkg.bat`
  * finally, install the libraries by running `vcpkg.exe install --triplet x64-windows libusb eigen3 boost opencv glfw3 glew gtest dirent`
    * Note that to avoid using `--triplet x64-windows`, which informs vcpkg to install packages for a x64-windows target,
      you can run `setx VCPKG_DEFAULT_TRIPLET x64-windows` (you need to close the command line and re-open it to ensure that this variable is set)

#### Install pybind

The Python bindings rely on the [pybind11](https://github.com/pybind) library (version 2.6.0).
You should install pybind using vcpkg in order to get the appropriate version: `vcpkg.exe install --triplet x64-windows pybind11`

*Note* that pybind11 is required only if you plan to use the Python API.
You can opt out of creating these bindings by passing the argument `-DCOMPILE_PYTHON3_BINDINGS=OFF` at step 2 during compilation (see section "Compilation using CMake").
In that case, you will not need to install pybind11, but you won't be able to use our Python interface.


#### Install Python 3.7 or 3.8

* Download "Windows x86-64 executable installer" for one of these Python versions:
  * [Python 3.7](https://www.python.org/downloads/release/python-379/)
  * [Python 3.8](https://www.python.org/downloads/release/python-389/)
* We advise you to check the box to update the `PATH` or update the `PATH` manually with the following paths
  after replacing the *Username* to your own and using the Python version you installed
  (here, we assume that the install is limited to the local user and the default install path was used):

```bash
C:\Users\Username\AppData\Local\Programs\Python\Python37
C:\Users\Username\AppData\Local\Programs\Python\Python37\Scripts
````

* Finally, install additionally required Python packages using pip:

```bash
python -m pip install pip --upgrade
python -m pip install "numpy==1.19.5" "opencv-python>=4.2.0.34" pytest
```

### Compilation

First, retrieve the codebase:

```bash
git clone https://github.com/prophesee-ai/openeb.git
```

#### Compilation using CMake

Open a command prompt inside the `openeb` folder (absolute path to this directory is called `OPENEB_SRC_DIR` in next sections) and do as follows:

 1. Create and open the build directory, where temporary files will be created: `mkdir build && cd build`
 2. Generate the makefiles using CMake: `cmake -A x64 -DCMAKE_TOOLCHAIN_FILE=<OPENEB_SRC_DIR>\cmake\toolchains\vcpkg.cmake -DVCPKG_DIRECTORY=<VCPKG_SRC_DIR> ..`.
    Note that the value passed to the parameter `-DCMAKE_TOOLCHAIN_FILE` must be an absolute path, not a relative one. 
 3. Compile: `cmake --build . --config Release --parallel 4`
 
You can now use OpenEB directly from the build folder.
For this, you will need to update your environment variables using this script:

```bash
<OPENEB_SRC_DIR>\build\utils\scripts\setup_env.bat
```

Optionally, you can deploy the OpenEB files (applications, samples, libraries etc.) in a directory of your choice.
To do so, configure the target folder (`OPENEB_INSTALL_DIR`) with `CMAKE_INSTALL_PREFIX` variable 
(default value is `C:\Program Files\Prophesee`) when generating the makefiles in step 2:

```bash
cmake .. -A x64 -DCMAKE_TOOLCHAIN_FILE=<OPENEB_SRC_DIR>\cmake\toolchains\vcpkg.cmake -DVCPKG_DIRECTORY=<VCPKG_SRC_DIR> -DCMAKE_INSTALL_PREFIX=<OPENEB_INSTALL_DIR> -DBUILD_TESTING=OFF
```

You can also configure the directory where the Python packages will be deployed using the `PYTHON3_SITE_PACKAGES` variable
(note that in that case, you will also need to edit your environment variable `PYTHONPATH` and append the `<PYTHON3_PACKAGES_INSTALL_DIR>` path):

```bash
cmake .. -A x64 -DCMAKE_TOOLCHAIN_FILE=<OPENEB_SRC_DIR>\cmake\toolchains\vcpkg.cmake -DVCPKG_DIRECTORY=<VCPKG_SRC_DIR> -DCMAKE_INSTALL_PREFIX=<OPENEB_INSTALL_DIR> -DPYTHON3_SITE_PACKAGES=<PYTHON3_PACKAGES_INSTALL_DIR> -DBUILD_TESTING=OFF
```

Once you performed this configuration, you can launch the actual installation of the OpenEB files:

```bash
cmake --build . --config Release --target install
```

#### Compilation using MS Visual Studio

Open a command prompt inside the `openeb` folder and do as follows:

 1. Create and open the build directory, where temporary files will be created: `mkdir build && cd build`
 2. Generate the Visual Studio files using CMake: `cmake -G "Visual Studio 16 2019" -A x64 -DCMAKE_TOOLCHAIN_FILE=<OPENEB_SRC_DIR>\cmake\toolchains\vcpkg.cmake -DVCPKG_DIRECTORY=<VCPKG_SRC_DIR> ..` (adapt to your Visual Studio version).
    Note that the value passed to the parameter `-DCMAKE_TOOLCHAIN_FILE` must be an absolute path, not a relative one.
 3. Open the solution file `metavision.sln`, select the `Release` configuration and build the `ALL_BUILD` project.

#### Installing Prophesee Plugins

To get started with OpenEB, you can download some [sample recordings](https://docs.prophesee.ai/stable/datasets.html) and stream them with [metavision_viewer](https://docs.prophesee.ai/stable/metavision_sdk/modules/driver/guides/viewer.html#chapter-sdk-driver-samples-viewer). The RAW plugins included in OpenEB will allow you to read the RAW files.
However, if you are planning to use a Prophesee camera, then you need to install **Prophesee plugins** by following this procedure:

 * Go to the [sign-up page for Prophesee Camera Plugins](https://www.prophesee.ai/metavision-intelligence-plugins-download/)
 * Follow the Camera Plugins download link provided after sign-up
 * Among the list of Camera Plugins installers, download the one with the version number matching your OpenEB version
 * Run the installer

### Running the test suite (Optional)


Running the test suite is a sure-fire way to ensure you did everything well with your compilation and installation process.

*Note* that the [Prophesee Camera Plugins](https://www.prophesee.ai/metavision-intelligence-plugins-download/) must be installed for most of these tests to run.

 * Download [the files](https://dataset.prophesee.ai/index.php/s/VfhepWI4MPVWh9o) necessary to run the tests.
   Click `Download` on the top right folder. Beware of the size of the obtained archive which weighs around 500 Mb.
   
 * Extract and put the content of this archive to `<OPENEB_SRC_DIR>/`. For instance, the correct path of sequence `gen31_timer.raw` should be `<OPENEB_SRC_DIR>/datasets/openeb/gen31_timer.raw`.

 * To run the test suite you need to reconfigure your build environment using CMake and to recompile


   * Compilation using CMake

    1. Regenerate the build using CMake (note that `-DCMAKE_TOOLCHAIN_FILE` must be absolute path, not a relative one)::

        ```
        cd <OPENEB_SRC_DIR>/build
        cmake -A x64 -DCMAKE_TOOLCHAIN_FILE=<OPENEB_SRC_DIR>\cmake\toolchains\vcpkg.cmake -DVCPKG_DIRECTORY=<VCPKG_SRC_DIR> -DBUILD_TESTING=ON ..
        ```
    2. Compile: `cmake --build . --config Release --parallel 4`


   * Compilation using MS Visual Studio

    1. Generate the Visual Studio files using CMake (adapt the command to your Visual Studio version):

        `cmake -G "Visual Studio 16 2019" -A x64 -DCMAKE_TOOLCHAIN_FILE=<OPENEB_SRC_DIR>\cmake\toolchains\vcpkg.cmake -DVCPKG_DIRECTORY=<VCPKG_SRC_DIR> -DBUILD_TESTING=ON ..`

        Note that the value passed to the parameter `-DCMAKE_TOOLCHAIN_FILE` must be an absolute path, not a relative one.

    2. Open the solution file `metavision.sln`, select the `Release` configuration and build the `ALL_BUILD` project.

 * Running the test suite is then simply `ctest -C Release`

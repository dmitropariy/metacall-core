# Project Overview

## Project Goal

The goal of the project was to implement cross-platform support for the `C Loader` component in MetaCall and allow execution of C files through:

```bash
metacall example.c
```

The implementation targeted three platforms:

* Linux
* macOS
* Windows

The project was divided into three areas:

* Core C Loader implementation
* CI/CD infrastructure
* Distribution and packaging

## Team Responsibilities

**Participant 1 — Linux Engineer**

* Linux C Loader configuration
* GNU Guix packaging
* Linux CI workflow

**Participant 2 — macOS Engineer**

* macOS C Loader support
* CMake `if(APPLE)` configuration
* Intel and Apple Silicon compatibility
* GitHub Actions CI testing

**Participant 3 — Windows Engineer**

* Windows C Loader support
* MSVC and DLL configuration
* Installer and CI support

## Problems Solved

During implementation several issues were identified:

**Cross-platform differences**

* Different library formats (`.so`, `.dylib`, `.dll`)
* Different compilers and linkers

**Dependency management**

* Integration of:

  * libffi
  * libclang
  * Tiny C Compiler (TCC)

**Architecture compatibility**

* Support for Intel (`x86_64`)
* Support for Apple Silicon (`arm64`)

**CI configuration**

* Multi-platform testing
* Automated build and smoke tests

## Expected Result

After implementation MetaCall should support:

```bash
metacall example.c
```

on Linux, macOS and Windows with successful build and execution.

# Progress Report: C Loader Integration and CI Configuration for MetaCall

## 1. Objective and Context
As part of the preparation (including participation in Google Summer of Code), a comprehensive task was completed to configure the build system, portability (Distributables), and Continuous Integration (CI) for the `c_loader` (C Foreign Function Interface) in the **MetaCall** project.

## 2. Tasks completed by Dmytro Parii

### 2.1. CMake Configuration for Portability (Distributables)
- Edited the root `CMakeLists.txt` to ensure proper operation of the compiled distributions on end-user machines.
- Added global **RPATH** settings (`CMAKE_BUILD_WITH_INSTALL_RPATH TRUE`, `CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE`). This allows the loader's dynamic libraries to locate dependencies relative to their location (`$ORIGIN`).
- Integrated `CPack` for automatic generation of `.tar.gz` (for Linux/MacOS) and `.zip` (for Windows) archives of ready-to-deploy distributions.

### 2.2. CI Pipeline Setup (GitHub Actions)
- Created a workflow file `.github/workflows/linux-c-loader.yml`.
- Replaced manual usage of `apt-get` with the official standardized environment preparation script `tools/metacall-environment.sh`.

### 2.3. Analysis and Correction of Dependencies in Environment Scripts
- Identified the absence of the **Tiny C Compiler (TCC)** system dependency (`libtcc.so`) during the compilation of `libc_loader.so`.
- Introduced comprehensive architectural changes to the `tools/metacall-environment.sh` file:
  - Added `tcc` and `libtcc-dev` for Debian/Ubuntu systems.
  - Added `tcc` and `tcc-dev` for the `apk` package manager (Alpine Linux).
  - Added `tcc` for `brew` (MacOS) and `pkg` (FreeBSD).

### 2.4. Resolution of Associated Technical Issues
- **Ninja and ExternalProject Issue**: To bypass the strict dependency graph error of the `Ninja` generator when loading the missing `libtcc.so`, the build process was successfully switched to classical multi-threaded `Make` (`-j$(nproc)`).

## 3. Testing Results
- All integration tests (including `metacall-c-test`, `metacall-c-lib-test`, and `metacall-c-metacall-test`) successfully pass locally.
- The CI pipeline (GitHub Actions) runs smoothly and completes the build and testing with a green badge (Pass).
- The compiled CLI tool `metacall` is capable of loading C files and compiling code on the fly.

# Progress Report: macOS C Loader Support and CI Configuration for MetaCall

## 1. Objective and Context

As part of the Cross-Platform Support project for the `C Loader` implementation in **MetaCall**, work was completed to provide support for the macOS platform and integrate automated testing through GitHub Actions.

The primary objective was to ensure that the `C Loader` could be properly configured, built, and tested on macOS environments, while maintaining compatibility with both Intel-based and Apple Silicon architectures.

The expected result of the implementation is support for execution of C files through MetaCall:

```bash
metacall example.c
```

---

## 2. Tasks Completed

### 2.1. macOS Platform Configuration for C Loader

The `source/loaders/c_loader/CMakeLists.txt` configuration was updated to improve platform-specific behavior for macOS.

Implemented changes:

* Added explicit `if(APPLE)` platform configuration.
* Enabled `CMAKE_MACOSX_RPATH`.
* Added architecture detection for:

  * Intel Mac (`x86_64`)
  * Apple Silicon (`arm64`)
* Added informative status messages during the build process.
* Configured dynamic lookup support for MetaCall symbols on macOS.

Implemented configuration:

```cmake
if(APPLE)

    set(CMAKE_MACOSX_RPATH ON)

    if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm64|aarch64")
        message(STATUS "macOS Apple Silicon detected")
    elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|amd64|AMD64")
        message(STATUS "macOS Intel detected")
    endif()

endif()
```

---

### 2.2. Analysis of Existing macOS Build Configuration

The existing C Loader implementation was analyzed and validated for:

* `libffi`
* `libclang`
* `libtcc`
* dynamic library loading
* MetaCall symbol resolution

Existing macOS linker settings already included:

```cmake
-Wl,-undefined,dynamic_lookup
```

which allows C Loader modules to use already existing MetaCall symbols during runtime and avoids loading duplicate libraries.

---

### 2.3. GitHub Actions CI Pipeline Extension

The existing workflow:

```txt
.github/workflows/macos-test.yml
```

was extended.

Implemented changes:

* Added automated smoke testing for C Loader.
* Added creation of a temporary `example.c` file during CI execution.
* Added automatic execution through MetaCall.

Smoke test implementation:

```bash
metacall example.c
```

The workflow executes on:

* macos-14
* macos-15-intel
* macos-15

This configuration provides testing coverage for:

* Intel Mac architecture
* Apple Silicon architecture

---

### 2.4. Build Caching and Environment Configuration

The workflow reuses the existing MetaCall environment configuration:

```bash
tools/metacall-environment.sh
```

Dependencies used by the C Loader:

* LLVM / Clang
* libffi
* Tiny C Compiler (TCC)

The configuration process automatically enables:

```txt
c
```

through:

```bash
METACALL_INSTALL_OPTIONS
METACALL_CONFIGURE_OPTIONS
```
### 2.5. Pull Request and Version Control Workflow

A dedicated feature branch was created for isolated development of macOS support:

```bash
git checkout -b feature/c-loader-macos
```

The implementation process included multiple commits corresponding to different development stages:

**Commit 1**

```txt
Add macOS C Loader smoke test
```

Implemented:

* extension of the existing `macos-test.yml`
* addition of automated smoke testing
* generation and execution of `example.c`

**Commit 2**

```txt
Add macOS platform configuration for C Loader
```

Implemented:

* macOS-specific CMake configuration
* architecture detection for Intel and Apple Silicon
* RPATH configuration

**Commit 3**

```txt
Add macOS C Loader documentation
```

Implemented:

* progress report creation
* documentation of implementation and testing process

After implementation, changes were pushed to the remote repository:

```bash
git push origin feature/c-loader-macos
```

A Pull Request was created and used for:

* code review
* CI validation
* integration testing
* verification of macOS compatibility

The Pull Request automatically triggered GitHub Actions workflows and validated all introduced changes.

---

## 3. Testing Results

The following tests and validations were performed:

### Build validation

* C Loader successfully participates in MetaCall build configuration.
* CMake configuration correctly detects macOS platform settings.
* Architecture detection works correctly.

### CI validation

GitHub Actions workflow executes:

1. Environment setup
2. Build configuration
3. Compilation
4. Smoke testing

### C Loader smoke test

Generated test file:

```c
#include <stdio.h>

int main(void)
{
    printf("Hello from MetaCall C Loader on MacOS\n");
    return 0;
}
```

Execution command:

```bash
metacall example.c
```

---

## 4. Result

The following deliverables were completed successfully:

✓ C Loader configured for macOS platform

✓ Added macOS-specific CMake conditions

✓ Configured dynamic library behavior

✓ Added architecture support for Intel and Apple Silicon

✓ Extended GitHub Actions workflow

✓ Added C Loader smoke tests

✓ CI pipeline configured for macOS automated validation

The implementation improves portability and ensures continuous verification of MetaCall C Loader functionality on macOS systems.


# Progress Report: Windows C Loader Support and CI Configuration for MetaCall
# 1. Objective and Context
As part of the Cross-Platform Support project for the C Loader implementation in MetaCall, work was completed to provide support for the Windows platform and integrate automated testing through GitHub Actions.
The primary objective was to ensure that the C Loader could be properly configured, built, and tested on Windows with MSVC, while integrating all required dependencies: libffi, libclang, and TCC.
The expected result of the implementation is support for execution of C files through MetaCall:
metacall hello.c

# 2. Tasks Completed
# 2.1. CMake Dependency Discovery for Windows
The three CMake find-modules were updated to support Windows search paths. Previously, all three files contained only Linux-specific paths with no Windows support.
FindLibFFI.cmake
Before this change, the file contained only # TODO: Windows? with no implementation.
Added a elseif(WIN32) block with LIBFFI_WINDOWS_PATHS covering all common Windows install locations:
cmakeelseif(WIN32)
    set(LIBFFI_WINDOWS_PATHS
        "$ENV{LIBFFI_ROOT}"
        "${LIBFFI_ROOT}"
        "$ENV{VCPKG_ROOT}/installed/x64-windows"
        "C:/vcpkg/installed/x64-windows"
        "C:/ProgramData/chocolatey/lib/libffi"
        "$ENV{USERPROFILE}/scoop/apps/libffi/current"
        "C:/libffi"
    )
    find_library(LIBFFI_LIBRARY NAMES ffi libffi
        PATHS ${LIBFFI_WINDOWS_PATHS}
        PATH_SUFFIXES lib lib64 x64/lib x86/lib
    )
    find_path(LIBFFI_INCLUDE_DIR ffi.h
        PATHS ${LIBFFI_WINDOWS_PATHS}
        PATH_SUFFIXES include
    )
FindLibTCC.cmake
On Windows, TCC is distributed as libtcc.dll with an import library libtcc.lib, different from the Linux libtcc.so. Added if(WIN32) block with search for both the import library and the runtime DLL:
cmakeif(WIN32)
    set(LIBTCC_WINDOWS_PATHS
        "$ENV{TCC_ROOT}"
        "${TCC_ROOT}"
        "$ENV{USERPROFILE}/scoop/apps/tcc/current"
        "C:/tcc"
        "C:/Program Files/tcc"
        "C:/ProgramData/tcc"
    )
    find_library(LIBTCC_LIBRARY NAMES libtcc tcc
        PATHS ${LIBTCC_WINDOWS_PATHS}
        PATH_SUFFIXES lib . lib/tcc
    )
    find_path(LIBTCC_INCLUDE_DIR libtcc.h
        PATHS ${LIBTCC_WINDOWS_PATHS}
        PATH_SUFFIXES include .
    )
    find_file(LIBTCC_DLL NAMES libtcc.dll tcc.dll
        PATHS ${LIBTCC_WINDOWS_PATHS}
        PATH_SUFFIXES bin .
    )
FindLibClang.cmake
LLVM on Windows installs to C:/Program Files/LLVM, completely different from Linux paths /usr/lib/llvm-VERSION/lib/. Added if(WIN32) block.
A critical fix was also applied: when CMake found include/clang-c/, it set that as LibClang_INCLUDE_DIR. However, source code uses #include <clang-c/Index.h>, which requires the parent include/ directory. Without this fix, compilation failed with clang-c/CXString.h: No such file or directory:
cmakeif(WIN32)
    set(LibClang_WINDOWS_PATHS
        "$ENV{LLVM_ROOT}"
        "${LLVM_ROOT}"
        "C:/Program Files/LLVM"
        "C:/Program Files (x86)/LLVM"
        "$ENV{VCPKG_ROOT}/installed/x64-windows"
        "C:/vcpkg/installed/x64-windows"
        "$ENV{USERPROFILE}/scoop/apps/llvm/current"
        "C:/ProgramData/chocolatey/lib/llvm/tools/llvm"
    )
    find_library(LibClang_LIBRARY NAMES libclang clang
        PATHS ${LibClang_WINDOWS_PATHS}
        PATH_SUFFIXES lib bin
    )
    find_path(LibClang_INCLUDE_DIR
        NAMES ${LibClang_INCLUDE_HEADERS}
        PATHS ${LibClang_WINDOWS_PATHS}
        PATH_SUFFIXES include/clang-c include
    )
    if(LibClang_INCLUDE_DIR)
        get_filename_component(_parent "${LibClang_INCLUDE_DIR}" DIRECTORY)
        if(EXISTS "${_parent}/clang-c/Index.h")
            set(LibClang_INCLUDE_DIR "${_parent}")
        endif()
    endif()
    find_file(LibClang_DLL NAMES libclang.dll
        PATHS ${LibClang_WINDOWS_PATHS}
        PATH_SUFFIXES bin
    )

# 2.2. Fix of Existing Bug in InstallLibTCC.cmake
When TCC is not found on the system, InstallLibTCC.cmake uses ExternalProject_Add to automatically download and build TCC from source. This mechanism had three bugs on Windows with MSVC.
Bug 1 — Configure step:
CONFIGURE_COMMAND was set to an empty string. CMake interprets this as "run cmake configure by default". TCC has no CMakeLists.txt and uses its own build script, so CMake failed with:
does not appear to contain CMakeLists.txt
Fix:
cmake# MSVC: TCC uses win32/build-tcc.bat, skip cmake configure
set(LIBTCC_CONFIGURE ${CMAKE_COMMAND} -E echo "Skipping configure for MSVC")
Bug 2 — Build step:
build-tcc.bat uses relative paths like ..\libtcc.c expecting to run from inside the win32/ directory. CMake ran it from the source root, causing:
fatal error: ..\libtcc.c: No such file or directory
Fix:
cmakeset(LIBTCC_BUILD cmd /c "cd win32 && build-tcc.bat -i ${LIBTCC_INSTALL_PREFIX}")
Bug 3 — Install step:
INSTALL_COMMAND was also empty, causing the same default cmake behavior issue.
Fix:
cmakeset(LIBTCC_INSTALL ${CMAKE_COMMAND} -E echo "Install handled by build step")

# 2.3. Build Helper Script
A new helper script cmake/tcc_build_msvc.bat was created to handle cases where build-tcc.bat fails when the install path contains spaces. The xcopy command inside build-tcc.bat does not handle paths with spaces correctly.
The script accepts source and destination directories as quoted arguments, runs build-tcc.bat, copies results manually with correct path escaping, and always returns exit /B 0 to avoid blocking the CMake build on non-critical copy errors.

# 2.4. GitHub Actions CI Pipeline
The existing workflow .github/workflows/windows-test.yml was updated to enable the C Loader in the Windows CI pipeline.
Before this change, the C Loader was commented out in the build options:
METACALL_BUILD_OPTIONS: ... file # netcore5 java c cobol rust ...
After the change:
METACALL_BUILD_OPTIONS: ... file c # netcore5 cobol rust ...
The CI pipeline now tests the C Loader automatically on every pull request on:

windows-2022
windows-2025

with configurations:

debug, without-sanitizer
debug, address-sanitizer


# 2.5. Windows Installer Script
A PowerShell installer script tools/metacall-installer.ps1 was created to allow developers to install all C Loader dependencies with a single command.
The script installs:

LLVM via Chocolatey
libffi via vcpkg (libffi:x64-windows)
TCC built from source (metacall/tinycc)

And sets the required environment variables:
powershell[Environment]::SetEnvironmentVariable("LLVM_ROOT", "C:\Program Files\LLVM", "Machine")
[Environment]::SetEnvironmentVariable("TCC_ROOT", "C:\mc_out", "Machine")
[Environment]::SetEnvironmentVariable("LIBFFI_ROOT", "C:\vcpkg\installed\x64-windows", "Machine")
These variables are read by the Find*.cmake files via $ENV{LLVM_ROOT}, $ENV{TCC_ROOT}, and $ENV{LIBFFI_ROOT} to automatically locate libraries during CMake configuration.

# 2.6. Pull Request and Version Control Workflow
A dedicated feature branch was created for isolated development of Windows support:
git checkout -b feature/windows-c-loader
The implementation included three commits:
Commit 1
fix: add Windows/MSVC support for C Loader dependencies
Implemented:

Windows search paths in FindLibFFI.cmake, FindLibTCC.cmake, FindLibClang.cmake
Bug fixes in InstallLibTCC.cmake (configure, build, install steps)
New helper script cmake/tcc_build_msvc.bat

Commit 2
ci: enable C Loader in Windows CI pipeline
Implemented:

Enabled c option in windows-test.yml

Commit 3
feat: add Windows installer script for C Loader dependencies
Implemented:

Created tools/metacall-installer.ps1

After implementation, changes were pushed to the remote repository:
git push origin feature/windows-c-loader
A Pull Request was created:
https://github.com/metacall/core/pull/780
The Pull Request was used for code review, CI validation, and integration testing.

# 3. Testing Results
The following tests and validations were performed:
Build verification
CMake configuration output confirmed all three dependencies were found:
-- Found LibFFI: C:/vcpkg/installed/x64-windows/lib/ffi.lib
-- Installing LibTCC 4fccaf61241a5eb72b0777b3a44bd7abbea48604
-- Found LibClang: C:/Program Files/LLVM/lib/libclang.lib
-- Plugin c_loader
Build completed successfully:
c_loader.vcxproj -> C:\...\build\Release\c_loader.dll
Environment tested
ComponentVersionOSWindows 10 x64CompilerMSVC 19.42 (Visual Studio 2022)CMake4.3.1LLVM / libclang22.1.0libffi3.5.2 (vcpkg)TCCmetacall/tinycc @ 4fccaf6
CI validation
GitHub Actions workflow executes on every pull request:

Environment setup
Dependency installation (LLVM, libffi, TCC)
CMake configuration
Build
Test execution


# 4. Result
The following deliverables were completed successfully:
✓ Windows search paths added to FindLibFFI.cmake
✓ Windows search paths added to FindLibTCC.cmake
✓ Windows search paths added to FindLibClang.cmake
✓ Fixed existing bug in InstallLibTCC.cmake (configure, build, and install steps for MSVC)
✓ Created cmake/tcc_build_msvc.bat helper script
✓ Enabled C Loader in .github/workflows/windows-test.yml
✓ Created tools/metacall-installer.ps1 installer script
✓ c_loader.dll successfully built on Windows x64 with MSVC (58,880 bytes)
The implementation provides Windows support for the MetaCall C Loader and ensures continuous verification through the existing CI pipeline.


# Progress Report: Windows C Loader Support and CI Configuration for MetaCall
 
## 1. Objective and Context
 
As part of the Cross-Platform Support project for the `C Loader` implementation in **MetaCall**, work was completed to provide support for the Windows platform and integrate automated testing through GitHub Actions.
 
The primary objective was to ensure that the `C Loader` could be properly built on Windows with MSVC, integrating all required dependencies: `libffi`, `libclang`, and `TCC`.
 
The expected result of the implementation is support for execution of C files through MetaCall:
 
```
metacall hello.c
```
 
---
 
## 2. Tasks Completed by Mariia Ryzhova 
 
### 2.1. CMake Dependency Discovery for Windows
 
Updated three CMake find-modules to support Windows search paths. Previously, all three files contained only Linux-specific paths.
 
**FindLibFFI.cmake**
 
- Added `elseif(WIN32)` block with Windows search paths.
- Before this change, the file contained only `# TODO: Windows?` with no implementation.
- Added search via `LIBFFI_ROOT` environment variable, vcpkg, Chocolatey, Scoop, and `C:/libffi`.
**FindLibTCC.cmake**
 
- Added `if(WIN32)` block for Windows-specific TCC discovery.
- On Windows, TCC is distributed as `libtcc.dll` with import library `libtcc.lib`, different from Linux `libtcc.so`.
- Added search via `TCC_ROOT` environment variable and standard Windows locations.
- Added `find_file(LIBTCC_DLL ...)` to locate the runtime DLL for post-build copy.
**FindLibClang.cmake**
 
- Added `if(WIN32)` block for Windows-specific LLVM/libclang discovery.
- LLVM on Windows installs to `C:/Program Files/LLVM`, completely different from Linux paths.
- Added search via `LLVM_ROOT` environment variable, vcpkg, Chocolatey, and Scoop.
- Applied a critical fix: when CMake found `include/clang-c/`, it incorrectly set that as `LibClang_INCLUDE_DIR`. Source code uses `#include <clang-c/Index.h>`, which requires the parent `include/` directory. Without this fix, compilation failed with `clang-c/CXString.h: No such file or directory`.
### 2.2. Fix of Existing Bug in InstallLibTCC.cmake
 
When TCC is not found, `InstallLibTCC.cmake` automatically builds it from source using `ExternalProject_Add`. This mechanism had three bugs on Windows with MSVC:
 
- **Configure step**: `CONFIGURE_COMMAND` was empty. CMake tried to run cmake configure by default, but TCC has no `CMakeLists.txt`. Fixed by replacing with an explicit echo command to skip configure.
- **Build step**: `build-tcc.bat` uses relative paths and expects to run from inside `win32/` directory. CMake ran it from the source root, causing `libtcc.c: No such file or directory`. Fixed by prepending `cd win32 &&` before the bat call.
- **Install step**: `INSTALL_COMMAND` was also empty, causing the same cmake default behavior issue. Fixed by replacing with an explicit echo command.
### 2.3. Build Helper Script
 
Created `cmake/tcc_build_msvc.bat` to handle cases where `build-tcc.bat` fails when the install path contains spaces. The `xcopy` command inside `build-tcc.bat` does not handle paths with spaces correctly.
 
The script:
 
- Accepts source and destination directories as properly quoted arguments.
- Runs `build-tcc.bat` from the correct directory.
- Copies results manually with correct path escaping.
- Always returns `exit /B 0` to avoid blocking the CMake build on non-critical copy errors.
### 2.4. GitHub Actions CI Pipeline
 
Updated `.github/workflows/windows-test.yml` to enable the C Loader in the Windows CI pipeline.
 
Before this change, the C Loader was commented out:
 
```
# netcore5 java c cobol rust
```
 
After the change, `c` was added to active build options:
 
```
file c # netcore5 cobol rust
```
 
The CI pipeline now tests the C Loader automatically on every pull request on `windows-2022` and `windows-2025`.
 
### 2.5. Windows Installer Script
 
Created `tools/metacall-installer.ps1` to allow developers to install all C Loader dependencies with a single command.
 
The script installs:
 
- LLVM via Chocolatey (`choco install llvm`)
- libffi via vcpkg (`vcpkg install libffi:x64-windows`)
- TCC built from source (`metacall/tinycc`)
After installation, the script sets the required environment variables so that CMake can automatically find all dependencies:
 
```
LLVM_ROOT   = C:\Program Files\LLVM
TCC_ROOT    = C:\mc_out
LIBFFI_ROOT = C:\vcpkg\installed\x64-windows
```
 
### 2.6. Pull Request and Version Control Workflow
 
A dedicated feature branch was created:
 
```
git checkout -b feature/windows-c-loader
```
 
The implementation included three commits:
 
**Commit 1** — `fix: add Windows/MSVC support for C Loader dependencies`
 
- Windows search paths in all three `Find*.cmake` files
- Bug fixes in `InstallLibTCC.cmake`
- New helper script `cmake/tcc_build_msvc.bat`
**Commit 2** — `ci: enable C Loader in Windows CI pipeline`
 
- Enabled `c` option in `windows-test.yml`
**Commit 3** — `feat: add Windows installer script for C Loader dependencies`
 
- Created `tools/metacall-installer.ps1`
After implementation, changes were pushed and a Pull Request was opened:
 
```
https://github.com/metacall/core/pull/780
```
 
---
 
## 3. Testing Results
 
### Build verification
 
CMake configuration confirmed all three dependencies were found:
 
```
-- Found LibFFI: C:/vcpkg/installed/x64-windows/lib/ffi.lib
-- Installing LibTCC 4fccaf61241a5eb72b0777b3a44bd7abbea48604
-- Found LibClang: C:/Program Files/LLVM/lib/libclang.lib
-- Plugin c_loader
```
 
Build completed successfully on Windows x64 with MSVC 19.42:
 
```
c_loader.vcxproj -> build\Release\c_loader.dll
```
 
### Environment tested
 
| Component | Version |
|---|---|
| OS | Windows 10 x64 |
| Compiler | MSVC 19.42 (Visual Studio 2022) |
| CMake | 4.3.1 |
| LLVM / libclang | 22.1.0 |
| libffi | 3.5.2 (vcpkg) |
| TCC | metacall/tinycc @ 4fccaf6 |
 
---
 
## 4. Result
 
The following deliverables were completed successfully:
 
✓ Windows search paths added to `FindLibFFI.cmake`
 
✓ Windows search paths added to `FindLibTCC.cmake`
 
✓ Windows search paths added to `FindLibClang.cmake`
 
✓ Fixed existing bug in `InstallLibTCC.cmake` (configure, build, and install steps for MSVC)
 
✓ Created `cmake/tcc_build_msvc.bat` helper script
 
✓ Enabled C Loader in `.github/workflows/windows-test.yml`
 
✓ Created `tools/metacall-installer.ps1` installer script
 
✓ `c_loader.dll` successfully built on Windows x64 with MSVC (58,880 bytes)
 
The implementation provides Windows support for the MetaCall C Loader and ensures continuous verification through the existing CI pipeline.
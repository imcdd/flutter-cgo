# Write Flutter plugin in cgo with cmake
1. Create new flutter plugin with --platforms=android,windows --template=plugin_ffi
2. Delete src/\*.c and src/\*.h
3. Delete src/CMakeLists.txt and copy flutter_cgo/CMakeLists.txt to src
4. Cd src，execute go init mod and create go c-shared project
5. Add custom targets to android/build.gradle
    ```
    defaultConfig {
        externalNativeBuild {
            cmake {
                targets "shared_go_lib"
            }
        }
    }
    ```
6. Open the project and debug

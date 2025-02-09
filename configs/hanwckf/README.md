# OpenWRT编译工具链

# 说明

自定义GCC版本

# 版本

- 7.5.0
- 8.4.0
- 9.3.0
- 10.3.0
- 11.2.0
- 11.3.0
- 12.3.0
- 13.3.0

默认选择为的GCC 11.2.0版本

# 修改默认版本

```shell
sed -i 's/default "11.2.0"/default "11.3.0"/' MyConfig/configs/hanwckf/toolchain/gcc/Config.version
```

# 问题

#### `uhttpd` 包在编译过程中遇到了代码问题

> error: 'o' may be used uninitialized [-Werror=maybe-uninitialized]
>
> ...
>
> ERROR: package/network/services/uhttpd failed to build.

原因：`uhttpd` 包在编译过程中遇到了代码问题，具体是 `ubus.c` 文件中的变量 `o` 可能未初始化就被使用，导致编译器报错并终止编译

解决：

1. 给变量初始化

- 手动修改

```c
//初始化后
void *t, *o;
//初始化前
void *t = NULL, *o = NULL;
```

2. 禁用 `-Werror` 选项

- 在CMakeLists.txt文件中去掉`-Werror` 选项

```cmake
#去掉前
ADD_DEFINITIONS(-D_FILE_OFFSET_BITS=64 -Os -Wall -Werror -Wmissing-declarations --std=gnu99 -g3)
#去掉后
ADD_DEFINITIONS(-D_FILE_OFFSET_BITS=64 -Os -Wall -Wmissing-declarations --std=gnu99 -g3)
```

3. 忽略特定文件的警告

- 在 `ADD_LIBRARY(uhttpd_ubus MODULE ubus.c)` 之后添加

```cmake
SET_SOURCE_FILES_PROPERTIES(ubus.c PROPERTIES COMPILE_FLAGS -Wno-error)
```

4. 升级`uhttpd` 包

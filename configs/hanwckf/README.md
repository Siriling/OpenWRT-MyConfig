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
- 12.2.0
- 13.3.0

默认选择为的GCC 11.2.0版本

# 修改默认版本

```shell
sed -i 's/default "11.2.0"/default "11.3.0"/' MyConfig/configs/hanwckf/toolchain/gcc/Config.version
```

# 问题

### `uhttpd` 包在编译过程中出错

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

### `elfutils ` 包在编译过程中出错

> ebl_syscall_abi.c:37:64: error: argument 5 of type 'int *' declared as a pointer [-Werror=array-parameter=]
>
> ...
>
> ERROR: package/libs/elfutils failed to build.

原因：

- 函数声明与定义不一致，可能是源码或补丁的问题

- 在 `ebl_syscall_abi.c` 文件中，函数 `ebl_syscall_abi` 的第 5 个参数被声明为 `int *`（指针），但在头文件 `libebl.h` 中，该参数被声明为 `int args[6]`（数组）。
- 由于编译器选项 `-Werror` 的存在，所有警告都被视为错误，导致编译失败。

解决：

1. 给变量初始化

- 手动修改`ebl_syscall_abi.c`文件的函数声明

```c
ebl_syscall_abi (Ebl *ebl, int *sp, int *pc, int *callno, int args[6])
```

2. 禁用 `-Werror` 选项

- 在Makefile.txt文件中添加

```makefile
ifneq ($(filter $(GCC_MAJOR_VERSION),11 12 13),)
TARGET_CFLAGS += \
	-Wno-error=use-after-free
endif
```

3. 升级`elfutils` 包

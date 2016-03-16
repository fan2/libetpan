
使用 Xcode 打开 `libetpan/build-mac/libetpan.xcworkspace`，其中包含：

- libetpan.xcodeproj：静态库项目
- libetpan Tests.xcodeproj：测试项目

先启动编译 libetpan.xcodeproj / static libetpan，再启动编译 libetpan Tests.xcodeproj 中的任一 demo（例如 smtpsend/pop-sample/imap-sample）。

## libetpan.xcodeproj
### targets
Framework Version 都为 `A` 。

target          | type             | Packaing
----------------|------------------|----------
libetpan        | Cocoa Framework  | Wrapper Extension 为 `framework`
static libetpan | Library          | 
libetpan ios    | Library          | 

### Build Settings
#### Architectures
Base SDK & Supported Platforms

- `libetpan` & `static libetpan`：**OS X**，依赖 External Build System 项目 `libetpan-prepare`
- `libetpan ios`：**iOS**，依赖 External Build System 项目 `libetpan-prepare-ios`

#### Search Paths
Header Search Paths

Search Path        | expand
-------------------|--------------
$(SRCROOT)/..      | libetpan/build-mac/..
$(SRCROOT)/include | libetpan/build-mac/include

1. `libetpan/build-mac/..` 返回到 `libetpan/`，其下包括：

	- config.h
	- libetpan-config.h

2. `libetpan/build-mac/include` ？

### Dependencies
#### cyrus-sasl
[SASL](http://asg.web.cmu.edu/sasl/) is the Simple Authentication and Security Layer
[Project Cyrus](http://asg.web.cmu.edu/cyrus)，[Do it Yourself From GIT](http://cyrusimap.org/imap/installation/diy.html#from-git)。

libetpan 项目依赖 cyrus-sasl，其代码目录：  
> libetpan/build-mac/dependencies/packages/cyrus-sasl-2.1.25.tar.gz

编译 `libetpan ios` 项目时，首先 Run external build tool `libetpan-prepare-ios`，执行 `prepare-cyrus-sasl.sh` 脚本。

```Shell
# 定义变量
ARCHIVE_NAME=cyrus-sasl-2.1.25.tar.gz
url=ftp://ftp.andrew.cmu.edu/pub/cyrus-mail/

# curl 下载源码（-O: Write output to a file named as the remote file）
curl -O "$url"

# 解压源代码到当前文件夹
tar -xzf "$current_dir/packages/$ARCHIVE_NAME"

# 打 patch
cd "$srcdir/$ARCHIVE"
patchfile=cyrus-2.1.25-libetpan.patch
patch -p1 < $current_dir/$patchfile

# 运行配置 libetpan/build-mac/dependencies/packages/cyrus-sasl-2.1.25/configure 
./configure

# 针对 TARGETS 和 MARCHS 两层 for 循环编译静态库
        cd lib
        make install >> "$logfile" 2>&1

# version=2.1.25
# 压缩到 libetpan/build-mac/dependencies/build/libsasl/builds/libsasl-$version-ios.tar.gz
cd "$BUILD_DIR"
mkdir -p libsasl-ios
cp -r "$INSTALL_PATH"/* libsasl-ios/
tar -czf "libsasl-$version-ios.tar.gz" libsasl-ios

# 解压到 libetpan/build-mac/libsasl-ios 
cd "$scriptdir/.."
tar xzf "$resultdir/libsasl-$version-ios.tar.gz"
```

cyrus-sasl 项目依赖 [openssl](https://github.com/openssl/openssl) 项目，解压 `cyrus-sasl-2.1.25.tar.gz` 可以看到  
> libetpan/build-mac/dependencies/packages/cyrus-sasl-2.1.25/cmulocal/libssl.m4

### Build Phases
#### Run Script
Shell ： `/bin/sh`

```
mkdir -p "$BUILT_PRODUCTS_DIR/include"
rsync -aL "$SRCROOT/../include/libetpan/" "$BUILT_PRODUCTS_DIR/include/libetpan/"
```

1. `$BUILT_PRODUCTS_DIR` 展开为： 
`libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug`  
在其下创建目录 `include`。

	**mkdir**
	
	```
	-p, --parents
	
	Create intermediate directories as required.
	If this option is not specified, the full path prefix of each operand must already exist.
	```

2. 将 `/libetpan/include/libetpan` 同步复制到发布到 `libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug/include/libetpan/` 目录。

	[**rsync**](http://www.ibm.com/developerworks/cn/aix/library/au-spunix_rsync/)
	
	```
	-a, --archive               archive mode; same as -rlptgoD (no -H)
	-L, --copy-links            transform symlink into referent file/dir
	```

## libetpan Tests.xcodeproj
### libetpan.a
External Frameworks and Libraries：`libetpan.a`。  
`Build Phases | Link Binaries With Libraries` 中包含 `libetpan.a`。  

需要先启动编译 `libetpan.xcodeproj` 的 target： static libetpan。  
`libetpan.xcodeproj` 的 Run Script 会拷贝 **`libetpan.a`** 到 `libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug` 目录。

### Search Paths

**smtpsend** 的 `Build Settings | Search Paths` 为  `"$(SRCROOT)/../../include"`，`libetpan/build-mac/../../include` 根本不存在？  
还是针对 `libetpan/tests/` 下的 `smtpsend.c` / `pop-sample.c` / `imap-sample.c` 展开为 `libetpan/include` ？

### Tests

可编译调试 Mac OS X 下的命令行demo：`smtpsend` / `pop-sample` / `imap-sample`

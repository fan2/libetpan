先启动编译 libetpan.xcodeproj / static libetpan，再启动编译 libetpan Tests.xcodeproj 中的任一 demo（例如 smtpsend/pop-sample/imap-sample）。

可在 Mac OS X 下编译调试命令行demo：`smtpsend` / `pop-sample` / `imap-sample`

## smtpsend
### usage

**usage**: `smtpsend [-f from] [-u user] [-v password] [-s server] [-p port] [-S] [-L] <rcpts>...`

### Launch Arguments

```
/* globals */
char *smtp_server;
uint16_t smtp_port = 25;
char *smtp_user;
char *smtp_password;
char *smtp_from;
int smtp_tls = 0;
int smtp_esmtp = 1;
int smtp_ssl = 0;
```

    case 's':smtp_server
    case 'p':smtp_port
    case 'u':smtp_user
    case 'v':smtp_password
    case 'f':smtp_from
    case 'S':smtp_tls（boolean switch）
    case 'E':smtp_esmtp（reverse boolean switch）
    case 'L':smtp_ssl（boolean switch）

[option（getopt,optind,optarg）的用法](http://blog.sina.com.cn/s/blog_9c8c37110102uyoq.html)

## pop-sample
### syntax
**syntax**: `pop-sample [gmail-email-address] [gmail-password]`

1. server 为 gmail："pop.gmail.com"

2. mailimap_login 需要参数：

	- argv[0]：Unix executable 程序名
	- argv[1]：mailpop3_user（gmail 账户）
	- argv[2]：mailpop3_pass（gmail 密码）

### Launch Arguments
有以下两种带参数启动运行方式：

1. Product | Scheme | Arguments

	`Arguments Passed On Launch` 添加参数，然后再启动运行。

2. 编译完成后，终端运行

	```
	cd libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug
	
	./pop-sample user pass
	```

## imap-sample
### usage
**usage**: `imap-sample [gmail-email-address] [password]`

1. server 为 gmail："imap.gmail.com"

2. mailimap_login 需要参数：

	- argv[0]：Unix executable 程序名
	- argv[1]：userid（gmail 账户）
	- argv[2]：password（gmail 密码）

### Launch Arguments
参考 pop-sample，不再赘述。

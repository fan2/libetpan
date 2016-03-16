先启动编译 libetpan.xcodeproj / static libetpan，再启动编译 libetpan Tests.xcodeproj 中的任一 demo（例如 smtpsend/pop-sample/imap-sample）。

可在 Mac OS X 下编译调试命令行demo：`smtpsend` / `pop-sample` / `imap-sample`

## 邮箱地址
[163 免费邮客户端设置的 POP3、SMTP、IMAP 地址](http://help.163.com/09/1223/14/5R7P3QI100753VB8.html)
[Gmail，QMail，163邮箱的 IMAP/SMTP/POP3 地址](http://blog.wpjam.com/m/gmail-qmail-163mail-imap-smtp-pop3/)

本人测试用例使用 163 邮箱。

## smtpsend
### call flow

> int main(int argc, char **argv)
>> int getopt(int, char * const [], const char *)
>> int collect(struct mem_message *message)
>>>  read(STDIN_FILENO)
>>
>> int send_message(char *data, size_t len, char**rcpts)

### usage

**usage**: `smtpsend [-f from] [-u user] [-v password] [-s server] [-p port] [-S] [-L] <rcpts>...`

有以下两种带参数（Launch Arguments）启动运行方式：

1. Product | Scheme | [Arguments](http://blog.csdn.net/siemenliu/article/details/7891293)

	> `Arguments Passed On Launch` 添加参数，然后再启动运行。

2. 编译完成后，终端运行

	> 终端 cd 进入 `/libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug`，执行 `./smtpsend`。
	
### getopt

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

通过 **getopt** 获取解析参数列表（char* argv[]），可参考：

> [option（getopt,optind,optarg）的用法](http://blog.sina.com.cn/s/blog_9c8c37110102uyoq.html)  
> [Linux命令行参数编程](http://my.oschina.net/jacobin/blog/159719)

### collect & send_message
选择 smtpsend 这个 target，编译成功后，终端 cd 进入 `/libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug`，执行 `./smtpsend`：

1. 其中 `your163pass` 为**客户端授权独立密码**。
2. 从 mail.txt 文件读取邮件正文：**collect** message by file [read](http://blog.csdn.net/xiaoxi2xin/article/details/5524769) from [STDIN_FILENO](http://www.powerxing.com/unix-write-async-vs-sync/)。

	以下为 mail.txt 文件内容：
	
	```
	subject:<test>
	from:<phunxm@163.com>
	to:<2306328146@qq.com>
	
	This is a test from telnet@163.com
	
	.
	
	```

3. 最终调用 **send_message** 函数发送邮件。控制台打印 `Sent ok.` 表明发送成功。

### test
#### 普通连接（非 SSL）
mailsmtp_socket_connect

```
faner@MBP-FAN:~/Projects/git/libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug|master⚡
⇒  ./smtpsend -f phunxm@163.com -u phunxm@163.com -v your163pass -s smtp.163.com -p 25 2306328146@qq.com < mail.txt 
Sent ok.
```

#### 安全连接（SSL）
mailsmtp_ssl_connect

```
⇒  ./smtpsend -f phunxm@163.com -u phunxm@163.com -v your163pass -s smtp.163.com -p 465 -L 2306328146@qq.com < mail.txt 
Sent ok.
```

**注意**：163 邮箱 SMTP 服务不支持 TLS。若开启 `-S` （smtp_tls），则 `mailsmtp_starttls` 将报错：

```
mailsmtp_starttls: TLS not available on server for temporary reason
```

## pop-sample
### call flow

> int main(int argc, char **argv)
>> pop3 = mailpop3_new(0, NULL);
>> mailpop3_ssl_connect(pop3, "pop.gmail.com", 995);
>> mailpop3_user(pop3, argv[1]);
>> mailpop3_pass(pop3, argv[2]);
>> mailpop3_list(pop3, &list);
>> mailpop3_quit(pop3);
>> mailpop3_free(pop3);

### syntax
**syntax**: `pop-sample [gmail-email-address] [gmail-password]`

1. server 为 gmail："pop.gmail.com"

2. mailimap_login 需要参数：

	- argv[0]：Unix executable 程序名
	- argv[1]：mailpop3_user（gmail 账户）
	- argv[2]：mailpop3_pass（gmail 密码）

### test
修改邮箱服务器为 163：

```
r = mailpop3_ssl_connect(pop3, "pop.163.com", 995); // "pop.gmail.com"
```

编译成功后，在终端命令行输入 `./pop-sample` 下载 Inbox 中的所有邮件。

其中 `your163pass` 为**客户端授权独立密码**。

```
⇒  ./pop-sample phunxm@163.com your163pass
fetched 1 xtbBRwdBuVO-r8HOrgAAsa
fetched 2 1tbiXARBuVEAK+Yh4AAAs9
fetched 3 1tbiXQhBuVEAK2MriQAAsW
fetched 4 1tbiQglBuVEAMUu4DwAAsv
fetched 5 1tbiXARBuVEAK+Yh4AABs8
fetched 6 xtbBzh5WuVQGzsG+-QAAsh
fetched 7 xtbBzh5WuVQGzsG+-QABsg
fetched 8 1tbiXAXJuVXlcUyufgAAs0
fetched 9 xtbBdgoBFlUL6TevcAAAb0
fetched 10 1tbiGh0euVaDnhvt9wAAsq
fetched 11 1tbiGh0euVaDnhvt9wABsr
fetched 12 1tbiygMuuVQG2AyizAAAs2
```

所有的邮件下载到目录 download 中（`mkdir("download", 0700);`），下载的 [eml](http://fileinfo.com/extension/eml) 格式的邮件，可使用 Microsoft Outlook Express、Apple Mail 或 Foxmail 等邮件客户端打开阅读。

## imap-sample
### call flow

> int main(int argc, char **argv)
>> imap = mailimap_new(0, NULL);
>> mailimap_ssl_connect(imap, "imap.gmail.com", 993);
>> mailimap_login(imap, argv[1], argv[2]);
>> mailimap_select(imap, "INBOX");
>> fetch_messages(imap);
>> mailimap_logout(imap);
>> mailimap_free(imap);

### usage
**usage**: `imap-sample [gmail-email-address] [password]`

1. server 为 gmail："imap.gmail.com"

2. mailimap_login 需要参数：

	- argv[0]：Unix executable 程序名
	- argv[1]：userid（gmail 账户）
	- argv[2]：password（gmail 密码）

### test
修改邮箱服务器为 163：

```
r = mailimap_ssl_connect(imap, "imap.163.com", 993); // "imap.gmail.com"
```

编译成功后，在终端命令行输入 `./imap-sample` 下载 Inbox 中最近30天的邮件。

其中 `your163pass` 为**客户端授权独立密码**。

```
faner@MBP-FAN:~/Projects/git/libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug|master⚡
⇒  ./imap-sample phunxm@163.com your163pass
connect: 2
fetch 1415701667
1415701667 has been fetched
fetch 1415701668
1415701668 has been fetched
```

最近30天的邮件下载到目录 download 中（`mkdir` - `fopen` - `fwrite` - `fclose`），执行 `ls download` 可看到下载了2封邮件。

```
faner@MBP-FAN:~/Projects/git/libetpan/build-mac/DerivedData/libetpan/Build/Products/Debug|master⚡
⇒  ls download
1415701667.eml 1415701668.eml
```


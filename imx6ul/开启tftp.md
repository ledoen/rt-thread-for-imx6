# 开启及使用tftp

## 一、按照文档操作

TFTP 工具依赖 lwIP，需要先在 Env 工具 开启 lwIP 的依赖才可见，步骤如下：

```c
-> RT-Thread Components
  -> Network stack
    -> light weight TCP/IP stack
      -> Enable lwIP stack
```

在 NetUtils 菜单栏使能 TFTP 选项：

```c
RT-Thread online packages
  -> IoT - internet of things
    -> netutils: Networking utilities for RT-Thread
    [*] Enable TFTP(Trivial File Transfer Protocol) server
```

## 二、修正bug

进行了以上操作并不能正常使用tftp，需要进行一下操作

### 2.1添加文件编译

修改kernel/components/net/lwip-2.1.2/SConscript，在最后添加以下代码：

```c
# 14. TFTP server files
lwiptftp_SRCS = Split("""
src/apps/tftp/tftp_server.c
src/apps/tftp/tftp_port.c
""")


src += lwiptftp_SRCS	/* 添加代码 */
```

## 2.2修改指令输出

修改kernel/components/net/lwip-2.1.2/src/apps/tftp/tftp_port.c，修改为以下代码

```c
static void tftp_server(uint8_t argc, char **argv)
{
    ctx.open = tftp_open;
    ctx.close = (void (*)(void *)) close;
    ctx.read = (int (*)(void *, void *, int)) read;
    ctx.write = tftp_write;

    if (tftp_init(&ctx) == ERR_OK)
    {
        rt_kprintf("TFTP server start successfully.\n");
    }
    else
    {
        rt_kprintf("TFTP server start failed.\n");
    }
}
MSH_CMD_EXPORT(tftp_server, start tftp server.)		/*此处为修改代码，注意结尾不要分号*/
```



## 三、使用tftp

参考https://blog.csdn.net/cunjiu9486/article/details/109074189

虚拟机作为客户端：

连接：tftp 192.168.50.225

发送：put ledoen.txt

拉取：get ledoen.txt

如果不能正常工作，则使用help进行不同的设置尝试

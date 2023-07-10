---
layout: post
category : Tech
title : Java 如何监听文件变化？
tags : [java]
excerpt: 日常开发中经常需要监听某个本地文件夹内部是否有文件创建或删除等，本文介绍两种方式：WatchService 和 FileAlterationObserver
---
{% include JB/setup %}

日常开发中经常需要监听某个本地文件夹内部是否有文件创建或删除等，本文介绍两种方式：

* JDK 内置的 `java.nio.file.WatchService`
* “commons-io” 中的 `org.apache.commons.io.monitor.FileAlterationObserver`

### WatchService

从 JDK7 开始，JDK 内置了 `WatchService`，用于监听文件创建、删除等事件，具体可参考 `java.nio.file.StandardWatchEventKinds`，注意 `OVERFLOW` 类型，当文件过多时可能时触发，下文会详细解释。

监听 `/tmp` 目录下的文件创建事件代码示例：

```java
var path = Paths.get("/tmp");
try (WatchService watcher = FileSystems.getDefault().newWatchService()) {
    // 注册监听 path 文件夹下的文件创建事件
    path.register(watcher, ENTRY_CREATE);
    for (;;) {
        // 获取注册后的监听对象
        WatchKey key = watcher.take();

        // 拉取事件
        for (WatchEvent<?> event: key.pollEvents()) {
            // 打印文件夹内的文件相对路径
            System.out.println(event.context());
        }

        // 重置监听对象，如果无效则表示监听对象失效
        boolean valid = key.reset();
    }
} catch (Exception ex) {
    ...
}
```

因为内部实现的原因会限制文件的数量，大量文件创建且消费较慢时会触发 `OVERFLOW` 事件，导致返回的文件为 NULL，因此少量文件时推荐使用，且性能更好。

为什么文件事件过多时会发生 `OVERFLOW` 呢？因为文件系统内部实现依赖本地实现，Linux 和 Windows 有所不同。

##### LinuxWatchService

当文件过多时返回事件变为 `StandardWatchEventKinds.OVERFLOW`，即操作系统返回的 `inotify_event` 的 `mask` 为 `IN_Q_OVERFLOW` 和  `IN_IGNORED`。JDK 默认的缓冲区大小为 `BUFFER_SIZE = 8192`，缓冲区中允许存储的事件数量估计不超过1000，具体需要看事件类型和文件类型，操作系统 API 详情参考 [inotify man page](https://www.man7.org/linux/man-pages/man7/inotify.7.html)。

`LinuxWatchService` 的 `OVERFLOW` 事件处理逻辑如下：

```java
private void processEvent(int wd, int mask, final UnixPath name) {
    // overflow - signal all keys
    // mask 大于等于 IN_Q_OVERFLOW 时
    if ((mask & IN_Q_OVERFLOW) > 0) {
        for (Map.Entry<Integer,LinuxWatchKey> entry: wdToKey.entrySet()) {
            entry.getValue()
                .signalEvent(StandardWatchEventKinds.OVERFLOW, null);
        }
        return;
    }
...
}
```

另外，`java.io.FileSystem#list` 在 Linux 中的实现 `src/java.base/unix/native/libjava/UnixFileSystem_md.c` 中的 `Java_java_io_UnixFileSystem_list(JNIEnv *env, jobject this, jobject file)` 调用 [readdir](https://linux.die.net/man/3/readdir) 读取文件夹，`dirent->d_name` 读取文件夹内文件名，因为文件夹内的文件是文件夹的属性，因此速度很快，但是想要获取文件夹内文件本身的属性信息需要根据文件 `inode` 值去遍历读取因此比较慢，读取文件属性函数为 [stat](https://www.man7.org/linux/man-pages/man2/stat.2.html)。

##### WindowsWatchService

当 `GetQueuedCompletionStatus` 返回 `ERROR_NOTIFY_ENUM_DIR` 错误时即表示缓冲区溢出，更多参考 [GetQueuedCompletionStatus function (ioapiset.h)](https://docs.microsoft.com/en-us/windows/win32/api/ioapiset/nf-ioapiset-getqueuedcompletionstatus) 和 [System Error Codes (1000-1299)](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--1000-1299-)

> ERROR_NOTIFY_ENUM_DIR, 1022 (0x3FE):  A notify change request is being completed and the information is not being returned in the caller's buffer. The caller now needs to enumerate the files to find the changes.  

`WindowsWatchService` 的 `OVERFLOW` 事件处理逻辑如下：

```java
public void run() {
    for (;;) {
        CompletionStatus info;
        try {
            info = GetQueuedCompletionStatus(port);
        } catch (WindowsException x) {
            // this should not happen
            x.printStackTrace();
            return;
        }

        // wakeup
        if (info.completionKey() == WAKEUP_COMPLETION_KEY) {
            boolean shutdown = processRequests();
            if (shutdown) {
                return;
            }
            continue;
        }

        // map completionKey to get WatchKey
        WindowsWatchKey key = ck2key.get((int)info.completionKey());
        if (key == null) {
            // We get here when a registration is changed. In that case
            // the directory is closed which causes an event with the
            // old completion key.
            continue;
        }

        boolean criticalError = false;
        int errorCode = info.error();
        int messageSize = info.bytesTransferred();
        if (errorCode == ERROR_NOTIFY_ENUM_DIR) {
            // 触发 OVERFLOW
            // buffer overflow
            key.signalEvent(StandardWatchEventKinds.OVERFLOW, null);
        } else if (errorCode != 0 && errorCode != ERROR_MORE_DATA) {
            // ReadDirectoryChangesW failed
            criticalError = true;
        } else {
            // ERROR_MORE_DATA is a warning about incomplete
            // data transfer over TCP/UDP stack. For the case
            // [messageSize] is zero in the most of cases.

            if (messageSize > 0) {
                // process non-empty events.
                processEvents(key, messageSize);
            } else if (errorCode == 0) {
                // insufficient buffer size
                // not described, but can happen.
                // 触发 OVERFLOW
                key.signalEvent(StandardWatchEventKinds.OVERFLOW, null);
            }
        ...
        }
    }
}
```

### FileAlterationObserver

“commons-io” 提供了 `FileAlterationObserver`，与 JDK 内置的 `WatchService`相比功能更丰富，使用简单，以回调方式通知创建，删除等事件，而且当程序刚启动时会把旧文件做为创建事件并进行通知来避免旧文件被忽略。它是以文件名进行对比来通知文件的变化，可以自定义是否大小写敏感，也可配置文件过滤器只关注感兴趣的文件。而且不受文件文件描述符数量有限，缓冲区不足等限制，大量文件且要求业务复杂时推荐使用。

```java
File directory = new File("/tmp");
FileAlterationObserver observer = new FileAlterationObserver(directory);
observer.addListener(...);
observer.addListener(...);
// 初始化
observer.initialize();
// 检查文件事件
observer.checkAndNotify();
// 结束监听
observer.destroy();
```

如果需要长时间监控可通过 `FileAlterationMonitor` 创建新线程并按传入的时间间隔来执行监听。

```java
long interval = ...
FileAlterationMonitor monitor = new FileAlterationMonitor(interval);
monitor.addObserver(observer);
monitor.start();
...
monitor.stop();
```

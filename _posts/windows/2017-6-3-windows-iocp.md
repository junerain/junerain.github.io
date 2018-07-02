---
layout: post
title: "IOCP on Windows"
keywords: "asynchronously,iocp,windows"
description: ""
category: IOCP
tags: [Windows]
---
{% include JB/setup %}

[TOC]

## Windows异步IO (Asynchronous IO) (二)

前一篇文章我们知道如何向设备驱动发送异步IO请求。显然，仅仅知道这些肯定是不够的，用户线程必须在必要的时候收到设备驱动的完成通知`(Completion Notification)`，以执行相关任务，不然异步IO没有任何意义。Windows提供四种方法来接受来自设备驱动的完成通知。

也许有朋友已经想到了一个方法。前一篇提到，我们可以通过`Overlapped`的`Internal`成员判断IO请求的状态，所以我们可以实现一个busy loop来检查`Internal`的值是否为`STATUS_PENDING`，这不就行了么？理论上来说，当然是可以的，不过这似乎没有什么实用性。众所周知，busy loop太浪费CPU了！好不容易从慢速设备上省下来的CPU时间怎么能轻易用在空循环上？！事实上，Windows为我们提供了四种方法来接受设备驱动的完成通知，这里先介绍前三种。
### 触发设备内核对象(Singaling a Device Kernel Object)
 

在`ReadFile`/`WriteFile`函数将IO请求加入请求队列前，函数会将设备内核对象设置为未触发状态(Nonsignaled)。当设备驱动完成IO请求后，驱动会将设备内核对象设为触发状态(Signaled)。示例代码：
```
#include <windows.h>
#include <stdio.h>

int main()
{
	HANDLE		hFile = NULL;
	BYTE		buff[1024];
	OVERLAPPED	ol = {0};
	BOOL		bRet = FALSE;
	DWORD		dwCode = 0;
	
	hFile = CreateFile(TEXT("data.txt"), GENERIC_READ, FILE_SHARE_READ,
		               0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, NULL);
	
	ol.Offset = 16; //Read data from file starting at byte 16
	bRet = ReadFile(hFile, buff, 1024, NULL, &ol);
	
	if (bRet) //IO request is completed immediately
	{
	
	}
	else if ((dwCode = GetLastError()) == ERROR_IO_PENDING) //IO request is queued successfully
	{
		WaitForSingleObject(hFile, INFINITE); // The IO is being performed asynchronously; wait for it to complete
	}
	else
	{
		//Error Handling here
	}

	return EXIT_SUCCESS;
}
```
面的代码和前一篇的代码几乎相同，只是在第二个if语句中以hFile作为参数调用WaitForSignalObject函数。此调用会将用户线程挂起直到hFile内核对象变成触发状态。设备驱动在完成IO请求后就会触发hFile内核对象，使WaitForSignalObject函数返回，这样用户线程就能执行相应的任务。到这里，好像问题解决了，不过细心的朋友可能发现，此方法看似有效，事实上没有多大用处。因为它不能处理用户线程同时发送多个IO请求的情况。设备驱动在完成任何一个IO请求后，都会触发设备内核对象，导致WaitForSignalObejct函数返回，但用户线程没有得到任何有用的信息区分是哪个IO请求完成了。:)

### 触发事件核心对象(Signaling an Event Kernel Object)

现在我们知道了第一种方法没有什么实用性，不过有朋友可能又想到了解决方法！前一篇说过的，每一个IO请求都有一个对应的`Overlapped`结构，每一个`Overlapped`又都有一个`hEvent`成员，也就是说每一个IO请求都对应一个事件，这不就解决了么！示例代码：

```
#include <windows.h>
#include <stdio.h>

int main() {
	HANDLE		hFile = NULL;
	BYTE		bReadBuff[1024];
	BYTE		bWriteBuff[16] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15};
	OVERLAPPED	olRead = {0};
	OVERLAPPED	olWrite = {0};
	HANDLE		hEvents[2] = {0};
	DWORD		dwWaitRet = 0;

	hFile = CreateFile(TEXT("data.txt"), GENERIC_READ, FILE_SHARE_READ,
		               0, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, NULL);

	olRead.Offset = 16; //Read data from file starting at byte 16
	olRead.hEvent = CreateEvent(NULL,TRUE, FALSE, NULL);
	ReadFile(hFile, bReadBuff, 1024, NULL, &olRead);

	olWrite.hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
	WriteFile(hFile, bWriteBuff, 16, NULL, &olWrite);
	
	hEvents[0] = olRead.hEvent;
	hEvents[1] = olWrite.hEvent;
	dwWaitRet = WaitForMultipleObjects(2, hEvents, FALSE, INFINITE); //Wait multiple IO requests
	switch(dwWaitRet - WAIT_OBJECT_0)
	{
		case 0: //Read completed
			break;
		case 1:
			break; //Write completed
	}
	return EXIT_SUCCESS;
}
```
在发送一个IO请求前，用户线程给每一个`Overlapped`结构中的`hEvent`成员指定一个Event内核对象。设备驱动在完成一个IO请求后，会检查其`Overlapped`结构中的`hEvent`成员是否为空，如果不为空，则用`SetEvent`函数将其设置为触发状态。然后，用户线程用`WaitForMultipleObjects`函数等待前面的事件对象数组。在`WaitForMultipleObjects`函数返回后，用户线程可以通过返回值区分是哪个IO请求完成。怎么样，比第一种方法好用很多吧。:)

但它有一个明显的缺点，就是`WaitForMultipleObjects`一次能等待的最大`HANDLE`数不超过`MAXIMUM_WAIT_OBJECTS`，此值为64。不过也是有办法通过`WaitForMultipleObjects`等待超过64的`HANDLE`，这里就不介绍了。

### 可提醒IO(Alertable IO)
到这里，除了第一篇的内容，前面的两种方法都可以暂时忘记了。因为这种方法跟前面的两种方法没有任何相似性。

Windows在创建一个线程时，同时会给每个线程创建一个队列，叫做异步过程调用队列`(Asynchronous Procedure Call, APC)`。为了使用这个特性，我们应该用`ReadFileEx/WriteFileEx`函数替换原来的`ReadFile`/`WriteFile`函数。

```
BOOL ReadFileEx(
				HANDLE      hFile,
				PVOID       pvBuffer,
				DWORD       nNumBytesToRead,
				OVERLAPPED* pOverlapped,
				LPOVERLAPPED_COMPLETION_ROUTINE pfnCompletionRoutine);

BOOL WriteFileEx(
				 HANDLE      hFile,
				 CONST VOID  *pvBuffer,
				 DWORD       nNumBytesToWrite,
				 OVERLAPPED* pOverlapped,
				 LPOVERLAPPED_COMPLETION_ROUTINE pfnCompletionRoutine);
```

这两个方法与原来方法有两处不同。首先，`*Ex`没有一个指向`DWORD`的指针用来返回以传输的字节数。其次，`*Ex`函数需要指定一个回调函数地址，这个回调函数称为完成函数(Completion Routine)。

```
VOID WINAPI CompletionRoutine(
							  DWORD       dwError,
							  DWORD       dwNumBytes,
							  OVERLAPPED* po);
```
 当用户线程用*Ex函数发送一个IO请求时，函数会将完成函数的地址传给设备驱动。当设备驱动完成一个IO请求时，会在发出此IO请求的线程的APC队列中添加一项，此项包括完成函数地址和Overlapped结构的地址。当线程置于可提醒状态时，系统会检查它的APC队列，对队列中的每一项，系统会调用完成函数，并传入IO错误码，已传输的字节数，以及`Overlapped`结构地址。也就是说，可提醒IO使用户线程能在每一个IO请求被完成后，都能通过调用对应的完成函数来执行相关任务。当然，你不能期望执行完成函数的次序与发送IO请求的次序一致。

现在出现了新问题，**什么是线程的可提醒状态**。可提醒状态是线程可以被安全中断的状态，Windows提供了六个函数使线程置于可提醒状态。

```
DWORD SleepEx(
   DWORD dwMilliseconds,
   BOOL  bAlertable);

DWORD WaitForSingleObjectEx(
   HANDLE hObject,
   DWORD  dwMilliseconds,
   BOOL   bAlertable);

DWORD WaitForMultipleObjectsEx(
   DWORD   cObjects,
   CONST HANDLE* phObjects,
   BOOL    bWaitAll,
   DWORD   dwMilliseconds,
   BOOL    bAlertable);

BOOL SignalObjectAndWait(
   HANDLE hObjectToSignal,
   HANDLE hObjectToWaitOn,
   DWORD  dwMilliseconds,
   BOOL   bAlertable);

BOOL GetQueuedCompletionStatusEx(
   HANDLE hCompPort,
   LPOVERLAPPED_ENTRY pCompPortEntries,
   ULONG ulCount,
   PULONG pulNumEntriesRemoved,
   DWORD dwMilliseconds,
   BOOL bAlertable);

DWORD MsgWaitForMultipleObjectsEx(
   DWORD   nCount,
   CONST HANDLE* pHandles,
   DWORD   dwMilliseconds,
   DWORD   dwWakeMask,
   DWORD   dwFlags);
```
这六个函数最后一个参数都是BOOL类型，表示调用线程是否应该把自己置于可提醒状态。事实上，这些*Ex函数对应的非扩展版本在内部都调用对应的*Ex函数，当然bAlertable参数被设为FALSE。

到这里，可提醒IO的概貌就浮现了。不过，我们不提倡使用可提醒IO，因为它太麻烦。:)

 1. 用户线程必须为每一个IO请求实现一个完成函数，过多的IO请求当然会导致代码臃肿。如果共用完成函数，一个函数又难以提供足够的信息区分不同的情况，总之，麻烦;
 2. 发出IO请求的线程必须处理这些请求的完成函数，如果一个线程发出过多的IO请求，该线程也必须任劳任怨的处理每一个完成函数，即使系统中有其它空闲的线程。

不错，下一篇介绍的方法(IOCP)会很好的解决这两个问题。


### [IOCP实现](https://www.ibm.com/developerworks/cn/java/j-lo-iocp/)

 1. 创建一个完成端口:`Port port = createIoCompletionPort(INVALID_HANDLE_VALUE, 0, 0, fixedThreadCount());`
 2. 创建一个线程 ThreadA;
 3. ThreadA 线程循环调用 GetQueuedCompletionStatus 方法来得到 I/O 操作结果，这个方法是一个阻塞方法:
```
While(true)
{ 
    getQueuedCompletionStatus(port, ioResult); 
}
```
 4. 主线程循环调用 `accept` 等待客户端连接上来;
 5. 主线程 `accept` 返回新连接建立以后，把这个新的套接字句柄用 `CreateIoCompletionPort` 关联到完成端口，然后发出一个异步的 `Read` 或者 `Write` 调用，因为是异步函数，Read/Write 会马上返回，实际的发送或者接收数据的操作由操作系统去做;
```
if (handle != 0L) { 
    createIoCompletionPort(handle, port, key, 0); 
}
```
 6. 主线程继续下一次循环，阻塞在 accept 这里等待客户端连接;
 7. 操作系统完成 Read 或者 Write 的操作，把结果发到完成端口;
 8. ThreadA 线程里的 `GetQueuedCompletionStatus()` 马上返回，并从完成端口取得刚完成的 Read/Write 的结果;
 9. 在 ThreadA 线程里对这些数据进行处理 ( 如果处理过程很耗时，需要新开线程处理 )，然后接着发出 Read/Write，并继续下一次循环阻塞在 `GetQueuedCompletionStatus()` 这里。


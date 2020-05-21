# LINUX 系統程式設計 讀書心得

## 1.介紹
在linux系統程式設計下，有三大重點，分別為:系統呼叫 ，c程式庫和c編輯器。 
linux下，不同的機器架構會有不同的系統呼叫，但是有90%是所有架構都會做的。
這本書就是在討論這些共同介面。

應用程式通過"**機器暫存器**"通知核心要執行哪個呼叫和使用哪些參數。

linux下"**一切皆檔案**"，所以大部分操作是透過**檔案的讀寫**進行。

檔案要先開始才能讀取，需透過**一個獨一無二的描述器(descriptor)**，來參用已開啟的檔案，在linux核心，描述器是透過一個稱為**檔案描述器**(fd)的整數進行操作，並且fd由用戶空間程式共享。

檔案的大小又稱為檔案的長度，所以空的檔案長度為0，可以透過一個稱為**截短**的操作修改檔案長度，同檔案可被行程開啟一次以上，每個被開啟的檔案都會賦予一個獨一無二的fd，行程間共享fd，因此我認為這是讓LINUX可以多人多工的條件。

檔案透過**inode**賦予一個**inode number**的整數值 ，inode存取檔案有關的中介資料但是卻沒有檔名。
***************
### 目錄與連結
由於經過i-nimber存取檔案很麻煩，所以用戶使用名稱來開啟檔案。因此**目錄**提供存取檔案時的名稱，並把名稱映射至i-mumber(**filename - > i-number**)，因此核心直接用此關係進行解析，所以過程為:用戶要求開啟檔案->核心透過file name 找到i-number -> i-number 找到 inode。

為了讓連結可以跨檔案系統，所以還實作了"**符號連結**"此行為更像**捷徑**。

### 錯誤描述

透過特殊變數errno，此變數宣告於`<errno.h>`如 `extern int errno;`所示。









    
  
 

## 2.檔案

fd 0 **標準輸入**

fd 1 **標準輸出**

fd 2 **標準錯誤** 簡寫為stderr


## open()系統呼叫

使用**open()系統呼叫**開啟檔案後，取得一個**fd**
``` 
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open (const char *name, int flags);
int opne (const char *name, int flags , mode_t mode);

```   
flag引數是一或多格旗標逐位元OR運算的結果，引數必須是 `O_RDONLY` ， `O_WRONLY` ，`O_RDWR` 之一，分別為 讀取 寫入 讀寫。

開啟 */home/kidd/mada* 進行讀取
```
int fd 

fd = open ("/home/kidd/mada" ,O_RDONLY );
if (fd == -1);
 /*錯誤*/
```

並且使用O_CREAT旗標時，也需要指定mode引數

因 O_WRONLY | O_CREAT | O_TRUNC 太常見 所以存在一個系統呼叫專門提供此行為:
``` 
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int creat (const char *name, mode_t mode);

```

下為典型的creat()呼叫用法:
```
int fd 

fd = creat (filename ,0644 );
if (fd == -1);
 /*錯誤*/
```
### read()
最基本且常見的read()系統呼叫:
```
#include <unistd.h>

ssize_t read (int fd, void *buf , size_t len);
```
每次呼叫read()會從fd引數所參照的**當前檔案位置**讀取len個位元組到buf。 成功時傳回寫進buf的位元組數目。失敗時，傳回-1和設定errno。

以下範例會從fd讀取位元組到word。讀進的位元組數目=unsigned long 資料行別的大小。
```
unsigned long word ;
ssize_t nr;

/*從 'fd' 讀取位元到 'word' */
nr = read (fd , &word , sizeof (unsigned long));
if (nr == -1)
    /* 錯誤*/
```

### write() 
同read()有最基本用法:
```
const char *buf = "eat flower";
ssize_t nr;

nr = write (fd ,buf, strlen (buf));
if (nr == -1)
    /*錯誤*/
```

但是這種用法不太正確。呼叫者還需要檢查**部分寫入**的可能性:
```
unsigned long word = 1720 ; 
ssize_t count;
ssize_t nr ;

count = sizeof (word);
nr = write (fd, &word ,count);
if (nr == -1)
    /*錯誤，檢查errno*/
else if (nr != count )
    /*可能錯誤，但是未設定errno*/
```
---
除了以上的函式還有用於**同步I/O的**"fsync( ) 與 fdatasync( )和close( )"這一章介紹了基礎知識:檔案I/O。在LINUX之類的系統上，會盡量將一切表示成檔案。知道如何開啟，讀，寫，關檔案是非常重要的基礎，這些是典型的UNIX操作。



## 3.緩衝式
標準的I/O是標準C程式庫提供的用戶緩衝程式庫，除些小缺陷外，標準I/O是唯一的選擇

* 預期出現多次系統呼叫的問題，想要降低呼叫次數


* 效能非常重要，而想要確保所有IO都在**對齊區塊的邊界上**使用區塊大小的團塊

* 存取方式以字符或列為基礎，不用額外的呼叫

* 比起低階的系統呼叫更喜歡高階介面

----

以上四點只要有一項符合，就可以使用標準IO

## 4.進階檔案

* >第二章中，我了解到了LINUX檔案I/O的基礎，包括了UNIX程式設計的基本知識，和一些系統呼叫的方法。

* >第三章裡也可以實際的看到用戶空間緩衝機制和標準C程式庫的實作。然而在這一章也更進階了I/O的各個層面，例如:功能更強但是更複雜的系統呼叫，優化的技術和磁碟查找對效能的影響


## 5.行程管理
 >這一個章節稍微了解了行程和執行緒一些，還有如何運行或是終止一個行程。

## 6.進階的行程管理

>由第5章做更深入的延伸，裡面有相當多的函式，可以更進階的操做。

## 7.執行緒

>執行緒是行程中的執行單位，也就是，行程是運行中的二元檔，執行緒則是系統裡面行程排班器可以安排的最小單位，可是這一本書裡面只能看見它做一個基本的介紹，如果想要更深入的研究，就必須找其他資料了。


## 結語:
>這本**linux系統程式設計**上面有著相當多LINUX系統相關的基礎知識，同時也有著大量的範例來幫助讀者做一個更深入的了解，並不會只是單純地看看文字，並且上面也寫著非常之多的函式，可是我認為，如果不是常常使用這類工具的人，或許不可以牢牢的背起來，應該是把這本書當作一本工具書，在需要的時候可以適時地翻閱並且上網查找資料來幫助自己完成一個linux系統程式的設計。

## 參考來源:
>Robert Love 著，蔣大偉 譯<br>《Linux系統程式設計 第二版》
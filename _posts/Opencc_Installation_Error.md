# Compile and Install OpenCC on Minimal CentOS 7

測試過程在虛擬機中進行，使用vm搭建，操作系統版本`CentOS Linux release 7.2.1511 (Core)`，內核版本`3.10.0-327.10.1.el7.x86_64`。

## Downloading & Uncompressing
---
`OpenCC`代碼開源，公佈在GitHub上，地址 [https://github.com/BYVoid/OpenCC](https://github.com/BYVoid/OpenCC)，使用`wegt`命令下載，保存路徑爲`/usr/local/src`。

執行命令
```
sudo wget https://github.com/BYVoid/OpenCC/archive/master.zip
#重命名
sudo mv master.zip opencc.zip
```
文件是zip格式壓縮包，需要安裝unzip命令，否則會報錯

```
-bash: unzip: command not found
```

執行命令

```
sudo yum -y install unzip
```
執行`unzip`命令解壓到當前目錄

```
sudo unzip opencc.zip
[vagrant@opencc src]$ ls -lh
total 1.8M
drwxr-xr-x. 9 root root 4.0K Mar 10 21:57 OpenCC-master
-rw-r--r--. 1 root root 1.8M Mar 22 21:05 opencc.zip
[vagrant@opencc src]$
```

## Compilation
---
編譯主要2步：`make`, `make install`，提示`gcc 4.6 is required`。

安裝OpenCC需安裝以下依賴包

```
sudo yum install -y cmake gcc gcc-c++ doxygen
```
爲`/usr/lib/libopencc.so.2`創建符號鏈接至`/usr/lib64/libopencc.so.2`

以下是摸索過程

**Error1**

進入目錄後，執行sudo make，報錯
```
cmake: command not found
```
安裝cmake解決，執行命令

```
sudo yum install -y cmake
```
**Error2**

再次執行`sudo make`，報錯
```
CMake Error: CMAKE_CXX_COMPILER not set
```
因已安裝gcc，故安裝gcc-c++，執行命令

```
sudo yum install -y gcc-c++
```
**Error3**

報錯
```
Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE)
```
安裝`doxygen`，執行命令
```
sudo yum install -y doxygen
```
再次執行`sudo make`，正常編譯

執行`make make install`安裝
```
[vagrant@opencc OpenCC-master]$ which opencc
/usr/bin/opencc
[vagrant@opencc OpenCC-master]$
```
**Error4**

執行命令`opencc --help`報錯
```
opencc: error while loading shared libraries: libopencc.so.2: cannot open shared object file: No such file or directory
```
查找文件`libopencc.so.2`
```
[vagrant@opencc OpenCC-master]$ sudo find / -name libopencc.so.2
/usr/local/src/OpenCC-master/build/rel/src/libopencc.so.2
/usr/lib/libopencc.so.2
```
根據[繁体转简体，CentOS安装OpenCC，升级到gcc4.6](http://www.linuxdown.net/install/soft/2016/0122/4445.html)，創建符號鏈接至/usr/lib64/目錄下
```
[vagrant@opencc OpenCC-master]$ sudo ln -s /usr/lib/libopencc.so.2 /usr/lib64/libopencc.so.2
[vagrant@opencc OpenCC-master]$ opencc --version
Open Chinese Convert (OpenCC) Command Line Tool
Version: 1.0.3
[vagrant@opencc OpenCC-master]$
```
成功安裝

**Error5**
执行
```
opencc -i wiki.zh.text -o wiki.zh.text.jian -c zht2zhs.ini
```
报错
```
PARSE JSON ERROR
```
use xxx.json file instead of xxx.ini
```
opencc -i wiki.zh.text -o wiki.zh.text.jian -c t2s.json
```
example dir -> /usr/share/opencc/

## Usage
---
```
Usage:
   opencc  [--noflush <bool>] [-i <file>] [-o <file>] [-c <file>] [--]
           [--version] [-h]
Options:
   --noflush <bool>
     Disable flush for every line
   -i <file>,  --input <file>
     Read original text from <file>.
   -o <file>,  --output <file>
     Write converted text to <file>.
   -c <file>,  --config <file>
     Configuration file
   --,  --ignore_rest
     Ignores the rest of the labeled arguments following this flag.
   --version
     Displays version information and exits.
   -h,  --help
     Displays usage information and exits.
```
* -i: 指定輸入文件
* -o: 指定轉換後的輸出文件
* -c: 指定配置文件，以何種形式轉換

配置文件有如下幾種：

* `s2t.json` Simplified Chinese to Traditional Chinese 簡體到繁體
* `t2s.json` Traditional Chinese to Simplified Chinese 繁體到簡體
* `s2tw.json` Simplified Chinese to Traditional Chinese (Taiwan Standard) 簡體到臺灣正體
* `tw2s.json` Traditional Chinese (Taiwan Standard) to Simplified Chinese 臺灣正體到簡體
* `s2hk.json` Simplified Chinese to Traditional Chinese (Hong Kong Standard) 簡>體到香港繁體（香港小學學習字詞表標準）
* `hk2s.json` Traditional Chinese (Hong Kong Standard) to Simplified Chinese 香>港繁體（香港小學學習字詞表標準）到簡體
* `s2twp.json` Simplified Chinese to Traditional Chinese (Taiwan Standard) with Taiwanese idiom 簡體到繁體（臺灣正體標準）並轉* 換爲臺灣常用詞彙
* `tw2sp.json` Traditional Chinese (Taiwan Standard) to Simplified Chinese with Mainland Chinese idiom 繁體（臺灣正體標準）到簡* 體並轉換爲中國大陸常用詞彙
* `t2tw.json` Traditional Chinese (OpenCC Standard) to Taiwan Standard 繁體（OpenCC 標準）到臺灣正體
* `t2hk.json` Traditional Chinese (OpenCC Standard) to Hong Kong Standard 繁體（OpenCC 標準）到香港繁體（香港小學學習字詞表標準）

## Usage Example
---
**Example1**

使用配置文件`tw2s`將正體文件轉換爲簡體
```
[vagrant@opencc tmp]$ ls
tw.txt
[vagrant@opencc tmp]$ cat tw.txt
昨夜西風凋碧樹，獨上高樓，望盡天涯路
[vagrant@opencc tmp]$ opencc -i tw.txt -o cn.txt -c tw2s
[vagrant@opencc tmp]$ ls
cn.txt  tw.txt
[vagrant@opencc tmp]$ cat cn.txt
昨夜西风凋碧树，独上高楼，望尽天涯路
[vagrant@opencc tmp]$ cat tw.txt
昨夜西風凋碧樹，獨上高樓，望盡天涯路
[vagrant@opencc tmp]$
```
**Example2**

使用管道符`|`
```
[vagrant@opencc tmp]$ cat tw.txt
昨夜西風凋碧樹，獨上高樓，望盡天涯路
[vagrant@opencc tmp]$ cat tw.txt | opencc -c tw2s
昨夜西风凋碧树，独上高楼，望尽天涯路
[vagrant@opencc tmp]$
```

## References
---
* [Open Chinese Convert](https://github.com/BYVoid/OpenCC)
* [繁体转简体，CentOS安装OpenCC，升级到gcc4.6](http://www.linuxdown.net/install/soft/2016/0122/4445.html)
* [Compile and Install OpenCC on Minimal CentOS 7](https://lempstacker.com/tw/Compile-and-Install-OpenCC-on-Minimal-CentOS7/)

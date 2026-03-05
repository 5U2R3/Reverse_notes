# CTFから学ぶShellcode実行の仕組み

## 概要
本記事はMalwareTech氏によるマルウェア解析初心者向け演習問題を解きながら、Shellcodeが実行される仕組みを解説していきます。  
MalwareTech氏のWindowsReversingは、初心者に一般的なマルウェアのテクニックの理解と解析に慣れてもらうことが目的となっています。  
演習問題となるファイルはMalwareTech氏のブログにて公開されています。  
https://malwaretech.com/

## 目的
CTF形式の問題を解きながら、Shellcodeが実行される仕組みを理解する。  

## 環境
OS:Windows11 24H2(OSビルド26100.7840)  
IDA Free 8.4  
Ghidra Version 12.0.3 ※solveスクリプト作成用  
Java Version 21.0.6  
pyghidraを使用  
> [!WARNING]
> マルウェア解析は専門的な技術を必要とする危険な作業です。  
> オフラインのVMや独立した検証環境等の適切な環境と知識を持って  
> マルウェア解析を実施してください。  

## 解析順序
マルウェアの機能と特徴を理解し、システムへの影響を評価するため、様々な解析手法が存在します。  
マルウェアの解析は次の順序でアプローチすることが重要であると私は考えます。  
1. 表層解析(解析難易度：易)  
   プログラムを実行せずに分析対象ファイルに関連付けられているメタデータを抽出し、マルウェアの情報を集める手法
2. 簡易動的解析(解析難易度:易)  
   プログラムを実行し、動作を監視しながら挙動を理解する手法 
3. コード解析・静的解析(解析難易度：難)  
   プログラムを実行せずに分析対象ファイルを逆アセンブルし、コードを調べて動作を理解する手法
4. 詳細動的解析(解析難易度：中)  
   デバッガを使用してプログラムを実行し、内部動作を追跡して詳細な動作を理解する手法
  
悪意のあるマルウェアは解析環境の検知や自己削除といった挙動を行う可能性もあり、  
解析の当たりを決めずに実行するだけではマルウェアの正しい挙動を把握できない可能性があります。  
そのため、詳細動的解析はコード解析・静的解析を先に実施し、挙動を理解した上で進めることが重要です。  
※主要な挙動を理解した上で実行する重要性は、ペネトレーションテストなどで使用されるPoCにも同様のことが言えます。

## shellcode1-[★☆☆☆☆]  
shellcode1はシェルコードベースのマルウェア解析の初心者向け入門として位置付けられています。  
提供されたバイナリを解析し、フラグとなる文字列を探していきます。  
今回は表層解析、静的解析を用いてShellcode1にアプローチしていきます。  
その後、詳細動的解析を用いてFlagが生成される過程を確認します。  

### 表層解析

↓Detect It Easyの結果
![Shellcode1-001](/shellcode1/Shellcode1-001.png)  

### 静的解析
0x00402289でGetProcessHeap()、0x00402290でHeapAlloc()が呼び出されています。  
解析時にWindowsAPIの定義を確認しながら進めていくことでプログラムがどのような動作をしているかのヒントになります。  
[HeapAlloc関数](https://learn.microsoft.com/ja-jp/windows/win32/api/heapapi/nf-heapapi-heapalloc)

>DECLSPEC_ALLOCATOR LPVOID HeapAlloc(  
　[in] HANDLE hHeap,  
　[in] DWORD  dwFlags,  
　[in] SIZE_T dwBytes  
);

[VirtualAlloc関数](https://learn.microsoft.com/ja-jp/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc)  

>LPVOID VirtualAlloc(  
　[in, optional] LPVOID lpAddress,  
　[in]           SIZE_T dwSize,  
　[in]           DWORD  flAllocationType,  
　[in]           DWORD  flProtect  
);  

VirtualAlloc関数は呼び出し元プロセスの仮想アドレス空間内のページ領域の状態を予約、コミット、または変更する関数です。  

## まとめ

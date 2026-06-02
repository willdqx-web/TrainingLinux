# Day 6 — gdb デバッグ

## この研修全体の流れ

```
Day 1  → Day 2  → Day 3  → Day 4  → Day 5  → [Day 6] → Day 7  → Day 8  → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
                                              ★今日
```

Day 5 でビルドができるようになりました。今日は「バグを見つけるツール（デバッガ）」を学びます。Visual Studioではマウスでブレークポイントを張れましたが、gdb はコマンドラインで操作します。最初はコマンドの多さに戸惑いますが、**実際に使うコマンドは10個以下**です。

---

## この日のゴール

- `gdb` を起動してプログラムを実行できる
- ブレークポイントを設定してステップ実行できる
- 変数の値をgdbで確認できる
- クラッシュ（セグメンテーション違反）の発生箇所をgdbで特定できる
- バックトレースを読んで呼び出し元を追跡できる

---

## この日の前提

- Day 5 の作業が完了していること
- `gdb --version` でバージョンが表示されること

---

## 1. Visual Studio デバッガとの対比

Visual Studioでは GUIでデバッグするが、gdbはすべてコマンドで操作する。

| Visual Studio | gdb コマンド | 説明 |
|-------------|------------|------|
| F5（デバッグ開始） | `run` または `r` | プログラムを実行する |
| F10（ステップオーバー） | `next` または `n` | 関数をまたがずに次の行へ |
| F11（ステップイン） | `step` または `s` | 関数の中に入る |
| Shift+F11（ステップアウト） | `finish` または `fin` | 現在の関数を抜ける |
| F9（ブレークポイント設定） | `break main` | 関数名や行番号でブレークポイントを設定 |
| ウォッチウィンドウ | `print 変数名` / `display 変数名` | 変数の値を表示 |
| コールスタックウィンドウ | `backtrace` または `bt` | 関数の呼び出し履歴を表示 |
| 変数ウィンドウ（ローカル） | `info locals` | ローカル変数を一覧表示 |
| Ctrl+Shift+F5（再起動） | `run` | 最初から再実行 |
| 停止ボタン | `quit` または `q` | gdb を終了 |

---

## 2. gdb の起動と基本的な使い方

### デバッグビルドが必要

gdbを使うには、**`-g` フラグ付きでビルドしたバイナリ**が必要。`-g` なしのバイナリでもgdbは起動できるが、ソースコードとの対応が取れないため「行番号不明」「変数名不明」になる。

```bash
gcc -g -O0 main.c -o myapp     # 必ず -g と -O0 をつける
```

### gdb の起動

```bash
gdb ./myapp
```

起動すると gdb のプロンプトが表示される：

```
GNU gdb (Ubuntu 12.1-0ubuntu1~22.04) 12.1
...
Reading symbols from ./myapp...
(gdb) _
```

`(gdb)` がgdbのコマンドプロンプト。ここでコマンドを入力する。

---

## 3. gdb のコマンド詳解

### ブレークポイントの設定

```gdb
break main                  # main 関数の先頭にブレークポイント
break calc.c:10             # calc.c の10行目にブレークポイント
break add                   # add 関数の先頭にブレークポイント
info breakpoints            # 設定済みのブレークポイントを一覧表示
delete 1                    # ブレークポイント番号1を削除
delete                      # すべてのブレークポイントを削除（確認あり）
```

### プログラムの実行と制御

```gdb
run                         # プログラムを開始（r でも可）
run arg1 arg2               # コマンドライン引数付きで実行
continue                    # 次のブレークポイントまで実行（c でも可）
next                        # 次の行へ進む。関数呼び出しは飛び越える（n でも可）
step                        # 次の行へ進む。関数の中に入る（s でも可）
finish                      # 現在の関数を最後まで実行して戻る（fin でも可）
until 20                    # 20行目まで実行
```

### 変数・メモリの確認

```gdb
print x                     # 変数 x の値を表示（p でも可）
print *ptr                  # ポインタの指す先の値を表示
print arr[0]                # 配列の要素を表示
print &x                    # 変数 x のアドレスを表示
display x                   # ステップ実行のたびに自動で x を表示
info locals                 # ローカル変数を一覧表示
info args                   # 関数の引数を一覧表示
x/10d 0x7fff...             # アドレスからメモリを10要素分表示
```

### バックトレース（呼び出し履歴）

```gdb
backtrace                   # 関数の呼び出し履歴を表示（bt でも可）
frame 1                     # フレーム1（1つ上の呼び出し元）に移動
up                          # 1つ上のフレームに移動
down                        # 1つ下のフレームに移動
```

バックトレースの読み方：

```
(gdb) backtrace
#0  add (a=3, b=4) at calc.c:4
#1  0x0000555555555195 in main () at main.c:8
```

`#0` が現在の位置（最も深いフレーム）、数字が増えるにつれ呼び出し元になる。クラッシュ時は `#0` から順に読んで「どこで」「何が」起きたかを追う。

---

## 4. クラッシュ（セグメンテーション違反）の解析

### セグメンテーション違反とは

プログラムが**アクセスしてはいけないメモリ領域にアクセスした**ときにカーネルが送るシグナル（SIGSEGV）によってプロセスが強制終了する。

よくある原因：

```c
int *ptr = NULL;
*ptr = 10;          /* NULLポインタの参照 → セグフォ */

int arr[5];
arr[10] = 1;        /* バッファオーバーフロー → セグフォ */

int *p;             /* 未初期化ポインタ */
*p = 5;             /* → セグフォ */
```

### gdb でクラッシュ箇所を特定する手順

```bash
# 1. デバッグビルド
gcc -g -O0 crash.c -o crash

# 2. gdb で実行
gdb ./crash
(gdb) run

# 3. クラッシュすると自動停止する
Program received signal SIGSEGV, Segmentation fault.
0x0000555555555149 in some_function (p=0x0) at crash.c:8

# 4. バックトレースで呼び出し元を確認
(gdb) backtrace

# 5. 変数の値を確認
(gdb) info locals
(gdb) print ptr
```

---

## 5. ハンズオン

### Step 1：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day06
cd ~/training/day06
```

### Step 2：デバッグ対象のプログラムを作成する

**calc.h**

```bash
vim calc.h
```

```c
#ifndef CALC_H
#define CALC_H

int add(int a, int b);
int subtract(int a, int b);
int factorial(int n);

#endif
```

**calc.c**

```bash
vim calc.c
```

```c
#include "calc.h"

int add(int a, int b)
{
    return a + b;
}

int subtract(int a, int b)
{
    return a - b;
}

int factorial(int n)
{
    if (n <= 0) return 1;
    return n * factorial(n - 1);
}
```

**main.c**

```bash
vim main.c
```

```c
#include <stdio.h>
#include "calc.h"

int main(void)
{
    int a = 10;
    int b = 3;
    int result;

    result = add(a, b);
    printf("add(%d, %d) = %d\n", a, b, result);

    result = subtract(a, b);
    printf("subtract(%d, %d) = %d\n", a, b, result);

    result = factorial(5);
    printf("factorial(5) = %d\n", result);

    return 0;
}
```

デバッグビルド：

```bash
gcc -g -O0 calc.c main.c -o calc_debug
```

### Step 3：gdb でステップ実行する

```bash
gdb ./calc_debug
```

以下を順番に実行する：

```gdb
(gdb) break main
(gdb) run
```

`main` の先頭で止まる。

```gdb
(gdb) next          # 次の行へ（int a = 10; を実行）
(gdb) next          # 次の行へ（int b = 3; を実行）
(gdb) next          # 次の行へ（int result; を実行）
(gdb) print a       # a の値を確認 → $1 = 10
(gdb) print b       # b の値を確認 → $2 = 3
```

`add` 関数の中に入る：

```gdb
(gdb) next          # result = add(a, b); の行まで進む
(gdb) step          # add 関数の中に入る
(gdb) info args     # 引数を確認（a=10, b=3）
(gdb) info locals   # ローカル変数を確認
(gdb) backtrace     # 呼び出し履歴を確認
(gdb) finish        # add 関数を抜けて main に戻る
(gdb) print result  # 戻り値が格納されたか確認
```

すべての操作が終わったら終了：

```gdb
(gdb) continue      # 最後まで実行
(gdb) quit          # gdb を終了
```

### Step 4：クラッシュするプログラムを解析する

```bash
vim crash.c
```

```c
#include <stdio.h>
#include <stdlib.h>

void process_data(int *data, int size)
{
    int i;
    for (i = 0; i <= size; i++) {   /* バグ：<= は 1 つ多い */
        data[i] = i * 2;
    }
}

int main(void)
{
    int buf[5];
    process_data(buf, 5);
    printf("完了\n");
    return 0;
}
```

デバッグビルドして実行：

```bash
gcc -g -O0 crash.c -o crash
./crash
# Segmentation fault (core dumped) と表示される
```

gdb でクラッシュ箇所を特定する：

```bash
gdb ./crash
(gdb) run
# Program received signal SIGSEGV
(gdb) backtrace
(gdb) info locals
(gdb) print i
(gdb) print size
```

**クラッシュの原因を確認する**：`i <= size`（`i` が0〜5）で `data[5]` にアクセスしているが、`buf` は要素数5（インデックス0〜4）のため、`data[5]` はバッファ外アクセス。

```bash
# 修正
vim crash.c
# i <= size を i < size に変更
gcc -g -O0 crash.c -o crash
./crash
# 完了
```

### Step 5：display でウォッチを設定する

```bash
gdb ./calc_debug
(gdb) break factorial
(gdb) run
(gdb) display n         # ステップごとに n を自動表示
(gdb) continue          # factorial(5) に到達
(gdb) next              # n の変化を見ながらステップ実行
(gdb) next
...
(gdb) quit
```

### Step 6：今日の作業をコミットする

```bash
cd ~/training
git add day06/
git commit -m "day06: gdbデバッグ（ステップ実行とクラッシュ解析）を学習"
git push origin main
```

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `No debugging symbols found in ./myapp` | `-g` フラグなしでビルドしている | `gcc -g -O0 main.c -o myapp` で再ビルド |
| `No source file named main.c` | ビルドした場所と違うディレクトリで gdb を起動している | gdb 起動後に `(gdb) directory /path/to/source` でソースパスを指定 |
| ブレークポイントで止まらない | `-O2` 最適化でコードが変形している | `-O0` で再ビルドする |
| `Quit anyway? (y or n)` | 実行中にquitしようとした | `y` を入力して終了する |
| ステップ実行が飛ぶ | 最適化でコードが並び替えられている | `-O0` で再ビルドする |

---

## この日の Git チェックポイント

### いつコミットするか

| タイミング | 理由 |
|-----------|------|
| Step 3 完了（ステップ実行の確認） | 基本的なデバッグ操作を習得した状態を記録 |
| Step 4 完了（クラッシュ解析と修正） | バグを発見・修正できた状態を記録 |

### コミットメッセージ例

```bash
git commit -m "day06: gdbでステップ実行とbreak/printを確認"
git commit -m "day06: セグフォのクラッシュ箇所をgdbで特定して修正"
```

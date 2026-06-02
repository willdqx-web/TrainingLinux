# Day 5 — gcc/g++ でのビルド

## この研修全体の流れ

```
Day 1  → Day 2  → Day 3  → Day 4  → [Day 5] → Day 6  → Day 7  → Day 8  → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
                                    ★今日
```

Day 4 まででLinuxの基本操作が一通りできるようになりました。今日からは本題である「C/C++開発ツールチェーン」に入ります。Visual Studio（MSVC）とコマンドラインの対比を意識しながら学びます。

---

## この日のゴール

- `gcc`/`g++` で C/C++ ファイルをコンパイル・リンクできる
- 主要なコンパイルフラグの意味と使い所を説明できる
- 複数ファイルを分割コンパイルしてリンクできる
- 静的ライブラリ（`.a`）を作成してリンクできる

---

## この日の前提

- Day 1〜4 の作業が完了していること
- `gcc --version` でバージョンが表示されること

---

## 1. MSVC と gcc/g++ の対比

Visual Studioでは「ソリューションを開いてF5キーを押す」だけでビルド・実行ができた。Linuxではコマンドラインでコンパイラを直接呼び出す。

### 対比表

| Visual Studio (MSVC) | Linux (gcc/g++) |
|---------------------|----------------|
| `cl.exe` | `gcc`（C）、`g++`（C++） |
| `.vcxproj`（プロジェクトファイル） | `Makefile` / `CMakeLists.txt`（Day 7, 8 で学ぶ） |
| `.obj`（オブジェクトファイル） | `.o`（オブジェクトファイル） |
| `.lib`（静的ライブラリ） | `.a`（静的ライブラリ） |
| `.dll`（動的ライブラリ） | `.so`（共有ライブラリ） |
| `.exe`（実行ファイル） | `.out` または名前なし（実行ファイル） |
| デバッグビルド（F5） | `gcc -g` |
| リリースビルド | `gcc -O2` |

### ビルドの流れ

```
ソースファイル（.c / .cpp）
        │
        │ コンパイル（gcc -c）
        ▼
オブジェクトファイル（.o）
        │
        │ リンク（gcc で複数の .o をまとめる）
        ▼
実行ファイル（a.out または指定した名前）
```

---

## 2. gcc の基本構文

```bash
gcc [オプション] ソースファイル [-o 出力ファイル名]
```

### 最もシンプルなコンパイル

```bash
gcc main.c              # コンパイルして a.out を生成（デフォルトの出力名）
./a.out                 # 実行

gcc main.c -o myapp     # 出力ファイル名を指定
./myapp                 # 実行
```

### C++ は `g++` を使う

```bash
g++ main.cpp -o myapp   # C++ は g++ を使う（gcc ではリンクエラーになる場合がある）
```

---

## 3. 主要なコンパイルフラグ

### 警告関連（必ず使う）

| フラグ | 意味 | MSVCの相当 |
|--------|------|-----------|
| `-Wall` | 基本的な警告を有効にする | `/W3` |
| `-Wextra` | 追加の警告を有効にする | `/W4` |
| `-Werror` | 警告をエラーとして扱う | `/WX` |

```bash
gcc -Wall -Wextra main.c -o myapp
```

> **実務での推奨**：最低でも `-Wall` は常につける。警告を放置すると組み込み開発では誤動作の原因になる。

### 最適化

| フラグ | 意味 | 用途 |
|--------|------|------|
| `-O0` | 最適化なし（デフォルト） | デバッグ時（コードと実行の対応が崩れない） |
| `-O1` | 軽い最適化 | — |
| `-O2` | 標準的な最適化 | リリースビルド |
| `-O3` | 積極的な最適化 | 速度重視（コードサイズが増えることがある） |
| `-Os` | サイズ優先の最適化 | 組み込みでフラッシュ容量が限られる場合 |

```bash
gcc -O2 main.c -o myapp        # リリースビルド
gcc -O0 -g main.c -o myapp_d   # デバッグビルド（次の -g を参照）
```

### デバッグ情報

| フラグ | 意味 |
|--------|------|
| `-g` | gdb 用のデバッグ情報を埋め込む |
| `-g3` | マクロ情報も含めた完全なデバッグ情報 |

```bash
gcc -g -O0 main.c -o myapp     # デバッグビルド（-g と -O0 を組み合わせる）
```

> **注意**：`-g` と `-O2` を同時に使うと、最適化によってコードが変形し、デバッガでステップ実行がずれることがある。デバッグ時は `-O0` を推奨。

### C/C++言語バージョン

| フラグ | 意味 |
|--------|------|
| `-std=c11` | C11 準拠 |
| `-std=c17` | C17 準拠（現在の推奨） |
| `-std=c++14` | C++14 準拠 |
| `-std=c++17` | C++17 準拠（現在の推奨） |

```bash
gcc -std=c17 -Wall main.c -o myapp
g++ -std=c++17 -Wall main.cpp -o myapp
```

### インクルードパスとライブラリパス

| フラグ | 意味 | MSVCの相当 |
|--------|------|-----------|
| `-I/path/to/include` | ヘッダファイルの検索パスを追加 | プロジェクト設定の「インクルードディレクトリ」 |
| `-L/path/to/lib` | ライブラリの検索パスを追加 | プロジェクト設定の「ライブラリディレクトリ」 |
| `-lmylib` | `libmylib.a` または `libmylib.so` をリンク | プロジェクト設定の「追加の依存ファイル」 |

```bash
gcc main.c -I./include -L./lib -lmylib -o myapp
# ./include/ でヘッダを探す
# ./lib/libmylib.a でライブラリを探す
```

### プリプロセッサマクロの定義

| フラグ | 意味 | MSVCの相当 |
|--------|------|-----------|
| `-DDEBUG` | `DEBUG` マクロを定義 | プロジェクト設定の「プリプロセッサの定義」 |
| `-DVERSION=2` | `VERSION=2` を定義 | |

```bash
gcc -DDEBUG -DBOARD_RPIBOARD main.c -o myapp
```

ソースコード内での使い方：

```c
#ifdef DEBUG
    printf("デバッグ情報: value = %d\n", value);
#endif

#if BOARD_RPIBOARD
    /* Raspberry Pi固有の処理 */
#endif
```

---

## 4. 分割コンパイル

### なぜ分割コンパイルするのか

大規模なプロジェクトでは、1つのファイルをすべて書くのではなく機能ごとにファイルを分ける。変更したファイルだけを再コンパイルすることで、ビルド時間を短縮できる。

```
math.c   → math.o   ─┐
string.c → string.o ─┤── リンク → myapp
main.c   → main.o   ─┘
```

`main.c` だけ変更した場合、`math.o` と `string.o` は再コンパイル不要。

### 分割コンパイルの手順

```bash
# Step 1：各ソースファイルを個別にコンパイルして .o を作る（-c オプション）
gcc -c math.c -o math.o
gcc -c string.c -o string.o
gcc -c main.c -o main.o

# Step 2：.o をリンクして実行ファイルを作る
gcc math.o string.o main.o -o myapp
```

---

## 5. 静的ライブラリ（`.a`ファイル）の作成

### 静的ライブラリとは

複数のオブジェクトファイルをまとめたアーカイブファイル。`ar` コマンドで作成する。

Windowsの `.lib` ファイルに相当する。リンク時に実行ファイルに取り込まれる（リンク先にライブラリファイルを配置する必要がない）。

```bash
# Step 1：各 .c をコンパイル
gcc -c math.c -o math.o
gcc -c string.c -o string.o

# Step 2：ar でアーカイブ（静的ライブラリ）を作成
ar rcs libutils.a math.o string.o
#  │││
#  │││ s = シンボルテーブルを追加（必須）
#  ││└── c = アーカイブを作成（なければ作る）
#  │└─── r = ファイルを追加・置換
#  └──── ar コマンド

# Step 3：ライブラリをリンクして実行ファイルを作る
gcc main.o -L. -lutils -o myapp
#               │       │
#               │       └── libutils.a をリンク（lib と .a は省略）
#               └── カレントディレクトリからライブラリを探す
```

---

## 6. ハンズオン

### Step 1：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day05/01_basic
cd ~/training/day05/01_basic
```

### Step 2：Hello World のコンパイルを試す

```bash
vim hello.c
```

以下を入力して `:wq` で保存：

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, Linux!\n");
    return 0;
}
```

コンパイルして実行：

```bash
gcc hello.c -o hello
./hello
# 出力：Hello, Linux!
```

### Step 3：警告フラグを試す

```bash
vim warning_test.c
```

以下を入力（警告が出るコード）：

```c
#include <stdio.h>

int main(void)
{
    int x;                  /* 初期化されていない変数 */
    printf("%d\n", x);      /* 未初期化の変数を使用 */
    return 0;
}
```

警告なし（ダメな例）と警告あり（正しい例）を比較：

```bash
# 警告なし
gcc warning_test.c -o warning_test
./warning_test         # 不定の値が出力される（危険）

# 警告あり
gcc -Wall -Wextra warning_test.c -o warning_test
# warning: 'x' is used uninitialized in this function
```

### Step 4：デバッグビルドを試す

```bash
# デバッグビルド（-g -O0）
gcc -g -O0 hello.c -o hello_debug

# リリースビルド（-O2）
gcc -O2 hello.c -o hello_release

# ファイルサイズを比較する
ls -lh hello_debug hello_release
```

デバッグビルドの方がサイズが大きい（デバッグ情報が含まれるため）。

### Step 5：分割コンパイルを実践する

```bash
mkdir -p ~/training/day05/02_split
cd ~/training/day05/02_split
```

ファイルを3つ作成する：

**calc.h**（インターフェース定義）

```bash
vim calc.h
```

```c
#ifndef CALC_H
#define CALC_H

int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);

#endif /* CALC_H */
```

**calc.c**（実装）

```bash
vim calc.c
```

```c
#include "calc.h"

int add(int a, int b)      { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }
```

**main.c**（メインプログラム）

```bash
vim main.c
```

```c
#include <stdio.h>
#include "calc.h"

int main(void)
{
    printf("3 + 4 = %d\n", add(3, 4));
    printf("10 - 3 = %d\n", subtract(10, 3));
    printf("5 * 6 = %d\n", multiply(5, 6));
    return 0;
}
```

分割コンパイルしてリンク：

```bash
# 各ファイルをコンパイル
gcc -c calc.c -o calc.o
gcc -c main.c -o main.o

# 確認
ls -l *.o

# リンク
gcc calc.o main.o -o calculator

# 実行
./calculator
```

期待する出力：

```
3 + 4 = 7
10 - 3 = 7
5 * 6 = 30
```

### Step 6：静的ライブラリを作成する

```bash
mkdir -p ~/training/day05/03_library
cd ~/training/day05/03_library

# 先ほどの calc.h と calc.c をコピー
cp ~/training/day05/02_split/calc.h .
cp ~/training/day05/02_split/calc.c .
cp ~/training/day05/02_split/main.c .
```

静的ライブラリを作成してリンク：

```bash
# .o を作成
gcc -c calc.c -o calc.o

# 静的ライブラリを作成
ar rcs libcalc.a calc.o

# ライブラリの中身を確認
ar -t libcalc.a

# ライブラリをリンクして実行ファイルを作成
gcc main.c -L. -lcalc -o calculator

# 実行
./calculator
```

### Step 7：今日の作業をコミットする

```bash
cd ~/training
git add day05/
git commit -m "day05: gcc分割コンパイルと静的ライブラリ作成を学習"
git push origin main
```

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `undefined reference to 'add'` | リンク時に calc.o が含まれていない | `gcc main.o calc.o -o myapp` のように全 .o を指定する |
| `cannot find -lutils` | ライブラリが見つからない | `-L.`（カレントディレクトリを検索）か正しいパスを `-L` で指定する |
| `implicit declaration of function 'add'` | ヘッダファイルがインクルードされていない | `#include "calc.h"` を main.c に追加する |
| `multiple definition of 'main'` | main 関数が複数のファイルにある | main.c だけに `main` を書く |
| `warning: implicit declaration` | 関数を使う前に宣言されていない | ヘッダファイルに関数プロトタイプを書いてインクルードする |

---

## この日の Git チェックポイント

### いつコミットするか

| タイミング | コミットの意味 | 理由 |
|-----------|--------------|------|
| Step 2 完了（Hello Worldビルド成功） | gcc の基本動作確認 | ツールチェーンが動く最小の安全地点 |
| Step 5 完了（分割コンパイル成功） | 分割コンパイルの実習完了 | 次の静的ライブラリ実習のベース |
| Step 6 完了（静的ライブラリ完成） | ライブラリ作成の実習完了 | Day 7（Makefile）でこれを自動化する |

### コミットメッセージ例

```bash
git commit -m "day05: gccの基本コンパイルを確認（hello world）"
git commit -m "day05: 分割コンパイルとリンクを実践"
git commit -m "day05: 静的ライブラリ（libcalc.a）の作成とリンクを完了"
```

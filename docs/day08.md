# Day 8 — CMake

## この研修全体の流れ

```
Day 1  → Day 2  → Day 3  → Day 4  → Day 5  → Day 6  → Day 7  → [Day 8] → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file            課題
                                                                ★今日
```

Day 7 でMakefileを学びました。Makefileはシンプルなプロジェクトに向いていますが、規模が大きくなると管理が複雑になります。CMakeは「Makefileを生成するツール」で、より構造的にビルド設定を書けます。現代の組み込み開発プロジェクトではCMakeが標準になりつつあります。

---

## この日のゴール

- CMakeの「生成（configure）→ ビルド」の2ステップを理解できる
- `CMakeLists.txt` の基本構造を書ける
- ライブラリの作成・リンクをCMakeで定義できる
- デバッグ・リリースビルドを切り替えられる
- `target_compile_options` でコンパイルフラグを設定できる

---

## この日の前提

- Day 7 の Makefile 実習が完了していること
- `cmake --version` でバージョンが表示されること

---

## 1. CMakeとMakefileの違い

### なぜCMakeが必要か

| 比較 | Makefile | CMake |
|------|---------|-------|
| 記述形式 | 手続き的（何をどう実行するか） | 宣言的（何を作りたいか） |
| クロスプラットフォーム | Linux専用の記述になりがち | Windows/Linux/Macで同じファイルが使える |
| 依存関係管理 | 手動 or `-MMD` で自動生成 | 自動 |
| IDE連携 | 難しい | VS Code、CLionのプロジェクト生成ができる |
| 規模 | 小〜中規模向け | 中〜大規模向け |

### CMakeのビルドの流れ

```
CMakeLists.txt（設定ファイル）
        │
        │ cmake コマンド（設定を読んでMakefileを生成）
        ▼
build/ ディレクトリ
  ├── Makefile（CMakeが生成）
  ├── CMakeCache.txt（設定のキャッシュ）
  └── ...
        │
        │ cmake --build . （makeを実行）
        ▼
実行ファイル・ライブラリ
```

> **アウトオブソースビルド**：ソースコードのあるディレクトリとは別の `build/` ディレクトリにビルド成果物を置く。`git clean` するだけでビルド成果物をきれいに消せる。Makefileでは手動で行う必要があった。

---

## 2. CMakeLists.txtの基本構造

```cmake
# CMakeの最低バージョンを指定（まず書く）
cmake_minimum_required(VERSION 3.16)

# プロジェクト名とバージョンを定義
project(MyApp VERSION 1.0)

# 使用するC標準を指定
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)

# 実行ファイルを定義
add_executable(myapp main.c calc.c)
```

---

## 3. ターゲットの概念

CMakeでは「ターゲット」単位でビルドを定義する。ターゲットには実行ファイルとライブラリがある。

```cmake
# 実行ファイルターゲット
add_executable(myapp main.c)

# 静的ライブラリターゲット
add_library(calc STATIC calc.c)

# 動的ライブラリターゲット
add_library(calc SHARED calc.c)
```

ターゲットにプロパティ（フラグ、インクルードパス、リンク先）を設定するには `target_*` コマンドを使う：

```cmake
# コンパイルオプションを設定
target_compile_options(myapp PRIVATE -Wall -Wextra)

# インクルードパスを設定
target_include_directories(myapp PRIVATE include/)

# ライブラリをリンク
target_link_libraries(myapp PRIVATE calc)
```

### `PRIVATE` / `PUBLIC` / `INTERFACE` の違い

| キーワード | 意味 |
|---------|------|
| `PRIVATE` | このターゲットだけに適用 |
| `PUBLIC` | このターゲットと、このターゲットをリンクするターゲットにも適用 |
| `INTERFACE` | このターゲットをリンクするターゲットだけに適用 |

通常は `PRIVATE` を使う。ライブラリのヘッダが使用側にも必要なときは `PUBLIC`。

---

## 4. ライブラリの作成とリンク

```cmake
cmake_minimum_required(VERSION 3.16)
project(Calculator)

set(CMAKE_C_STANDARD 17)

# calc ライブラリ（静的）を定義
add_library(calc STATIC calc.c)
target_include_directories(calc PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
# PUBLIC にすることで、calc をリンクするターゲットも
# このディレクトリをインクルードパスとして使える

# 実行ファイルを定義
add_executable(calculator main.c)

# calc ライブラリをリンク
target_link_libraries(calculator PRIVATE calc)
```

---

## 5. コンパイルオプションの設定

```cmake
# すべてのターゲットにデフォルトフラグを設定（好ましくないが簡単）
add_compile_options(-Wall -Wextra)

# ターゲット個別に設定（推奨）
target_compile_options(myapp PRIVATE -Wall -Wextra -Werror)

# プリプロセッサマクロを定義
target_compile_definitions(myapp PRIVATE DEBUG_MODE)
target_compile_definitions(myapp PRIVATE "BOARD_VERSION=2")
```

---

## 6. デバッグ・リリースビルドの切り替え

CMakeでは `-DCMAKE_BUILD_TYPE` オプションでビルドタイプを指定する。

| ビルドタイプ | gccフラグ | 用途 |
|------------|---------|------|
| `Debug` | `-g -O0` | デバッグ時 |
| `Release` | `-O3 -DNDEBUG` | リリース時 |
| `RelWithDebInfo` | `-O2 -g -DNDEBUG` | リリースだがデバッグ情報あり |
| `MinSizeRel` | `-Os -DNDEBUG` | サイズ最小化（組み込み向け） |

```bash
# デバッグビルド
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build

# リリースビルド
cmake -B build_release -DCMAKE_BUILD_TYPE=Release
cmake --build build_release
```

---

## 7. サブディレクトリ構成

大きなプロジェクトはディレクトリを分けて管理する。

```
project/
├── CMakeLists.txt      # トップレベル
├── src/
│   ├── CMakeLists.txt  # 実行ファイルの定義
│   └── main.c
└── lib/
    ├── CMakeLists.txt  # ライブラリの定義
    ├── calc.c
    └── calc.h
```

**トップレベルの CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.16)
project(Calculator)

set(CMAKE_C_STANDARD 17)

add_subdirectory(lib)   # lib/CMakeLists.txt を読み込む
add_subdirectory(src)   # src/CMakeLists.txt を読み込む
```

**lib/CMakeLists.txt**

```cmake
add_library(calc STATIC calc.c)
target_include_directories(calc PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

**src/CMakeLists.txt**

```cmake
add_executable(calculator main.c)
target_link_libraries(calculator PRIVATE calc)
target_compile_options(calculator PRIVATE -Wall -Wextra)
```

---

## 8. ハンズオン

### Step 1：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day08
cd ~/training/day08
```

### Step 2：シンプルなCMakeプロジェクトを作成する

```bash
cp ~/training/day05/02_split/calc.h .
cp ~/training/day05/02_split/calc.c .
cp ~/training/day05/02_split/main.c .
```

```bash
vim CMakeLists.txt
```

```cmake
cmake_minimum_required(VERSION 3.16)
project(Calculator VERSION 1.0)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_executable(calculator main.c calc.c)
target_compile_options(calculator PRIVATE -Wall -Wextra)
```

### Step 3：ビルドする（アウトオブソースビルド）

```bash
# build ディレクトリを作成して cmake を実行（Makefileが生成される）
cmake -B build -DCMAKE_BUILD_TYPE=Debug

# ビルド
cmake --build build

# 実行
./build/calculator
```

期待する出力：

```
3 + 4 = 7
10 - 3 = 7
5 * 6 = 30
```

ディレクトリ構成を確認する：

```bash
ls build/
# CMakeCache.txt  CMakeFiles  Makefile  calculator  cmake_install.cmake
```

ソースディレクトリ（`~/training/day08/`）は `.o` ファイルなどで汚れていない。

### Step 4：ライブラリを分離したプロジェクトに改善する

```bash
mkdir -p ~/training/day08/v2/lib
mkdir -p ~/training/day08/v2/src
cd ~/training/day08/v2
```

ファイルを配置する：

```bash
cp ~/training/day08/calc.h lib/
cp ~/training/day08/calc.c lib/
cp ~/training/day08/main.c src/
```

**lib/CMakeLists.txt** を作成：

```bash
vim lib/CMakeLists.txt
```

```cmake
add_library(calc STATIC calc.c)
target_include_directories(calc PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

**src/CMakeLists.txt** を作成：

```bash
vim src/CMakeLists.txt
```

```cmake
add_executable(calculator main.c)
target_link_libraries(calculator PRIVATE calc)
target_compile_options(calculator PRIVATE -Wall -Wextra)
```

**トップレベルの CMakeLists.txt** を作成：

```bash
vim CMakeLists.txt
```

```cmake
cmake_minimum_required(VERSION 3.16)
project(Calculator VERSION 1.0)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_subdirectory(lib)
add_subdirectory(src)
```

ビルドして実行：

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
./build/src/calculator
```

### Step 5：デバッグ・リリースビルドを切り替える

```bash
# デバッグビルド（シンボルあり）
cmake -B build_debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build_debug
file build_debug/src/calculator
# ... with debug_info, not stripped

# リリースビルド（最適化あり）
cmake -B build_release -DCMAKE_BUILD_TYPE=Release
cmake --build build_release
file build_release/src/calculator
# ... stripped
```

### Step 6：gdbでデバッグビルドを確認する

```bash
gdb ./build_debug/src/calculator
(gdb) break main
(gdb) run
(gdb) next
(gdb) info locals
(gdb) quit
```

### Step 7：今日の作業をコミットする

```bash
cd ~/training
git add day08/
git commit -m "day08: CMakeのアウトオブソースビルドとライブラリ分離を学習"
git push origin main
```

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `CMake Error: The source directory does not appear to contain CMakeLists.txt` | CMakeLists.txtが見つからない | `cmake -B build` を正しいディレクトリで実行しているか確認 |
| `target_link_libraries: PRIVATE requires cmake 3.x` | cmake のバージョンが古い | `cmake_minimum_required(VERSION 3.16)` を確認し、`cmake --version` でバージョンを確認 |
| `cannot find -lcalc` | ライブラリのターゲット名が間違っている | `add_library(calc ...)` のターゲット名と `target_link_libraries(... calc)` の名前が一致しているか確認 |
| ビルドが通らない（CMakeCache の問題） | CMakeCache.txtに古い設定が残っている | `rm -rf build/` で build ディレクトリを削除してやり直す |
| ヘッダが見つからない | インクルードパスが設定されていない | `target_include_directories` でパスを追加する |

---

## この日の Git チェックポイント

### いつコミットするか

| タイミング | 理由 |
|-----------|------|
| Step 3 完了（最初のCMakeビルド成功） | CMakeの基本動作を確認した状態を記録 |
| Step 4 完了（サブディレクトリ構成） | プロジェクト構造の整理ができた状態を記録 |

### コミットメッセージ例

```bash
git commit -m "day08: 最初のCMakeLists.txtでアウトオブソースビルドを確認"
git commit -m "day08: lib/src分離構成のCMakeプロジェクトを作成"
git commit -m "day08: Debug/Releaseビルドタイプの切り替えを確認"
```

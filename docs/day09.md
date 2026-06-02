# Day 9 — 統合課題

## この研修全体の流れ

```
Day 1  → Day 2  → Day 3  → Day 4  → Day 5  → Day 6  → Day 7  → Day 8  → [Day 9]
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
                                                                        ★今日
```

Day 1〜8 で学んだすべてのスキルを使って、**ゼロからC言語プロジェクトを構築します**。手順書なしで自分の力だけで完成させることが目標です。

---

## この日のゴール

- ゼロからCMakeプロジェクトを構築できる
- 静的ライブラリと実行ファイルをCMakeで定義できる
- gdbでバグを発見・修正できる
- Gitでコミットして最終提出できる

**この課題が完了したとき、「OJT初日から戦力として動けるレベル」に達したことを意味します。**

---

## 課題の概要

「センサーデータ処理ライブラリ」と「そのライブラリを使うCLIツール」を作成する。

```
project/
├── CMakeLists.txt
├── lib/
│   ├── CMakeLists.txt
│   ├── sensor.h        ← 提供する
│   └── sensor.c        ← 自分で実装する
└── app/
    ├── CMakeLists.txt
    └── main.c          ← 自分で実装する
```

---

## 課題仕様

### ライブラリ（`lib/sensor.c`）が実装すべき関数

```c
/* センサーデータを格納する構造体 */
typedef struct {
    float temperature;    /* 温度（摂氏） */
    float humidity;       /* 湿度（%） */
    int   timestamp;      /* タイムスタンプ（Unix秒） */
} SensorData;

/* 温度の平均値を計算する
 * data    : SensorDataの配列
 * count   : 配列の要素数
 * 戻り値   : 平均温度。count が 0 以下の場合は 0.0f を返す
 */
float calc_avg_temperature(const SensorData *data, int count);

/* 温度の最大値を返す
 * 戻り値   : 最高温度。count が 0 以下の場合は 0.0f を返す
 */
float calc_max_temperature(const SensorData *data, int count);

/* 温度の最小値を返す
 * 戻り値   : 最低温度。count が 0 以下の場合は 0.0f を返す
 */
float calc_min_temperature(const SensorData *data, int count);

/* 湿度の平均値を計算する
 * 戻り値   : 平均湿度。count が 0 以下の場合は 0.0f を返す
 */
float calc_avg_humidity(const SensorData *data, int count);
```

### CLIツール（`app/main.c`）が出力すべき内容

テストデータ（以下のデータをコード内にハードコード）を処理して以下を出力する：

```c
/* テストデータ（main.c 内にハードコードする） */
SensorData test_data[] = {
    {25.3f, 60.0f, 1000},
    {27.1f, 58.5f, 1060},
    {24.8f, 62.3f, 1120},
    {26.5f, 61.0f, 1180},
    {28.0f, 55.2f, 1240},
};
```

期待する出力：

```
=== センサーデータ解析結果 ===
データ件数  : 5
平均気温    : 26.34 ℃
最高気温    : 28.00 ℃
最低気温    : 24.80 ℃
平均湿度    : 59.40 %
```

---

## 提供ファイル（コピーして使うこと）

### `lib/sensor.h`

```bash
mkdir -p ~/training/day09/lib
mkdir -p ~/training/day09/app
```

```bash
vim ~/training/day09/lib/sensor.h
```

以下をそのまま入力する（変更しないこと）：

```c
#ifndef SENSOR_H
#define SENSOR_H

typedef struct {
    float temperature;
    float humidity;
    int   timestamp;
} SensorData;

float calc_avg_temperature(const SensorData *data, int count);
float calc_max_temperature(const SensorData *data, int count);
float calc_min_temperature(const SensorData *data, int count);
float calc_avg_humidity(const SensorData *data, int count);

#endif /* SENSOR_H */
```

---

## 実施手順

### Step 1：ブランチを作成する

```bash
cd ~/training
git checkout -b feature/day09-integration
```

### Step 2：ディレクトリ構成を作成する

```bash
mkdir -p ~/training/day09/lib
mkdir -p ~/training/day09/app
cd ~/training/day09
```

`sensor.h` を `lib/` に配置する（上記の提供ファイルをコピー）。

### Step 3：`lib/sensor.c` を実装する

```bash
vim lib/sensor.c
```

**ヒント**：

- `float` の配列を `for` ループで回して合計を計算し、要素数で割る
- `count <= 0` の場合のガード条件を忘れずに入れる（仕様を再確認）
- 最大・最小の初期値は配列の最初の要素（`data[0]`）を使う

### Step 4：`app/main.c` を実装する

```bash
vim app/main.c
```

**ヒント**：

- `#include "sensor.h"` または `#include "../lib/sensor.h"` でヘッダをインクルードする
- `printf` のフォーマット文字列：`%.2f` で小数点2桁
- テストデータは仕様の通りにハードコードする
- データ件数は `sizeof(test_data) / sizeof(test_data[0])` で計算できる

### Step 5：CMakeLists.txt を作成する

**トップレベル（`~/training/day09/CMakeLists.txt`）**

```bash
vim CMakeLists.txt
```

Day 8 で学んだ構造を参考に作成する。必要な要素：
- `cmake_minimum_required`
- `project`
- `set(CMAKE_C_STANDARD 17)`
- `add_subdirectory(lib)`
- `add_subdirectory(app)`

**`lib/CMakeLists.txt`**

```bash
vim lib/CMakeLists.txt
```

必要な要素：
- `add_library(sensor STATIC sensor.c)`
- `target_include_directories(sensor PUBLIC ...)` でヘッダの場所を指定

**`app/CMakeLists.txt`**

```bash
vim app/CMakeLists.txt
```

必要な要素：
- `add_executable(sensor_app main.c)`
- `target_link_libraries(sensor_app PRIVATE sensor)`
- `target_compile_options(sensor_app PRIVATE -Wall -Wextra)`

### Step 6：ビルドする

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

エラーが出た場合はメッセージを読んで修正する。よくあるエラーと対処法は下記を参照。

### Step 7：実行して出力を確認する

```bash
./build/app/sensor_app
```

期待する出力と一致することを確認する：

```
=== センサーデータ解析結果 ===
データ件数  : 5
平均気温    : 26.34 ℃
最高気温    : 28.00 ℃
最低気温    : 24.80 ℃
平均湿度    : 59.40 %
```

出力が一致しない場合は Step 8 に進む。

### Step 8：gdb でデバッグする（出力がおかしい場合）

デバッグビルドでビルド済みなので、そのまま gdb で調査できる。

```bash
gdb ./build/app/sensor_app
(gdb) break calc_avg_temperature
(gdb) run
(gdb) print count         # 件数を確認
(gdb) print data[0].temperature   # 1件目のデータを確認
(gdb) next                # ループの中をステップ実行
(gdb) print sum           # 累積値を確認（変数名は実装に合わせる）
```

### Step 9：コンパイル警告をゼロにする

`-Wall -Wextra` で警告が出た場合はすべて修正する。

```bash
cmake --build build 2>&1 | grep "warning:"
```

警告が出なくなるまで修正する。

### Step 10：コミットして提出する

```bash
cd ~/training
git add day09/
git commit -m "day09: センサーデータ処理ライブラリと解析CLIの統合課題を完成"
git push origin feature/day09-integration
```

GitHub でプルリクエストを作成する。

---

## 完了チェックリスト

以下をすべてチェックできたら課題完了。

- [ ] `cmake --build build` がエラー・警告ゼロで完了する
- [ ] `./build/app/sensor_app` の出力が期待値と一致する
- [ ] gdb で `calc_avg_temperature` の中に入ってステップ実行できる
- [ ] `git log --oneline` でコミット履歴が残っている
- [ ] GitHubにプッシュされている

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `cannot find -lsensor` | CMakeLists.txtのライブラリ名が違う | `add_library(sensor ...)` と `target_link_libraries(... sensor)` の名前が一致しているか確認 |
| `sensor.h: No such file or directory` | インクルードパスが設定されていない | `target_include_directories(sensor PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})` が正しく書かれているか確認 |
| `undefined reference to 'calc_avg_temperature'` | `sensor.c` がライブラリに含まれていない | `add_library(sensor STATIC sensor.c)` で `sensor.c` を指定しているか確認 |
| `warning: implicit declaration of function` | `sensor.h` がインクルードされていない | `main.c` の先頭に `#include "sensor.h"` があるか確認 |
| 出力の数値が 0.00 になる | count が 0 または関数の戻り値が間違っている | gdb でブレークポイントを張って count と計算途中の値を確認する |

---

## 研修終了後のOJTに向けて

この研修でカバーしていない、OJT中に習得する内容：

### 実機SBC（Raspberry Pi等）での開発

```bash
# SSHでSBCに接続
ssh pi@192.168.1.x

# Ubuntu VMでビルドしたバイナリをSBCに転送
scp ./build/app/sensor_app pi@192.168.1.x:~/

# SBC上で実行
ssh pi@192.168.1.x ./sensor_app
```

### クロスコンパイル（より高度）

```bash
# ARM向けクロスコンパイラでビルド
sudo apt install gcc-arm-linux-gnueabihf

# CMakeでクロスコンパイルする場合（ツールチェーンファイルを使う）
cmake -B build_arm \
    -DCMAKE_TOOLCHAIN_FILE=arm_toolchain.cmake \
    -DCMAKE_BUILD_TYPE=Release
cmake --build build_arm
```

### デバイスファイルへのアクセス

```c
#include <fcntl.h>
#include <unistd.h>

int fd = open("/dev/ttyUSB0", O_RDWR);   /* シリアルポートを開く */
```

# Day 7 — Makefile

## この研修全体の流れ

```
Day 1  → Day 2  → Day 3  → Day 4  → Day 5  → Day 6  → [Day 7] → Day 8  → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
                                                      ★今日
```

Day 5〜6 でビルドとデバッグの方法を学びました。今日は「ビルドの自動化・効率化」を担うMakefileを学びます。Day 5 で手動で実行していた `gcc -c calc.c -o calc.o` などのコマンドを、`make` の1コマンドで実行できるようにします。

---

## この日のゴール

- Makefileの基本構文（ターゲット・依存関係・レシピ）を理解できる
- 自動変数（`$@`, `$<`, `$^`）を使いこなせる
- パターンルールで複数ファイルを一括処理できる
- `clean` ターゲットでビルド成果物を削除できる
- 差分コンパイルの仕組みを理解できる

---

## この日の前提

- Day 5 の分割コンパイルの実習が完了していること
- `make --version` でバージョンが表示されること

---

## 1. Makefileとは何か・なぜ必要か

### Visual Studioのプロジェクトファイルとの対比

Visual Studio では `.vcxproj`（プロジェクトファイル）が「どのファイルをどのオプションでコンパイルするか」を管理する。LinuxではMakefileがその役割を担う。

| Visual Studio | Makefile |
|-------------|---------|
| `.vcxproj` | `Makefile` |
| F7（ビルド） | `make` |
| ビルド→クリーン | `make clean` |
| デバッグビルド | `make debug` |
| リリースビルド | `make release` |

### 差分コンパイルの仕組み

Makefileの最大の利点は**変更されたファイルだけを再コンパイルする**こと。

```
10ファイルあるプロジェクトで1ファイルだけ変更した場合：

make なし：10ファイルすべてをコンパイル（毎回全体をビルド）
make あり：変更した1ファイルだけをコンパイル（差分ビルド）
```

Makefileはファイルのタイムスタンプを比較して「どのファイルを再コンパイルすべきか」を判断する。

---

## 2. Makefileの基本構文

```makefile
ターゲット: 依存ファイル1 依存ファイル2
	レシピ（コマンド）
```

> **重要**：レシピの先頭は**タブ文字**でなければならない。スペースではエラーになる。

### 最もシンプルなMakefile

```makefile
# 最終的に作りたいファイル（ターゲット）: 必要なファイル（依存関係）
myapp: main.c
	gcc main.c -o myapp
```

`make` を実行すると：
1. `myapp` が存在しない → コンパイルが走る
2. `myapp` が存在して `main.c` より新しい → 何もしない（Already up to date）
3. `main.c` が `myapp` より新しい → 再コンパイル

---

## 3. 変数の定義

```makefile
CC = gcc                    # C コンパイラ
CXX = g++                   # C++ コンパイラ
CFLAGS = -Wall -Wextra -g   # コンパイルフラグ
LDFLAGS = -lm               # リンクフラグ（-lm は数学ライブラリ）
TARGET = myapp
SRCS = main.c calc.c
OBJS = main.o calc.o

$(TARGET): $(OBJS)
	$(CC) $(OBJS) -o $(TARGET) $(LDFLAGS)

main.o: main.c
	$(CC) $(CFLAGS) -c main.c -o main.o

calc.o: calc.c
	$(CC) $(CFLAGS) -c calc.c -o calc.o
```

### なぜ変数を使うのか

コンパイラやフラグを変更するとき、変数なしだと全行を書き換える必要がある。変数を使えば1行変えるだけで全体に反映される。

---

## 4. 自動変数

レシピの中で使える特殊な変数。これを使うと、ターゲット名や依存ファイル名をハードコードしなくてよい。

| 変数 | 意味 | 例 |
|------|------|-----|
| `$@` | ターゲット名 | ターゲットが `myapp` なら `$@` = `myapp` |
| `$<` | 最初の依存ファイル | 依存が `main.c calc.c` なら `$<` = `main.c` |
| `$^` | すべての依存ファイル | `main.c calc.c` |

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -g

myapp: main.o calc.o
	$(CC) $^ -o $@          # $^ = main.o calc.o, $@ = myapp

main.o: main.c
	$(CC) $(CFLAGS) -c $< -o $@   # $< = main.c, $@ = main.o

calc.o: calc.c
	$(CC) $(CFLAGS) -c $< -o $@
```

---

## 5. パターンルール

同じ処理を繰り返し書かなくて済む。`%.o: %.c` は「任意の `.c` ファイルから `.o` を作る」という汎用ルール。

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -g
TARGET = myapp
OBJS = main.o calc.o

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

# パターンルール：任意の .c を .o にコンパイルする
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(TARGET) $(OBJS)
```

`calc.o` と `main.o` のルールを個別に書く必要がなくなった。

---

## 6. `clean` ターゲット

ビルド成果物（`.o`, 実行ファイル）を削除するターゲット。

```makefile
clean:
	rm -f $(TARGET) $(OBJS)
```

実行：

```bash
make clean      # ビルド成果物を削除
make            # 最初からビルド
```

### フォニーターゲット（.PHONY）

`clean` はファイルを作らない「擬似ターゲット」。もし `clean` という名前のファイルが存在すると、makeが「clean は最新だ」と判断して何もしなくなる。`.PHONY` で防ぐ。

```makefile
.PHONY: all clean

all: $(TARGET)

clean:
	rm -f $(TARGET) $(OBJS)
```

---

## 7. 依存関係の自動生成

ヘッダファイル（`.h`）が変更されたとき、それをインクルードしている `.c` ファイルも再コンパイルしなければならない。依存関係を手動で書くのは大変なため、gccに自動生成させる。

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -g -MMD -MP
TARGET = myapp
SRCS = main.c calc.c
OBJS = $(SRCS:.c=.o)
DEPS = $(OBJS:.o=.d)      # 依存関係ファイル

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

-include $(DEPS)           # 依存関係ファイルを読み込む（-include はファイルがなくてもエラーにしない）

.PHONY: clean
clean:
	rm -f $(TARGET) $(OBJS) $(DEPS)
```

`-MMD -MP` フラグを追加すると、コンパイル時に `.d` ファイル（依存関係ファイル）が自動生成される。例：`calc.d`:

```makefile
calc.o: calc.c calc.h
```

ヘッダ `calc.h` が変更されると `calc.o` が再コンパイルされる。

---

## 8. デバッグ・リリースビルドの切り替え

```makefile
CC = gcc
TARGET = myapp
SRCS = main.c calc.c
OBJS = $(SRCS:.c=.o)

# DEBUG=1 で make すると -g -O0、それ以外は -O2
ifdef DEBUG
    CFLAGS = -Wall -Wextra -g -O0 -DDEBUG
else
    CFLAGS = -Wall -Wextra -O2
endif

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean debug release
clean:
	rm -f $(TARGET) $(OBJS)

debug:
	$(MAKE) DEBUG=1

release:
	$(MAKE)
```

使い方：

```bash
make debug      # デバッグビルド（-g -O0）
make release    # リリースビルド（-O2）
make clean      # クリーン
```

---

## 9. ハンズオン

### Step 1：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day07
cd ~/training/day07
```

### Step 2：Day 5 のファイルをコピーする

```bash
cp ~/training/day05/02_split/calc.h .
cp ~/training/day05/02_split/calc.c .
cp ~/training/day05/02_split/main.c .
```

### Step 3：最初のMakefileを作成する

```bash
vim Makefile
```

まず最もシンプルな形から始める：

```makefile
myapp: main.c calc.c
	gcc -Wall -g main.c calc.c -o myapp

clean:
	rm -f myapp
```

実行して動作確認：

```bash
make
./myapp
make clean
ls          # myapp が消えていることを確認
```

### Step 4：分割コンパイルに対応したMakefileに改善する

```bash
vim Makefile
```

以下に書き換える：

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -g -O0
TARGET = calculator
OBJS = main.o calc.o

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean
clean:
	rm -f $(TARGET) $(OBJS)
```

```bash
make
./calculator
```

差分コンパイルの動作を確認する：

```bash
# 1回目のビルド
make
# 出力：gcc -Wall ... -c main.c -o main.o
#       gcc -Wall ... -c calc.c -o calc.o
#       gcc main.o calc.o -o calculator

# 変更なしで2回目
make
# 出力：make: 'calculator' is up to date （何もしない）

# calc.c だけ変更してから
touch calc.c
make
# 出力：gcc -Wall ... -c calc.c -o calc.o   （calc.c だけ再コンパイル）
#       gcc main.o calc.o -o calculator
```

### Step 5：依存関係の自動生成を追加する

```bash
vim Makefile
```

`-MMD -MP` を追加した版に書き換える：

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -g -O0 -MMD -MP
TARGET = calculator
SRCS = main.c calc.c
OBJS = $(SRCS:.c=.o)
DEPS = $(OBJS:.o=.d)

$(TARGET): $(OBJS)
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

-include $(DEPS)

.PHONY: clean
clean:
	rm -f $(TARGET) $(OBJS) $(DEPS)
```

```bash
make
ls *.d              # main.d calc.d が生成される
cat calc.d          # 依存関係が書かれている
```

`calc.h` を変更したとき `calc.o` が再コンパイルされることを確認：

```bash
touch calc.h
make
# calc.c が calc.h に依存しているため、calc.o が再コンパイルされる
```

### Step 6：デバッグ・リリースビルドを追加する

セクション8のMakefileを作成して動作確認する：

```bash
make debug          # デバッグビルド
file calculator     # デバッグシンボルが含まれていることを確認
# calculator: ELF 64-bit ... with debug_info, not stripped

make clean
make release        # リリースビルド
file calculator
# calculator: ELF 64-bit ... stripped
```

### Step 7：今日の作業をコミットする

```bash
cd ~/training
git add day07/
git commit -m "day07: Makefileの基本構文・パターンルール・依存関係自動生成を学習"
git push origin main
```

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `Makefile:3: *** missing separator. Stop.` | レシピの先頭がスペースになっている（タブが必要） | Vimで `:set list` でタブとスペースを区別表示。`^I` がタブ、`·` がスペース |
| `make: Nothing to be done for 'all'` | すべてのターゲットが最新 | `make clean && make` で最初からビルド |
| `make: 'clean' is up to date` | `clean` という名前のファイルが存在する | `.PHONY: clean` を Makefile に追加する |
| `No rule to make target 'calc.o'` | `.c` ファイルが見つからない | ファイル名のスペルを確認。`ls *.c` でファイルを確認 |
| リンクエラー `undefined reference` | OBJS に `.o` が含まれていない | `OBJS` 変数にすべての `.o` が含まれているか確認 |

---

## この日の Git チェックポイント

### いつコミットするか

| タイミング | 理由 |
|-----------|------|
| Step 3 完了（make が動く） | 最初の動くMakefileを記録する |
| Step 4 完了（差分コンパイル確認） | 「差分コンパイルが動く」状態を記録 |
| Step 5 完了（依存関係自動生成） | 本格的なMakefileが完成した状態を記録 |

### コミットメッセージ例

```bash
git commit -m "day07: 最初のMakefile（シングルターゲット版）を作成"
git commit -m "day07: パターンルールと自動変数に書き換え"
git commit -m "day07: -MMDフラグでヘッダ依存関係を自動生成"
```

# Day 1 — Linuxファイルシステム・基本コマンド

## この研修全体の流れ

```
[Day 1] → Day 2  → Day 3  → Day 4  → Day 5  → Day 6  → Day 7  → Day 8  → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
 ★今日
```

事前準備でUbuntuが使える状態になりました。今日は「Linuxというシステムのどこに何があるか」を理解し、ファイル操作の基本コマンドを習得します。Windows開発者がLinuxで最初に戸惑う「ディレクトリ構造の違い」を理解することが今日の核心です。

---

## この日のゴール

- `/etc`, `/var`, `/dev`, `/proc` 各ディレクトリの役割を説明できる
- `ls`, `cd`, `mkdir`, `cp`, `mv`, `rm`, `find`, `grep` を実務で使えるレベルで操作できる
- Windowsのパスとの違いを説明できる

---

## この日の前提

- `doc/00_setup.md` の手順が完了していること（UbuntuへのSSH接続ができる状態）

---

## 1. Linuxファイルシステムの全体像

### Windowsとの最大の違い：ドライブレターがない

Windowsでは `C:\Users\username\Desktop` のようにドライブレター（`C:`）から始まる。Linuxには**ドライブレターがない**。すべてのファイルは `/`（ルート）から始まる1本の木構造に格納される。

```
Windows                    Linux
C:\                        /（ルート）
C:\Users\                  /home/
C:\Users\username\         /home/user/
C:\Program Files\          /usr/local/
C:\Windows\System32\       /usr/lib/
C:\Windows\                /etc/, /var/, /usr/
```

### なぜドライブレターがないのか

Linuxでは、HDDやUSBメモリなどの別デバイスも「マウント」という操作でディレクトリの一部として組み込む。

```
/（ルートディレクトリ）
└── /mnt/usb/    ← USBメモリを /mnt/usb にマウントするとここでアクセスできる
```

組み込み開発との関連：SBCでも `/dev/sda1` のようなデバイスファイルをマウントして使うため、この概念は重要。

### 主要ディレクトリの役割

```
/
├── home/         ユーザーのホームディレクトリ（Windowsの C:\Users\ に相当）
│   └── user/     ← あなたのホームディレクトリ（~ で表記できる）
├── etc/          設定ファイル（サービスの設定、hosts ファイルなど）
├── var/          変動するデータ（ログ /var/log/、一時ファイルなど）
├── usr/          ユーザー向けプログラム（Windowsの C:\Program Files\ に相当）
│   ├── bin/      一般コマンド（ls, grep など）
│   ├── lib/      ライブラリ（.so ファイル。Windowsの .dll に相当）
│   └── local/    手動インストールしたソフト
├── bin/          基本コマンド（/usr/bin へのシンボリックリンク）
├── dev/          デバイスファイル（HDD、USB、シリアルポートなど）
├── proc/         実行中プロセスの情報（カーネルが動的に生成する仮想ファイル）
├── sys/          カーネル・デバイスの設定（/proc と似た仮想ファイルシステム）
├── tmp/          一時ファイル（再起動で消える）
└── root/         rootユーザーのホームディレクトリ（/home/root ではない）
```

### 組み込み開発で特に重要なディレクトリ

| ディレクトリ | 組み込み開発での用途 |
|------------|-----------------|
| `/dev/` | `ttyUSB0`（シリアル）、`i2c-1`（I2C）などデバイスファイルへのアクセス |
| `/proc/` | `cpuinfo`（CPUアーキテクチャ確認）、`meminfo`（メモリ確認）など |
| `/sys/` | GPIOの操作、デバイスドライバの設定 |
| `/etc/` | ネットワーク設定、起動スクリプト |

---

## 2. パスの書き方

### 絶対パスと相対パス

```bash
# 絶対パス：/ から始まる完全なパス
/home/user/projects/hello.c

# 相対パス：現在地（カレントディレクトリ）からの相対位置
./hello.c         # 現在のディレクトリにある hello.c
../hello.c        # 1つ上のディレクトリにある hello.c
../../etc/hosts   # 2つ上に戻って etc/hosts
```

### 特殊な表記

| 表記 | 意味 | 例 |
|------|------|----|
| `~` | ホームディレクトリ | `~/projects` = `/home/user/projects` |
| `.` | カレントディレクトリ | `./a.out` = 現在地の a.out |
| `..` | 1つ上のディレクトリ | `cd ..` で1つ上に移動 |

---

## 3. 基本コマンドの解説

### `ls`：ディレクトリの内容を表示する

Windowsの `dir` コマンドに相当。

```bash
ls              # カレントディレクトリの内容を表示
ls /etc         # /etc の内容を表示
ls -l           # 詳細表示（パーミッション、サイズ、更新日時）
ls -la          # 隠しファイル（.で始まるファイル）も含めて詳細表示
ls -lh          # ファイルサイズを人間が読みやすい形式で表示（KB, MB など）
```

`ls -l` の出力の読み方：

```
drwxr-xr-x 2 user user 4096  1月 1 10:00 projects
-rw-r--r-- 1 user user  128  1月 1 10:00 hello.c
│           │ │    │     │               └── ファイル名
│           │ │    │     └── ファイルサイズ（バイト）
│           │ │    └── グループ
│           │ └── オーナー（所有者）
│           └── リンク数
└── パーミッション（Day 4 で詳しく学ぶ）
    d = ディレクトリ、- = 通常ファイル
```

### `cd`：ディレクトリを移動する

Windowsの `cd` コマンドと同じ操作感。

```bash
cd /etc         # /etc に移動
cd ~            # ホームディレクトリに戻る（cd だけでも同じ）
cd ..           # 1つ上のディレクトリに移動
cd -            # 直前のディレクトリに戻る（便利）
```

### `pwd`：現在地を確認する

Print Working Directory の略。迷子になったときに使う。

```bash
pwd
# 出力例：/home/user/projects
```

### `mkdir`：ディレクトリを作成する

```bash
mkdir projects              # projects ディレクトリを作成
mkdir -p a/b/c              # 中間のディレクトリも含めて一括作成（-p が重要）
```

`-p` オプションなしで存在しないディレクトリの中にディレクトリを作ろうとするとエラーになる：

```bash
mkdir a/b/c    # a が存在しない場合：mkdir: cannot create directory 'a/b/c': No such file or directory
mkdir -p a/b/c # -p をつけると a, a/b, a/b/c を一括作成する
```

### `cp`：ファイル・ディレクトリをコピーする

```bash
cp hello.c hello_backup.c           # ファイルをコピー
cp hello.c /tmp/hello.c             # 別ディレクトリにコピー
cp -r projects/ projects_backup/    # ディレクトリを再帰的にコピー（-r が必要）
```

### `mv`：ファイル・ディレクトリを移動 / 名前変更する

```bash
mv hello.c main.c           # ファイル名を変更（Windowsの rename に相当）
mv hello.c /tmp/hello.c     # ファイルを移動
mv projects/ old_projects/  # ディレクトリの名前を変更（-r 不要）
```

### `rm`：ファイル・ディレクトリを削除する

**ゴミ箱に入らない。削除したら元に戻せない。** 慎重に使うこと。

```bash
rm hello.c              # ファイルを削除
rm -r projects/         # ディレクトリを再帰的に削除
rm -i hello.c           # 削除前に確認を求める（推奨）
```

> **注意**：`rm -rf /` は全ファイルを削除する危険なコマンド。`rm -rf` を使う際は対象パスを必ず確認すること。

### `cat`：ファイルの内容を表示する

```bash
cat /etc/hostname       # ホスト名のファイルを表示
cat hello.c             # ソースファイルを表示
```

### `less`：長いファイルをスクロールして表示する

```bash
less /var/log/syslog    # ログファイルをスクロール表示
```

操作方法：
- `j` / `k` または矢印キー：上下スクロール
- `Space` / `b`：1ページ下 / 上
- `/keyword`：検索（nで次の候補へ）
- `q`：終了

---

## 4. 検索コマンド

### `find`：ファイルを検索する

Windowsのエクスプローラーの検索に相当するが、コマンドラインで柔軟に使える。

```bash
# 基本構文：find [検索場所] [条件]
find /home/user -name "*.c"         # .c ファイルを検索
find /tmp -name "*.log" -mtime +7   # 7日以上前に更新された .log ファイル
find . -type d                       # カレントディレクトリ以下のディレクトリのみ
find . -type f -size +1M             # 1MB以上のファイル
```

### `grep`：ファイルの中身を検索する

ファイルの中から特定の文字列を含む行を探す。組み込み開発では「このヘッダファイルでこの関数が定義されているか」などの調査に頻繁に使う。

```bash
grep "main" hello.c                 # hello.c の中で "main" を含む行を表示
grep -r "malloc" /usr/include/      # /usr/include 以下を再帰検索
grep -n "printf" hello.c            # 行番号も表示
grep -i "error" build.log           # 大文字小文字を区別しない
grep -v "DEBUG" app.log             # マッチしない行を表示（逆検索）
```

---

## 5. ハンズオン

### Step 1：SSH で接続する

Windows Terminal を開いて接続する。

```powershell
ssh user@127.0.0.1 -p 2222
```

### Step 2：現在地とホームディレクトリを確認する

```bash
pwd
ls -la
```

期待する出力（例）：

```
/home/user
total 32
drwxr-x--- 4 user user 4096 Jan  1 10:00 .
drwxr-xr-x 3 root root 4096 Jan  1 09:00 ..
-rw-r--r-- 1 user user  220 Jan  1 09:00 .bash_logout
-rw-r--r-- 1 user user 3526 Jan  1 09:00 .bashrc
-rw-r--r-- 1 user user  807 Jan  1 09:00 .profile
```

`.bash_logout`, `.bashrc`, `.profile` はBashの設定ファイル（`.` で始まるため「隠しファイル」）。

### Step 3：主要ディレクトリを探索する

```bash
ls /
ls /etc
ls /dev
```

`/dev` の中に `tty`、`null`、`zero` などのデバイスファイルが見える。

```bash
ls /proc
cat /proc/cpuinfo
cat /proc/meminfo
```

`/proc/cpuinfo` はVMのCPU情報を仮想ファイルとして提供している。実機SBCではARMアーキテクチャの情報が表示される。

### Step 4：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day01
cd ~/training/day01
pwd
```

### Step 5：ファイルを作成・操作する

```bash
# ファイルを作成（touch：空のファイルを作る）
touch hello.c main.c

# 一覧確認
ls -l

# ファイルをコピー
cp hello.c hello_backup.c

# ファイル名を変更
mv hello.c program.c

# 現在の状態を確認
ls -l

# ファイルを削除
rm hello_backup.c

# 最終確認
ls -l
```

期待する状態：`program.c` と `main.c` だけが残っている。

### Step 6：検索コマンドを試す

```bash
# /etc 配下の .conf ファイルを探す
find /etc -name "*.conf" 2>/dev/null | head -10
```

> `2>/dev/null` は「エラーメッセージを捨てる」という意味。権限のないディレクトリのエラーを非表示にする。`| head -10` は最初の10件だけ表示。詳細はDay 3で学ぶ。

```bash
# /usr/include の中で "malloc" が定義されているヘッダを探す
grep -r "void \*malloc" /usr/include/stdlib.h
```

出力例：

```
/usr/include/stdlib.h:extern void *malloc (size_t __size) __THROW __attribute_malloc__
```

### Step 7：Git のユーザー設定をする

Day 4 以降でGitを使うため、今日のうちに設定しておく。

```bash
git config --global user.name "あなたの名前"
git config --global user.email "your-email@example.com"
git config --global core.editor vim

# 設定確認
git config --list
```

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `No such file or directory` | パスが間違っている or ファイルが存在しない | `ls` で内容を確認し、パスのスペルを確認する |
| `Permission denied` | そのファイル・ディレクトリへのアクセス権がない | `sudo` をつけて実行するか、パーミッションを確認（Day 4で詳しく学ぶ） |
| `mkdir: cannot create directory 'a/b': No such file or directory` | 中間ディレクトリが存在しない | `mkdir -p` を使う |
| `rm: cannot remove 'projects': Is a directory` | ディレクトリを `rm` で削除しようとした | `rm -r projects/` のように `-r` をつける |
| `bash: cd: /nonexistent: No such file or directory` | 存在しないディレクトリに移動しようとした | `ls` で存在確認してから `cd` する |

---

## この日の Git チェックポイント

この研修では、各日の課題をGitでコミットして記録する。

### Step 1：リポジトリを初期化する

```bash
cd ~/training
git init
git branch -M main
```

### Step 2：.gitignore を作成する

```bash
cat > .gitignore << 'EOF'
*.o
*.a
*.out
build/
EOF
```

### Step 3：今日の作業をコミットする

```bash
git add day01/
git commit -m "day01: Linuxファイルシステムの基本コマンドを学習"
```

### いつコミットするか

| タイミング | 理由 |
|-----------|------|
| 各日の課題が完了したとき | 「その日に学んだことが実行できた」状態を記録する |
| 手順途中で詰まって別の方法を試すとき | 試行前の動作する状態に戻せるようにする |

### コミットメッセージ例

```bash
git commit -m "day01: Linuxファイルシステムの基本コマンドを学習"
git commit -m "day01: find/grepコマンドの実習を完了"
```

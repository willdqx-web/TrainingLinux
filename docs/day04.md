# Day 4 — パーミッション・パッケージ管理・Git CLI

## この研修全体の流れ

```
Day 1  → Day 2  → Day 3  → [Day 4] → Day 5  → Day 6  → Day 7  → Day 8  → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
                            ★今日
```

Day 3 でシェル操作の基本ができました。今日は「Linuxの権限管理（パーミッション）」「パッケージのインストール方法」「コマンドラインでのGit操作」を学びます。特にパーミッションは組み込み開発でデバイスファイルにアクセスする際に必ず必要になる知識です。

---

## この日のゴール

- `ls -l` のパーミッション表示を読んで意味を説明できる
- `chmod`, `chown` でファイルの権限を変更できる
- `sudo` と root の違いを説明できる
- `apt` でパッケージをインストール・削除できる
- `git clone`, `add`, `commit`, `push`, `branch`, `checkout` をコマンドラインで実行できる

---

## この日の前提

- Day 3 の作業が完了していること
- GitHubのアカウントを持っていること（持っていない場合は今日作成する）

---

## 1. パーミッション（ファイルの権限）

### パーミッションとは

Linuxでは**すべてのファイルに「誰が何をできるか」の権限**が設定されている。複数ユーザーが同じシステムを使うため、ファイルへのアクセスを制御する仕組みが必要。

組み込み開発との関連：
- `/dev/ttyUSB0`（シリアルポート）にアクセスするには適切なグループに属している必要がある
- ビルド済みの実行ファイルに「実行権限」が必要
- デバイスドライバの設定ファイルはroot権限が必要

### `ls -l` の読み方

```
-rw-r--r-- 1 user group 1024 Jan 1 10:00 main.c
│├────────┘ │ │    │
││          │ │    └── グループ
││          │ └── オーナー（所有者）
││          └── リンク数
│└── パーミッション（9文字）
└── ファイルタイプ（- = 通常ファイル、d = ディレクトリ、l = シンボリックリンク）
```

**パーミッションの9文字**

```
rw-  r--  r--
│     │    └── others（その他のユーザー）の権限
│     └── group（グループ）の権限
└── owner（所有者）の権限

r = read（読み取り）
w = write（書き込み）
x = execute（実行）
- = 権限なし
```

### 数値表記

| 権限 | 数値 |
|------|------|
| r (read) | 4 |
| w (write) | 2 |
| x (execute) | 1 |
| なし | 0 |

各グループの権限を足し算する：

| 表記 | 数値 | 意味 |
|------|------|------|
| `rwx` | 7 (4+2+1) | 読み・書き・実行すべて可 |
| `rw-` | 6 (4+2+0) | 読み・書き可、実行不可 |
| `r--` | 4 (4+0+0) | 読み取りのみ |
| `---` | 0 (0+0+0) | すべて不可 |

例：`chmod 644 main.c` = `rw-r--r--`（所有者は読み書き、グループと他者は読み取りのみ）

### `chmod`：パーミッションを変更する

```bash
# 数値表記
chmod 644 main.c        # -rw-r--r--（ソースファイルの一般的な設定）
chmod 755 build.sh      # -rwxr-xr-x（実行スクリプトの一般的な設定）
chmod 600 secret.key    # -rw-------（秘密鍵など）

# シンボル表記
chmod +x build.sh       # 実行権限を追加
chmod -w readonly.txt   # 書き込み権限を削除
chmod u+x,g-w file      # 所有者に実行権限追加、グループから書き込み権限削除

# ディレクトリを再帰的に変更
chmod -R 755 projects/
```

### `chown`：所有者を変更する

```bash
chown user:group main.c         # 所有者とグループを変更
chown user main.c               # 所有者だけ変更
chown :group main.c             # グループだけ変更
sudo chown -R user:user /opt/   # 再帰的に変更（root権限が必要）
```

### `sudo` と root の概念

**root** はLinuxの最高権限ユーザー（WindowsのAdministratorに相当）。なんでもできる代わりに、誤操作が致命的になる。

**sudo** は「一時的にroot権限で特定のコマンドを実行する」仕組み。

```bash
sudo apt install vim        # root権限で apt を実行
sudo systemctl start nginx  # root権限でサービスを起動
sudo vim /etc/hosts         # root権限でシステムファイルを編集
```

組み込み開発での典型的なパターン：

```bash
# デバイスファイルへのアクセスにsudoが必要な場合
sudo ls -l /dev/ttyUSB0

# 解決策：自分をdialoutグループに追加する（次回ログインから有効）
sudo usermod -aG dialout user
```

### シンボリックリンク

Windowsのショートカット（.lnkファイル）に相当するが、より透過的に機能する。

```bash
ln -s /usr/local/arm-gcc/bin/arm-linux-gnueabihf-gcc /usr/local/bin/arm-gcc
# arm-gcc というコマンドで長いパスのツールを呼び出せるようになる
```

組み込み開発では、クロスコンパイラへのシンボリックリンクをよく使う。

---

## 2. `apt` パッケージ管理

### パッケージ管理とは

Windows では `.exe` や `.msi` ファイルを手動でダウンロードしてインストールする。Linuxでは**パッケージマネージャー（apt）**が依存関係も含めて自動的にインストール・管理する。

```
apt install vim
  ↓
apt: "vimはvim-commonにも依存している → 一緒にインストールしよう"
  ↓
必要なパッケージをすべて自動でインストール
```

### よく使う apt コマンド

```bash
# パッケージ一覧を更新（まず必ずこれ）
sudo apt update

# パッケージをインストール
sudo apt install vim
sudo apt install -y vim     # -y で確認プロンプトを省略

# パッケージを削除
sudo apt remove vim

# 設定ファイルも含めて完全に削除
sudo apt purge vim

# インストール済みパッケージの更新
sudo apt upgrade

# パッケージを検索
apt search "gcc"
apt search --names-only "gcc"   # パッケージ名だけで検索

# パッケージの詳細情報を表示
apt show gcc

# インストール済みパッケージを一覧表示
apt list --installed
apt list --installed | grep gcc
```

### 組み込み開発でよく使うパッケージ

```bash
# クロスコンパイラ（ARM向け）
sudo apt install gcc-arm-linux-gnueabihf
sudo apt install g++-arm-linux-gnueabihf

# ビルドツール
sudo apt install cmake make

# デバッグツール
sudo apt install gdb valgrind strace

# 静的解析
sudo apt install cppcheck

# シリアル通信
sudo apt install picocom minicom
```

---

## 3. Git CLI（コマンドラインでのGit操作）

### GUIクライアントからの移行

Windowsでは GitKrakenや GitHub Desktop のGUIクライアントを使っていた場合、SSH接続先ではGUIが使えない。**コマンドラインでのGit操作が必須**になる。

| GUIクライアントの操作 | git コマンド |
|--------------------|-----------| 
| リポジトリをクローン | `git clone URL` |
| 変更をステージング | `git add ファイル名` |
| コミット | `git commit -m "メッセージ"` |
| プッシュ | `git push origin ブランチ名` |
| プル | `git pull origin ブランチ名` |
| ブランチ作成 | `git checkout -b ブランチ名` |
| ブランチ切り替え | `git checkout ブランチ名` |
| 差分確認 | `git diff` |
| 状態確認 | `git status` |

### 基本的なワークフロー

```
作業前                 作業中                   作業後
──────────────────────────────────────────────────────
git checkout -b        [ファイルを編集]         git add .
feature/xxx                                     git commit -m "..."
                                                git push origin feature/xxx
```

### SSH キーの設定（GitHub との接続）

```bash
# SSHキーを生成する
ssh-keygen -t ed25519 -C "your-email@example.com"
# パスフレーズは空欄でも可（Enter を2回）

# 公開鍵を表示する（これをGitHubに登録する）
cat ~/.ssh/id_ed25519.pub
```

表示されたテキスト（`ssh-ed25519 AAAA...` から始まる1行）を、GitHubの Settings → SSH and GPG keys → New SSH key に貼り付ける。

```bash
# 接続確認
ssh -T git@github.com
# 成功例：Hi username! You've successfully authenticated...
```

### よく使う git コマンド

```bash
# リポジトリの状態を確認する
git status

# 差分を確認する
git diff                    # ステージング前の差分
git diff --staged           # ステージング後の差分

# 変更をステージングする
git add main.c              # 特定のファイル
git add src/                # ディレクトリ全体
git add .                   # カレントディレクトリ以下すべて

# コミットする
git commit -m "feat: main 関数を実装"

# リモートにプッシュする
git push origin main
git push origin feature/add-sensor    # ブランチ名を指定

# 履歴を確認する
git log                     # 詳細
git log --oneline           # 1行表示
git log --oneline --graph   # ブランチのグラフも表示

# ブランチを操作する
git branch                  # ブランチ一覧
git checkout -b feature/xxx # ブランチを作成して切り替え
git checkout main           # main ブランチに切り替え
git branch -d feature/xxx   # ブランチを削除
```

### .gitignore の重要性

ビルド成果物（`.o`, `.a`, `a.out` など）をGitで管理してはいけない。コンパイル環境が違うと動かないファイルであり、リポジトリが肥大化する原因になる。

```bash
# 組み込みC/C++開発用の .gitignore
cat > ~/.gitignore_global << 'EOF'
*.o
*.a
*.out
*.elf
*.bin
*.hex
build/
.cache/
EOF

git config --global core.excludesfile ~/.gitignore_global
```

---

## 4. ハンズオン

### Step 1：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day04
cd ~/training/day04
```

### Step 2：パーミッションを確認・変更する

```bash
# テスト用ファイルを作成
echo "#!/bin/bash" > test.sh
echo "echo Hello" >> test.sh

# 実行権限がないことを確認
ls -l test.sh
# -rw-rw-r-- 1 user user 20 Jan 1 10:00 test.sh

# 実行しようとするとエラー
./test.sh
# bash: ./test.sh: Permission denied

# 実行権限を追加
chmod +x test.sh
ls -l test.sh
# -rwxrwxr-x 1 user user 20 Jan 1 10:00 test.sh

# 実行できるようになる
./test.sh
# Hello
```

### Step 3：デバイスファイルのパーミッションを確認する

```bash
ls -l /dev/null /dev/zero /dev/random
```

出力例：

```
crw-rw-rw- 1 root root 1, 3 Jan 1 09:00 /dev/null
crw-rw-rw- 1 root root 1, 5 Jan 1 09:00 /dev/zero
crw-rw-rw- 1 root root 1, 8 Jan 1 09:00 /dev/random
```

`c` はキャラクタデバイスを表す（`b` はブロックデバイス）。

### Step 4：パッケージを追加インストールする

```bash
# cppcheck（C/C++の静的解析ツール）をインストール
sudo apt update
sudo apt install -y cppcheck

# インストール確認
cppcheck --version

# インストール済み確認
apt list --installed | grep cppcheck
```

### Step 5：GitHubにSSHキーを設定する（まだの場合）

```bash
# SSHキーを生成
ssh-keygen -t ed25519 -C "your-email@example.com"

# 公開鍵を表示
cat ~/.ssh/id_ed25519.pub
```

表示された内容をGitHubに登録し、接続確認：

```bash
ssh -T git@github.com
```

### Step 6：GitHubにリポジトリを作成してプッシュする

1. GitHubで `linux-practice` リポジトリを作成（Public）
2. ローカルのリポジトリをGitHubに接続する：

```bash
cd ~/training

# リモートを追加（URLはGitHubのリポジトリページから確認）
git remote add origin git@github.com:あなたのユーザー名/linux-practice.git

# 現在の状態を確認
git remote -v

# プッシュ
git push -u origin main
```

GitHubのリポジトリページを開いて、ファイルが反映されていれば成功。

### Step 7：ブランチを切って作業する

```bash
# 新しいブランチを作成
git checkout -b feature/day04-permission

# ファイルを追加
mkdir day04
cp test.sh day04/

# コミット
git add day04/
git commit -m "day04: パーミッションとGit CLIの実習を追加"

# プッシュ
git push origin feature/day04-permission
```

GitHubのリポジトリで Pull Request を作成できることを確認する。

---

## よくあるエラーと対処法

| エラー | 原因 | 対処 |
|--------|------|------|
| `Permission denied (publickey)` | SSHキーがGitHubに登録されていない or 鍵ファイルが見つからない | `cat ~/.ssh/id_ed25519.pub` の内容をGitHubに再登録する |
| `git push` で `rejected` | リモートに自分より新しいコミットがある | `git pull --rebase origin main` で最新を取得してからpush |
| `chmod: changing permissions of '...': Operation not permitted` | 他ユーザーのファイルのパーミッションを変更しようとした | `sudo chmod` を使う（root権限が必要） |
| `sudo apt install` でロックエラー | 別の apt プロセスが実行中 | `sudo rm /var/lib/dpkg/lock-frontend` で解除（他のaptが終わるのを待つのが安全） |
| `git status` で `fatal: not a git repository` | gitリポジトリ外でgitコマンドを実行した | `git init` でリポジトリを初期化するか、正しいディレクトリに移動する |

---

## この日の Git チェックポイント

### いつブランチを切るか

**タイミング**：機能や学習の単位ごとに切る。

```bash
git checkout -b feature/day04-permission    # Day 4 の実習
git checkout -b feature/add-sensor-driver   # センサードライバを追加する作業（OJT想定）
```

### コミットメッセージ例

```bash
git commit -m "day04: パーミッションの基本操作を確認"
git commit -m "day04: apt でcppcheckをインストール"
git commit -m "day04: GitHubへのSSH接続を設定"
```

### いつ PR をマージするか

**条件**：その日のハンズオンが完了し、ファイルが正しくコミットされていること。

GitHub でPRを作成し、変更内容を確認してからマージする。

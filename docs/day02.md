# Day 2 — Vim・Bash基本設定

## この研修全体の流れ

```
Day 1  → [Day 2] → Day 3  → Day 4  → Day 5  → Day 6  → Day 7  → Day 8  → Day 9
ファイル  Vim・    パイプ・  パーミ   gcc/     gdb      Make    CMake   統合
システム  Bash     プロセス  ッション g++      デバッグ  file           課題
          ★今日
```

Day 1 でファイルの場所はわかるようになりました。今日は「ターミナル上でファイルを編集する方法」と「Bashを自分好みに設定する方法」を学びます。

SSH接続先（本番のSBCやサーバー）にはGUI（ウィンドウ画面）がありません。**Vimでファイルを編集できることは組み込み開発の基本スキル**です。

---

## この日のゴール

- Vimで「開く → 編集 → 保存 → 閉じる」の基本操作ができる
- Vimで既存ファイルの特定の行を検索・編集できる
- `.bashrc` にaliasを追加してコマンドを短縮できる
- `$PATH` の概念を理解してパスを追加できる

---

## この日の前提

- Day 1 の作業が完了していること（`~/training/day01/` が存在する）

---

## 1. なぜVimを学ぶのか

### SSHの先にはGUIがない

Windows開発では Visual Studio や VS Code でマウスを使ってコードを書く。しかし SSH 接続先の Linux サーバーや SBC では、GUIが動かないのが通常。

```
Windows（あなたのPC）
    │ SSH接続
    ▼
Ubuntu VM（テキストしか表示できないターミナル）
    │ 将来はさらにここから
    ▼
SBC（Raspberry Pi など）← ここにもGUIはない
```

SSH接続先でファイルを編集するには、**ターミナル上で動くテキストエディタ**が必要。VimはどのLinuxにも標準でインストールされているため、サーバーや組み込みボードで必ず使える。

> **救済策**：Visual Studio Code の「Remote - SSH」拡張を使えば、SSH接続先のファイルをWindows側のVSCodeで編集できる。今日はVimを学ぶが、詰まったときの逃げ道として[後述](#vscode-remote-ssh-を使う逃げ道)を参照。

### Vim の「モード」という概念

他のエディタとの最大の違いは**モード**の存在。キーボードのキーが「文字入力」と「コマンド操作」で切り替わる。

```
┌─────────────────────────────────────────────────────┐
│ Normal（ノーマル）モード                               │
│ キーを押す = コマンド実行（移動・削除・コピーなど）      │
│                                                     │
│    i を押す → Insert モードに切り替わる               │
│    : を押す → Command モードに切り替わる               │
└───────────────────┬─────────────────────────────────┘
                    │
        ┌───────────▼───────────┐
        │ Insert モード          │
        │ キーを押す = 文字入力   │
        │                       │
        │ Esc を押す → Normal へ │
        └───────────────────────┘
```

**最初に覚えること**：「Esc を押すと Normal モードに戻る」。迷子になったら Esc。

---

## 2. Vim サバイバルガイド（最低限これだけ）

SSH先でファイルを触るときに「詰まって抜け出せない」を防ぐための最小セット。

```bash
# ファイルを開く
vim filename.c

# 文字を入力する（Insert モードに入る）
i

# 編集をやめて Normal モードに戻る
Esc

# 保存して終了
:wq

# 保存せずに終了（変更を捨てる）
:q!

# 保存だけ（終了しない）
:w
```

---

## 3. Vim の基本操作

### 3.1 モード切替

| キー | 動作 |
|------|------|
| `i` | Insert モード（カーソル位置の前から入力） |
| `a` | Insert モード（カーソル位置の後から入力） |
| `o` | Insert モード（カーソル行の下に新しい行を追加して入力） |
| `Esc` | Normal モードに戻る |
| `:` | Command モード（Normal モードから） |

### 3.2 Normal モードでの移動

Vimでは矢印キーでも移動できるが、慣れてきたら以下のキーを使うと効率的。

| キー | 動作 |
|------|------|
| `h` / `l` | 左 / 右 |
| `j` / `k` | 下 / 上 |
| `0` | 行頭へ |
| `$` | 行末へ |
| `gg` | ファイル先頭へ |
| `G` | ファイル末尾へ |
| `10G` | 10行目へ移動（行番号+G） |
| `Ctrl + f` | 1ページ下へ |
| `Ctrl + b` | 1ページ上へ |

### 3.3 Normal モードでの編集

| キー | 動作 |
|------|------|
| `x` | カーソル位置の1文字を削除 |
| `dd` | 現在行を削除（切り取り） |
| `yy` | 現在行をコピー |
| `p` | カーソルの下に貼り付け |
| `u` | アンドゥ（1つ戻る） |
| `Ctrl + r` | リドゥ（やり直し） |

### 3.4 検索と置換

```vim
/keyword        Normal モードで / を押すと検索
n               次の候補へ
N               前の候補へ

:%s/old/new/g   ファイル全体で old を new に置換
:5,10s/old/new  5〜10行目だけ置換
```

### 3.5 Command モードのよく使うコマンド

```vim
:w              保存
:q              終了（変更がある場合は終了できない）
:wq             保存して終了
:q!             変更を捨てて強制終了
:set number     行番号を表示する
:10             10行目にジャンプ
```

---

## 4. Vimの設定ファイル（.vimrc）

Vimは起動時に `~/.vimrc` を読み込む。設定を書いておくと毎回適用される。

```bash
vim ~/.vimrc
```

以下を書いてから `:wq` で保存する：

```vim
set number          " 行番号を表示
set tabstop=4       " タブ幅を4に設定
set expandtab       " タブをスペースに変換
set autoindent      " 自動インデント
set hlsearch        " 検索結果をハイライト
set incsearch       " 入力しながら検索（インクリメンタルサーチ）
set ignorecase      " 検索で大文字小文字を区別しない
syntax on           " シンタックスハイライトを有効にする
```

---

## 5. Bashの設定

### .bashrc とは

SSHでログインして新しいシェル（Bash）が起動するたびに読み込まれる設定ファイル。ここに設定を書いておくと、次回のログイン時から自動的に適用される。

Windowsの「環境変数の設定」や「PowerShellプロファイル」に相当する。

### .bashrc を編集する

```bash
vim ~/.bashrc
```

### alias（コマンドの短縮形）を設定する

ファイルの末尾に以下を追加する（`G` でファイル末尾、`o` で新しい行を追加）：

```bash
# 短縮コマンド
alias ll='ls -la'           # ls -la を ll で呼び出す
alias ..='cd ..'            # .. で1つ上に移動
alias ...='cd ../..'        # ... で2つ上に移動
alias gs='git status'       # git status を gs で呼び出す
alias gb='git branch'
alias gl='git log --oneline --graph'
```

設定を即座に反映させる（再ログインしなくてよい）：

```bash
source ~/.bashrc
```

動作確認：

```bash
ll                          # ls -la と同じ動作になる
```

### $PATH の仕組み

`PATH` 環境変数はコマンドを探す場所のリスト。コマンドを実行すると、`PATH` に書かれたディレクトリを左から順に探して実行ファイルを見つける。

```bash
# 現在のPATHを確認する
echo $PATH
# 出力例：/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

将来、自分でビルドしたツールを `~/bin/` に置いて使いたい場合は、`$PATH` に追加する。

```bash
# ~/.bashrc の末尾に追加する
export PATH="$HOME/bin:$PATH"   # $HOME/bin を PATH の先頭に追加

# 反映
source ~/.bashrc

# 確認
echo $PATH
```

### よく使う環境変数

| 変数名 | 内容 | 確認方法 |
|--------|------|---------|
| `$HOME` | ホームディレクトリのパス | `echo $HOME` |
| `$PATH` | コマンドの検索パス | `echo $PATH` |
| `$USER` | 現在のユーザー名 | `echo $USER` |
| `$SHELL` | 使用しているシェルのパス | `echo $SHELL` |

---

## 6. ハンズオン

### Step 1：作業ディレクトリを作成する

```bash
mkdir -p ~/training/day02
cd ~/training/day02
```

### Step 2：Vimで C ソースファイルを作成する

```bash
vim hello.c
```

1. `i` を押して Insert モードに入る
2. 以下を入力する：

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, Linux!\n");
    return 0;
}
```

3. `Esc` を押して Normal モードに戻る
4. `:wq` で保存して終了する

確認：

```bash
cat hello.c
```

### Step 3：行番号表示と検索を練習する

```bash
vim hello.c
```

1. `:set number` を実行して行番号を表示する
2. `/printf` で `printf` を検索する（`n` で次の候補へ）
3. `:q` で終了する（変更がないので終了できる）

### Step 4：.vimrc を設定する

```bash
vim ~/.vimrc
```

セクション4の内容を入力して `:wq` で保存する。

次にVimを開いたときから行番号が自動表示される：

```bash
vim hello.c
```

行番号が表示されていれば設定完了。

### Step 5：.bashrc にaliasを追加する

```bash
vim ~/.bashrc
```

- `G` でファイル末尾に移動
- `o` で新しい行を追加して Insert モードに入る
- セクション5のaliasを入力する
- `Esc` → `:wq` で保存

反映：

```bash
source ~/.bashrc
```

動作確認：

```bash
ll          # ls -la と同じ出力になる
gs          # git status が動く
```

### Step 6：今日の作業をコミットする

```bash
cd ~/training
git add day02/
git add .vimrc .bashrc     # ← 設定ファイルも記録しておく
git commit -m "day02: Vim基本操作とBash設定を完了"
```

---

## VSCode Remote-SSH を使う逃げ道

Vimに慣れるまでの間、または複雑な編集が必要な場合は、Windows側のVSCodeでSSH先のファイルを編集できる。

### 設定手順

1. Windows側のVSCodeを開く
2. 拡張機能（Ctrl+Shift+X）で `Remote - SSH`（Microsoft製）を検索・インストールする
3. 左サイドバーの「リモートエクスプローラー」アイコンをクリック
4. 「SSH」セクションの「+」をクリックして接続先を追加：
   ```
   ssh user@127.0.0.1 -p 2222
   ```
5. 接続するとVSCodeがUbuntu側で動作し、ファイルをGUIで編集できる

この方法では、VSCodeの拡張機能（C/C++補完、シンタックスハイライト）も使える。

> **研修の方針**：Vimも使えるようにしておく。SSH接続先に必ずVSCodeが使える環境があるとは限らないため。

---

## よくあるエラーと対処法

| 問題 | 原因 | 対処 |
|------|------|------|
| Vimを開いたが何も入力できない | Normalモードのまま | `i` を押してInsertモードに切り替える |
| `:wq` と入力したのに保存できない | Insertモードのまま `:` を入力した | `Esc` を押してNormalモードに戻ってから `:wq` |
| Vimから抜け出せない | よくあるVimあるある | `Esc` → `:q!` で強制終了（変更は捨てられる） |
| `source ~/.bashrc` 後にエラーが出る | .bashrcに文法ミスがある | エラーメッセージの行番号を確認して .bashrc を修正する |
| `ll` コマンドが動かない | aliasが反映されていない | `source ~/.bashrc` を実行する |

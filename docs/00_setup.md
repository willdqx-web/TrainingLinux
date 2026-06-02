# 事前準備：VirtualBox + Ubuntu のインストール

> Day 1 の作業を始める前に、このドキュメントの手順をすべて完了させてください。

---

## この研修で作る環境

10日間で「Linux上でC/C++プログラムをビルド・デバッグできる開発環境」を構築します。

```
┌─────────────────────────────────────────────────┐
│  Windows 11（あなたのPC）                         │
│                                                  │
│  ┌───────────────────────────────────────────┐   │
│  │  VirtualBox                               │   │
│  │                                           │   │
│  │  ┌─────────────────────────────────────┐  │   │
│  │  │  Ubuntu 22.04 LTS（仮想マシン）      │  │   │
│  │  │  ・gcc / g++（コンパイラ）           │  │   │
│  │  │  ・gdb（デバッガ）                   │  │   │
│  │  │  ・make / cmake（ビルドツール）       │  │   │
│  │  │  ・git（バージョン管理）             │  │   │
│  │  └─────────────────────────────────────┘  │   │
│  └───────────────────────────────────────────┘   │
│                                                  │
│  Windows Terminal（SSH接続でUbuntuを操作する）     │
└─────────────────────────────────────────────────┘
```

### 10日間のスケジュール

| Day | 内容 | 完成する状態 |
|-----|------|------------|
| **事前** | VirtualBox + Ubuntu インストール | SSHでUbuntuに接続できる |
| **Day 1** | Linuxファイルシステム・基本コマンド | ls, find, grepが使える |
| **Day 2** | Vim・Bash設定 | ターミナルでコードを編集できる |
| **Day 3** | パイプ・リダイレクト・プロセス管理 | 標準的なシェル操作ができる |
| **Day 4** | パーミッション・パッケージ管理・Git CLI | sudoとaptとgitが使える |
| **Day 5** | gcc/g++でのビルド | コマンドラインでC/C++をコンパイルできる |
| **Day 6** | gdbデバッグ | コマンドラインでブレークポイントを張れる |
| **Day 7** | Makefile | makeコマンドでビルドできる |
| **Day 8** | CMake | cmake --buildでビルドできる |
| **Day 9** | 統合課題 | ゼロからプロジェクトを構築できる |

---

## インストールするツール一覧

| ツール | 役割 | 確認コマンド |
|--------|------|------------|
| VirtualBox | 仮想マシン（Linux PCの代わり）を動かすソフト | — |
| Ubuntu 22.04 LTS | 仮想マシン上で動かすLinux | — |
| Windows Terminal | SSH接続のターミナル（標準アプリ） | — |

---

## 1. VirtualBox のインストール

### VirtualBox とは

「仮想マシン（VM）」を作るソフト。自分のWindows PC上に、本物そっくりのLinux PCを丸ごと再現できる。

```
物理PC（Windows）
├── VirtualBox（仮想化ソフト）
│   ├── 仮想マシンA：Ubuntu 22.04（→ 今回作るもの）
│   └── 仮想マシンB：別のOSも作れる
└── Windows の通常アプリ（Word、Chrome など）
```

VMwareや Hyper-Vも同種のソフトだが、今回は無料で使えるVirtualBoxを使う。

### インストール手順

1. 以下の公式サイトにアクセスする
   ```
   https://www.virtualbox.org/wiki/Downloads
   ```

2. 「Windows hosts」をクリックしてインストーラーをダウンロードする

3. ダウンロードした `VirtualBox-x.x.x-xxxxxx-Win.exe` を実行する

4. インストール中の設定はすべてデフォルトのまま「Next」を押す

5. 「Would you like to proceed with installation now?」で「Yes」を選ぶ

6. インストール完了後、「Finish」をクリックしてVirtualBoxを起動する

### インストール確認

VirtualBoxが起動し、以下の画面が表示されれば成功。

```
Oracle VirtualBox マネージャー
[ようこそ VirtualBox へ！] の画面が表示される
```

### Hyper-V との競合について（重要）

Windows 11 では「Hyper-V」という別の仮想化機能が有効になっている場合があり、VirtualBoxと干渉してVMが起動できないことがある。

**確認方法**

PowerShellを管理者で開いて以下を実行：

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
```

`State : Enabled` と表示された場合は以下で無効化する：

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
```

再起動後にVirtualBoxを起動して動作を確認する。

> **注意**：Docker DesktopやWSL2を使っている場合、Hyper-Vを無効化するとそちらが動かなくなる。研修中はこれらを一時的に使わない前提で進める。

---

## 2. Ubuntu 22.04 LTS の ISO をダウンロードする

### Ubuntu とは

Linuxディストリビューション（Linuxの配布形式の一種）の中で最もユーザーが多く、情報も豊富。本番のSBC環境でもDebian系（Ubuntuの元となったOS）が広く使われる。

`22.04 LTS` の意味：
- `22.04`：2022年4月リリース
- `LTS`：Long Term Support（長期サポート版）。2027年まで更新が提供される安定版

### ダウンロード手順

1. 以下の公式サイトにアクセスする
   ```
   https://ubuntu.com/download/server
   ```

2. 「Download Ubuntu Server 22.04 LTS」をクリックしてダウンロードする（約1.4GB）

> **Serverエディションを使う理由**：GUI（デスクトップ画面）なしのモードで動かすため。本番のSBC/サーバーと同じ環境を研修段階から体験する。DesktopエディションでもGUIを無効にできるが、Serverエディションの方が軽量。

### チェックサムの確認（推奨）

ダウンロードが正常に完了したか確認する。

公式サイトに掲載されているSHA256値と、ダウンロードしたファイルのSHA256値が一致すれば正常。

```powershell
Get-FileHash C:\Users\あなたのユーザー名\Downloads\ubuntu-22.04.x-live-server-amd64.iso -Algorithm SHA256
```

---

## 3. 仮想マシンを作成する

### Step 1：新しい VMを作成する

VirtualBoxマネージャーを開き、「新規」をクリックする。

**名前とオペレーティングシステム**

| 項目 | 設定値 |
|------|--------|
| 名前 | `ubuntu-training`（任意） |
| マシンフォルダー | デフォルトのまま |
| タイプ | `Linux` |
| バージョン | `Ubuntu (64-bit)` |

### Step 2：メモリサイズを設定する

スライダーを動かして **2048 MB（2GB）** に設定する。

> **なぜ2GBか**：ホストPCのRAMが8GBの場合、VMに2〜3GB割り当てるのが適切。4GB以上割り当てるとホスト側のWindows動作が重くなる。

>※メモリサイズに余裕があるなら、2048以上をVMに割り当てても良い

### Step 3：ハードディスクを作成する

「仮想ハードディスクを作成する」を選択して「作成」をクリック。

| 項目 | 設定値 |
|------|--------|
| ファイルタイプ | VDI（VirtualBox Disk Image） |
| 物理ハードディスクにあるストレージ | 可変サイズ |
| ファイルの場所とサイズ | **20.00 GB** |

> **可変サイズを選ぶ理由**：実際に使った分だけWindowsのディスクを消費する。最初から20GBすべて確保する「固定サイズ」より効率的。

### Step 4：CPUコア数を設定する

作成されたVMを選択し、「設定」→「システム」→「プロセッサー」を開く。

| 項目 | 設定値 |
|------|--------|
| プロセッサー数 | **2** |

> **なぜ2コアか**：ホストの6コアのうち2コアをVMに割り当て、残り4コアをWindowsに残す。コンパイル（`make -j2`）が並列で動くため1コアより速い。

> ※CPUコア数に余裕がある場合、2コア以上をVMに割り当てても良い。

### Step 5：ネットワークを確認する

「設定」→「ネットワーク」を開く。

| 項目 | 設定値 |
|------|--------|
| アダプター1 | 有効（チェックを入れる） |
| 割り当て | **NAT** |

NATモードでは、VMからインターネットへのアクセス（`apt`でのパッケージ取得など）はできるが、ホストPCからVMへのSSH接続が必要なため、次のポートフォワーディング設定を行う。

**ポートフォワーディングの設定**

「詳細」→「ポートフォワーディング」をクリックし、「+」ボタンで以下を追加する：

| 名前 | プロトコル | ホストIP | ホストポート | ゲストIP | ゲストポート |
|------|-----------|---------|------------|---------|------------|
| SSH | TCP | 127.0.0.1 | 2222 | （空欄） | 22 |

> **この設定の意味**：Windows側の `localhost:2222` へのアクセスを、VM内の `22番ポート（SSH）` に転送する。

---

## 4. Ubuntu をインストールする

### Step 1：ISOファイルをセットする

作成したVMを選択し、「設定」→「ストレージ」を開く。

「コントローラー：IDE」の「空」をクリックし、右側の「光学ドライブ」のディスクアイコンをクリック → 「ディスクファイルを選択」でダウンロードしたISOファイルを選ぶ。

### Step 2：VMを起動してインストーラーを起動する

VirtualBoxマネージャーで「起動」をクリックする。

しばらくするとUbuntuのインストーラーが起動する。

### Step 3：インストーラーの設定

キーボード操作（↑↓キーとEnterキー）で選択する。マウスは使えない。

| 画面 | 選択 |
|------|------|
| 言語選択 | `English`（そのままEnter） |
| インストーラーのアップデート | `Continue without updating` |
| キーボード設定 | `Japanese`（または`English (US)`） |
| ネットワーク設定 | そのままEnter（DHCPで自動設定） |
| プロキシ設定 | 空欄のままEnter |
| Ubuntuアーカイブミラー | そのままEnter |
| ストレージ設定 | `Use an entire disk`（デフォルト） |
| ストレージ構成確認 | `Done` → `Continue` |

**プロファイル設定（重要）**

| 項目 | 設定例 |
|------|--------|
| Your name | `training`（任意） |
| Your server's name | `ubuntu-training`（任意） |
| Pick a username | `user`（任意。英小文字のみ） |
| Choose a password | 任意のパスワード（忘れないように） |

**SSH設定**

`Install OpenSSH server` にスペースキーでチェックを入れてから `Done` を選ぶ。

| 残りの画面 | 選択 |
|-----------|------|
| Snaps | そのまま `Done` |
| インストール完了 | `Reboot Now` |

### Step 4：再起動後の確認

再起動後、以下のプロンプトが表示されればインストール完了。

```
ubuntu-training login:
```

ユーザー名とパスワードを入力してログインできることを確認する。

---

## 5. Windows Terminal から SSH で接続する

研修中の操作はすべてSSH経由で行う。VirtualBoxのウィンドウは閉じてよい。

### Windows Terminal とは

Windows 11 に標準で入っているターミナルアプリ。複数のタブを開けるため、SSH接続しながら別タブでWindowsのコマンドも実行できる。

スタートメニューで「Windows Terminal」を検索して起動する。

### SSH 接続コマンド

```powershell
ssh user@127.0.0.1 -p 2222
```

- `user`：Ubuntuインストール時に設定したユーザー名
- `127.0.0.1`：自分自身のPC（ポートフォワーディングでVMに転送される）
- `-p 2222`：先ほど設定したポートフォワーディングのホスト側ポート

初回接続時に以下のメッセージが表示される：

```
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

`yes` と入力してEnterを押す。パスワードを入力してログインする。

### 接続成功の確認

以下のプロンプトが表示されれば成功。

```
user@ubuntu-training:~$
```

### 接続を簡単にする（任意）

毎回 `-p 2222` を入力するのが手間な場合、`~/.ssh/config` に設定を書いておくと `ssh ubuntu-vm` だけで接続できる。

Windows側の `C:\Users\あなたのユーザー名\.ssh\config` に以下を追記：

```
Host ubuntu-vm
    HostName 127.0.0.1
    Port 2222
    User user
```

以降は `ssh ubuntu-vm` だけで接続できる。

---

## 6. 初期設定コマンドを実行する

SSH接続後、以下のコマンドを順番に実行する。

### パッケージを最新にする

```bash
sudo apt update && sudo apt upgrade -y
```

- `sudo`：管理者（root）権限でコマンドを実行する（Windowsの「管理者として実行」に相当）
- `apt update`：パッケージ一覧を最新に更新する
- `apt upgrade -y`：すべてのパッケージを更新する（`-y` は確認を自動でYesにする）

完了まで数分かかる。

### 開発ツールを一括インストールする

```bash
sudo apt install -y build-essential gdb cmake git vim curl
```

インストールされるツール：

| パッケージ名 | 含まれるツール |
|------------|-------------|
| `build-essential` | gcc, g++, make（C/C++開発の基本セット） |
| `gdb` | GNU デバッガ |
| `cmake` | CMakeビルドシステム |
| `git` | バージョン管理 |
| `vim` | テキストエディタ |
| `curl` | ファイルダウンロードツール |

### インストール確認

```bash
gcc --version
gdb --version
cmake --version
git --version
```

各コマンドでバージョン番号が表示されれば成功。

出力例（バージョン番号は異なっても問題ない）：

```
gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
```

---

## 7. 全体の最終確認

以下をすべて実行し、エラーなく完了することを確認する。

```bash
# 現在のユーザーとホスト名を確認
whoami && hostname

# Linuxのバージョンを確認
uname -r

# 開発ツールのバージョンを確認
gcc --version && make --version && git --version
```

期待する出力（例）：

```
user
ubuntu-training
5.15.0-91-generic
gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
...
GNU Make 4.3
...
git version 2.34.1
```

以上がすべて確認できれば事前準備は完了です。Day 1 の作業に進んでください。

---

## インストールでよくある問題

| 問題 | 原因 | 対処 |
|------|------|------|
| VMが起動しない（VT-x/AMD-Vエラー） | BIOSで仮想化が無効 | PCのBIOS設定を開き「Intel VT-x」または「AMD-V」を有効にして再起動 |
| VMが起動しない（Hyper-Vエラー） | Hyper-Vが有効になっている | 本文「Hyper-Vとの競合」セクションを参照 |
| SSH接続で `Connection refused` | SSHサーバーが起動していない or ポート設定ミス | VMに直接ログインして `sudo systemctl status ssh` で状態確認 |
| SSH接続で `Connection timed out` | ポートフォワーディング設定が間違っている | VirtualBoxのポートフォワーディング設定を再確認（ホストポート2222、ゲストポート22） |
| `apt update` でエラー | ネットワーク接続なし | VirtualBoxのネットワーク設定がNATになっているか確認 |
| ログインできない | パスワードを忘れた | VMを再起動してGrub画面でrecoveryモードに入りパスワードをリセット |

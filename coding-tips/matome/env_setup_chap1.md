---
marp: true
theme: default
paginate: true
header: '開発環境構築マニュアル'
footer: '第1章: 基本的な環境構築（OS別）'
---

<!-- _class: lead -->

# 開発環境構築マニュアル
## 第1章: 基本的な環境構築（OS別）

---

## 第1章 目次

- **1-1. セットアップの基本方針**
- **1-2. macOSでの環境構築**
  - Homebrew と `pyenv` の導入
- **1-3. Windowsでの環境構築**
  - Winget と `pyenv-win` の導入
- **1-4. シェル環境のカスタマイズ**
  - `.zshrc` の設定と活用

---

## 1-1. セットアップの基本方針

- **OSネイティブなパッケージマネージャーの活用**
  - **macOS**: `Homebrew` を使用
  - **Windows**: `Winget` を使用
- **Pythonバージョン管理ツールの導入**
  - **macOS**: `pyenv` を使用
  - **Windows**: `pyenv-win` を使用
- **統一されたPythonバージョンの利用**
  - チーム内で使用するPythonのバージョンを固定し、一貫性を保つ
- **シンプルな手順での再構築・再現性**
  - 新規環境や再インストール時にもスムーズにセットアップできるよう配慮

---

## 1-2. macOSでの環境構築 (Homebrew)

### **Homebrew のインストール**
まだインストールしていない場合のみ実行します。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### **基本ツールのインストール**
git, VSCode, iTerm2などを一括で導入します。

```bash
brew install git
brew install --cask visual-studio-code
brew install --cask iterm2 # 推奨ターミナル
brew install --cask obsidian
```

---

## 1-2. macOSでの環境構築 (pyenv)

### **pyenv のインストールと設定**
Pythonバージョンを管理するために `pyenv` を導入します。

```bash
# pyenvのインストール
brew install pyenv

# pyenvの設定をシェルに追加（~/.zshrc に追記）
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init --path)"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

# 設定を反映させるため、ターミナルを再起動してください
```

---

## 1-2. macOSでの環境構築 (pyenv)

### **Python 統一バージョンのインストール**
チームで決めた特定のPythonバージョンをインストールし、グローバルに設定します。

```bash
# 例: Python 3.11.9 をインストール
PYVER="3.11.9"
pyenv install $PYVER
pyenv global $PYVER

# インストールされたバージョンを確認
python --version

# pipを最新化
python -m pip install --upgrade pip
```

---

## 1-3. Windowsでの環境構築 (Winget)

### **基本ツールのインストール**
管理者としてPowerShellを開き、Git, VSCode, Windows Terminalなどをインストールします。

```powershell
# Git, VSCode, Windows Terminal, Obsidian のインストール
winget install --id Git.Git -e --source winget
winget install --id Microsoft.WindowsTerminal -e --source winget
winget install --id Microsoft.VisualStudioCode -e --source winget
winget install --id Obsidian.Obsidian -e --source winget
```

---

## 1-3. Windowsでの環境構築 (pyenv-win)

### **pyenv-win のインストールと設定**
Pythonバージョン管理のために `pyenv-win` を導入します。

```powershell
# pyenv-win のインストール
winget install --id pyenv.pyenv-win -e --source winget

# pyenv-win の環境変数設定スクリプトを実行
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"

# PowerShellの実行ポリシーを設定 (一時的)
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# システム環境変数へのパス追加を確実に反映させるため、一度PowerShellを再起動してください。
```

---

## 1-3. Windowsでの環境構築 (pyenv-win)

### **Python 統一バージョンのインストール**
チームで決めた特定のPythonバージョンをインストールし、グローバルに設定します。

```powershell
# 例: Python 3.11.9 をインストール
$PYVER = "3.11.9" # 必要に応じてバージョンを変更
pyenv install $PYVER
pyenv global $PYVER

# インストールされたバージョンを確認
python --version

# pipを最新化
python -m pip install --upgrade pip
```

---

## 1-4. シェル環境のカスタマイズ (.zshrc)

### **`.zshrc` とは**
macOSのデフォルトシェルであるZshの設定ファイルです。エイリアスや環境変数を記述することで、ターミナル環境をカスタマイズできます。

### **`.zshrc` の編集**
エディタでファイルを開き、設定を追記します。

```bash
# エディタで開く
nano ~/.zshrc
# または VSCode で開く
code ~/.zshrc
```

### **設定の反映**
変更を保存後、`source`コマンドで設定を即座に反映させます。

```bash
# 例: カスタムスクリプトのPATHを追加
echo 'export PATH="$HOME/workspace/utils/bin:$PATH"' >> ~/.zshrc

# 設定を反映
source ~/.zshrc
```

---

<!-- _class: lead -->

# 第1章 まとめ

**OS別の基本ツールとPython環境のセットアップが完了しました**

次の章では、Pythonのプロジェクト管理のベストプラクティスについて学びます。

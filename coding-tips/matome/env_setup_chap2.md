---
marp: true
theme: default
paginate: true
header: '開発環境構築マニュアル'
footer: '第2章: Pythonプロジェクト管理'
---

<!-- _class: lead -->

# 開発環境構築マニュアル
## 第2章: Pythonプロジェクト管理

---

## 第2章 目次

- **2-1. 仮想環境(venv)のベストプラクティス**
  - プロジェクトごとに環境を分離する
- **2-2. 依存関係のモダンな管理方法**
  - `pip-tools`による依存関係の宣言と固定
- **2-3. 機密情報（APIキー等）の取り扱い**
  - `.env`ファイルによる安全な情報管理

---

## この章のゴール

複数のプロジェクトが互いに影響し合うことなく、**クリーンで再現性の高いPython環境**を構築・管理する手法を確立します。

**「あのプロジェクトを動かしたら、こっちが動かなくなった…」** という問題を根絶することが目的です。

---

## 2-1. 仮想環境(venv)のベストプラクティス

### **原則： 1プロジェクト = 1仮想環境**
各プロジェクトのルートに専用の仮想環境(`.venv`)を作成し、依存関係を完全に分離します。

```
src/
└── backtester/
    ├── .venv/          # ← このプロジェクト専用のPython環境
    ├── pyproject.toml
    └── .vscode/
        └── settings.json # .venv内のPythonインタプリタを指定
```

**`.gitignore` に以下を必ず追記し、仮想環境自体はGit管理に含めません。**
```
**/.venv/
```
---

## 2-1. venvの作成と起動（スクリプト化）

毎回同じ手順で作成・起動できるよう、共通のヘルパースクリプトを用意します。

### **作成用スクリプト (`setup_venv.sh`)**
Pythonの仮想環境を作成し、`pip-tools`などの基本ツールをインストールします。
```bash
#!/usr/bin/env bash
set -euo pipefail
proj_dir=${1:?"Usage: setup_venv.sh <project_dir>"}
cd "$proj_dir"
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
# 依存管理ツールをインストール
pip install pip-tools
```

---

## 2-1. venvの作成と起動（スクリプト化）

### **起動用スクリプト (`start_dev.sh`)**
指定したプロジェクトの仮想環境を有効化します。

```bash
#!/usr/bin/env bash
set -euo pipefail
proj=${1:-"src/backtester"} # デフォルトのプロジェクトを指定
cd "$(git rev-parse --show-toplevel)"/"$proj"
if [ ! -d .venv ]; then echo "No .venv found. Run setup_venv.sh"; exit 1; fi
source .venv/bin/activate
echo "Activated venv for $proj"
```
これにより、`./scripts/start_dev.sh src/kiyohara` のようにコマンド一つで環境を切り替えられます。

---

## 2-2. 依存関係のモダンな管理方法

`pip freeze` で生成される`requirements.txt`は、間接的な依存関係もすべて含まれ、可読性が低いという問題があります。
そこで **`pip-tools`** を導入し、**「宣言」と「固定」を分離**します。

- **`requirements.in`**: **人間が管理するファイル**
  - `pandas` や `requests` など、プロジェクトで直接使うパッケージのみを記述。
- **`requirements.txt`**: **機械が生成するファイル**
  - `.in` ファイルを元に、孫依存まで含めたすべてのパッケージのバージョンを固定したもの。

---

## 2-2. `pip-tools` の具体的なワークフロー

**1. `requirements.in` を編集する**
プロジェクトで必要なライブラリを追記・編集します。
```txt
# requirements.in
# データ処理
pandas>=2.2

# API通信
requests

# 開発ツール
ruff
pytest
```
**2. `pip-compile` で `requirements.txt` を生成・更新する**
```bash
pip-compile -o requirements.txt requirements.in
```

**3. `pip-sync` で仮想環境を同期する**
生成された `requirements.txt` の内容と、現在の仮想環境を完全に一致させます。
```bash
pip-sync requirements.txt
```
---

## 2-3. 機密情報（APIキー等）の取り扱い

APIキーやパスワードなどの機密情報をコードに直接書き込むのは非常に危険です。
**`.env` ファイル** を使って、環境変数として管理・分離します。

### **ルール**
1. 各プロジェクトのルートに `.env` ファイルを置く。
2. `.env` ファイルは必ず `.gitignore` に追加する。
   ```
   # .gitignore
   .env
   **/.env
   ```
3. チームメンバー向けに、設定すべき項目を記載した `.env.example` を用意する。
   ```
   # .env.example (中身は空にする)
   EDINET_API_KEY=
   ```

---

## 2-3. Pythonでの `.env` の読み込み

**`python-dotenv`** ライブラリを使い、`.env`ファイルの内容をプログラムの実行時に環境変数として読み込ませます。

**1. インストール**
`requirements.in` に `python-dotenv` を追加し、`pip-sync` します。

**2. コードでの読み込み**
プログラムの冒頭で `load_dotenv()` を呼び出します。
```python
import os
from dotenv import load_dotenv

# .env ファイルの内容を環境変数として読み込む
load_dotenv()

# 環境変数から値を取得
api_key = os.getenv("EDINET_API_KEY")

if not api_key:
    raise ValueError("環境変数 EDINET_API_KEY が設定されていません。")
```

---

<!-- _class: lead -->

# 第2章 まとめ

- **プロジェクトごとに`.venv`を作成し環境を分離**
- **`pip-tools`で依存関係をクリーンに管理**
- **`.env`ファイルで機密情報を安全に分離**

これにより、複数人・複数プロジェクトでの開発が堅牢になります。

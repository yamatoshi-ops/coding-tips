---
marp: true
theme: default
paginate: true
header: '開発環境構築マニュアル'
footer: '第4章: 作業の自動化と効率化'
---

<!-- _class: lead -->

# 開発環境構築マニュアル
## 第4章: 作業の自動化と効率化

---

## 第4章 目次

- **4-1. ユーティリティスクリプト集の作成**
  - 「自分だけのコマンド」を作る
- **4-2. 自動化スクリプトの実例**
  - 論文インデックス作成、データセット同期、バックアップ
- **4-3. クラウドストレージとの連携**
  - シンボリックリンクの活用

---

## この章のゴール

日々の定型作業や雑務を自動化し、**「再利用可能な自分だけのツール群」**を構築します。

これにより、本来集中すべき創造的な作業に、より多くの時間を費やせるようになります。

---

## 4-1. ユーティリティスクリプト集の作成

### **コンセプト: 「自分だけのコマンド」を作る**
よく使うスクリプトを特定のフォルダ（例: `~/workspace/utils/bin`）に集約し、そのフォルダに`PATH`を通すことで、ターミナルのどこからでも短いコマンド名でスクリプトを呼び出せるようにします。

### **`PATH`の設定**
`~/.zshrc` に以下の一行を追記します。
```bash
# .zshrc
export PATH="$HOME/workspace/utils/bin:$PATH"
```
保存後、`source ~/.zshrc` を実行すると、`~/workspace/utils/bin` 内の実行可能なファイルが、`ls`や`cd`のような通常のコマンドと同じように扱えるようになります。

---

## 4-1. 共通venvとラッパースクリプト

`utils` ディレクトリ配下のPythonスクリプトは、すべて共通の仮想環境 (`~/workspace/utils/.venv`) を利用します。

`bin` フォルダに置くのは、Pythonロジックを呼び出すだけのシンプルな **「ラッパースクリプト」** です。

```bash
#!/usr/bin/env bash
# ~/workspace/utils/bin/make_paper_index (ラッパースクリプトの例)

# 共通のvenvを有効化
source "$HOME/workspace/utils/.venv/bin/activate"

# Pythonスクリプト本体を実行
python "$HOME/workspace/utils/papers/make_paper_index.py" "$@"

# venvを無効化
deactivate
```
この構造により、コマンドの呼び出し側はPythonの環境を意識する必要がなくなります。

---

## 4-2. 自動化スクリプトの実例

`PATH` を通したことで、以下のような自作コマンドがターミナルから直接実行可能になります。

- `make_paper_index`
- `datasets_sync`
- `backup_workspace`
- `memos_organize`

---

## 4-2. 実例①: 論文インデックス作成

### **コマンド: `make_paper_index`**
指定したフォルダ内にある大量の論文PDFをスキャンし、ファイル名やメタデータから論文一覧のMarkdownファイルを自動生成します。

**実行前のフォルダ**
```
papers/
├── DeepLearning2016.pdf
└── Transformer2017.pdf
```
**実行後の生成物 (`README.md`)**
```markdown
# Papers Index
| Title | Authors | Year |
|---|---|---|
| A review of deep learning | ... | 2016 |
| Attention is all you need | ... | 2017 |
```

---

## 4-2. 実例②: データセット同期

### **コマンド: `datasets_sync`**
`rsync`を使い、クラウドストレージとローカルの分析プロジェクト間で、データセットの同期（ダウンロード/アップロード）を効率的に行います。

**設定ファイル (`sync_config.yml`) で同期ルールを定義**
```yaml
# 例: クラウドからローカルへデータを"pull"
pull:
  - name: house-prices
    from:  ~/workspace/shared-drive/datasets/house-prices
    to:    ~/workspace/kaggle-challenges/house-prices/data

# 例: ローカルからクラウドへ成果物を"push"
push:
  - name: kensho-logs
    from:  ~/workspace/kensho-san/experiments/logs
    to:    ~/workspace/shared-drive/archive/kensho-logs
```
`datasets_sync`コマンドは、この設定ファイルに従って差分だけを同期します。

---

## 4-2. 実例③: ワークスペースのバックアップ

### **コマンド: `backup_workspace`**
現在のワークスペース全体を、不要なファイルを除外して圧縮・バックアップします。

- `.git` や `__pycache__`、`node_modules` などの不要なフォルダを自動で除外
- クラウドと同期している巨大な`shared-drive`フォルダも除外
- `workspace_YYYYmmdd_HHMM.tar.gz` のようなタイムスタンプ付きファイル名で保存
- （任意）古いバックアップを自動で削除

---

## 4-3. クラウドストレージとの連携

### **シンボリックリンクの活用**
Google Driveなどのクラウドストレージのフォルダを、あたかもローカルに存在するかのように見せかける技術です。

```bash
# "Google Drive/Project" を "workspace/shared-drive" としてリンクする
ln -s "~/Library/CloudStorage/GoogleDrive-*****/マイドライブ/Project" ~/workspace/shared-drive
```
これにより、
- Gitリポジトリ自体を肥大化させることなく、大容量データ（論文、データセット）を扱える
- `datasets_sync`のようなスクリプトで、クラウド上のファイルを透過的に操作できる
というメリットが生まれます。

---

<!-- _class: lead -->

# マニュアル総まとめ

- **第1章**: OS別の基本ツールとPython環境をセットアップしました。
- **第2章**: プロジェクトごとに環境を分離・管理する方法を学びました。
- **第3章**: VSCodeをプロジェクト指向の強力なIDEに進化させました。
- **第4章**: 定型作業を自動化する自分だけのコマンド群を構築しました。

**これで、あなたの開発環境は再現性が高く、堅牢で、非常に効率的なものになりました。**

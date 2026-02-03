---
marp: true
theme: default
paginate: true
header: '開発環境構築マニュアル'
footer: '第3章: 開発エディタ（VSCode）の最適化'
---

<!-- _class: lead -->

# 開発環境構築マニュアル
## 第3章: 開発エディタ（VSCode）の最適化

---

## 第3章 目次

- **3-1. マルチプロジェクトワークスペースの構築**
  - `.code-workspace`による複数プロジェクトの統合管理
- **3-2. プロジェクトごとの設定**
  - Python仮想環境との連携
  - タスクとデバッグの自動化
- **3-3. よくある権限問題の解決**
  - `sudo`実行による所有権問題とその対策

---

## この章のゴール

VSCodeを単なる高機能なテキストエディタから、**各プロジェクトの特性を理解した強力な統合開発環境（IDE）**へと昇華させます。

- 複数プロジェクトを横断したシームレスな開発
- プロジェクトに最適化されたPython環境の自動認識
- 定型作業のコマンド化による効率向上

---

## 3-1. マルチプロジェクトワークスペースの構築

### **`.code-workspace` ファイルとは**
複数のプロジェクトフォルダを、**1つのVSCodeウィンドウで同時に開く**ための設定ファイルです。

```json
// DevWorkspace.code-workspace
{
  "folders": [
    { "path": "src/backtester" },
    { "path": "src/kiyohara" },
    { "path": "docs" }
  ],
  "settings": {
    // ここにワークスペース全体の共通設定を記述
  }
}
```

このファイルをVSCodeで開くことで、指定したフォルダ群がエクスプローラーに並んで表示され、横断的な検索や編集が可能になります。

---

## 3-1. ワークスペース共通設定 (`settings`)

`.code-workspace`ファイル内の`settings`は、ワークスペース全体に適用される共通ルールを定義します。特に**パフォーマンス改善**のための設定が重要です。

### **検索・監視対象の除外**
仮想環境やキャッシュ、巨大なデータセットフォルダなどをVSCodeの監視対象から外すことで、CPU負荷を軽減し、動作を軽量化します。

```json
// DevWorkspace.code-workspace
"settings": {
    "search.exclude": {
        "**/.git": true,
        "**/.venv": true,
        "**/__pycache__": true,
        "**/datasets/**": true
    },
    "files.watcherExclude": {
        "**/.git/**": true,
        "**/.venv/**": true
    }
}
```

---

## 3-2. プロジェクトごとの設定（Python連携）

各プロジェクトフォルダの直下に`.vscode/settings.json`を作成し、プロジェクト固有の設定を定義します。

### **Pythonインタプリタの自動選択**
VSCodeがそのプロジェクト専用の仮想環境（`.venv`）を自動で認識するように設定します。

```json
// src/backtester/.vscode/settings.json
{
  // このプロジェクトを開いた際のデフォルトのPythonインタプリタを指定
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",

  // デバッグ時に読み込む.envファイルを指定
  "python.envFile": "${workspaceFolder}/.env"
}
```
`${workspaceFolder}`は、この`.vscode`フォルダが含まれるプロジェクトのルートパスを指します。

---

## 3-2. タスクとデバッグの自動化

`.vscode`フォルダ内に`tasks.json`と`launch.json`を作成し、定型作業をコマンドパレットから実行可能にします。

### **`tasks.json` (タスクの定義)**
リンターの実行やテストの起動などをタスクとして登録します。
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Run Tests",
      "type": "shell",
      "command": "pytest -q"
    }
  ]
}
```

### **`launch.json` (デバッグ設定)**
`F5`キーでデバッガを起動するための設定です。
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Run CLI",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/cli.py"
    }
  ]
}
```
---

## 3-3. よくある権限問題の解決

### **問題の状況**
スクリプトを`sudo`で実行した結果、作成されたファイルやフォルダの所有者が`root`になってしまい、VSCodeから保存しようとすると**「権限がありません(Permission Denied)」**とエラーが出ることがあります。

![h:150](https://img.icons8.com/ios-filled/50/FA5252/cancel.png)

VSCodeは現在のユーザー権限で動作するため、`root`が所有するファイルは編集できません。

---

## 3-3. 権限問題の診断と修正

### **診断方法**
`ls -l` コマンドで、ファイルやフォルダの所有者を確認します。
```bash
ls -l ~/workspace/
# 出力例（問題あり）
# drwxr-xr-x   5 root   staff   160 Sep  7  motor-control
```
`root`と表示されていたら、所有権が自分にありません。

### **修正コマンド**
`chown`コマンドで、所有権を自分に戻します。
```bash
# -R オプションで、指定したフォルダ配下すべてを再帰的に変更
sudo chown -R $(whoami) ~/workspace/motor-control
```
`$(whoami)`には、現在のユーザー名が自動で入ります。

**予防策**: ファイルやフォルダを作成するスクリプトは、原則として`sudo`なしで実行しましょう。

---

<!-- _class: lead -->

# 第3章 まとめ

- **`.code-workspace`で複数プロジェクトを統合管理**
- **`.vscode/settings.json`でPython環境を自動認識**
- **`tasks.json`と`launch.json`で定型作業を自動化**
- **`chown`コマンドでファイルの所有権問題を解決**

これでVSCodeがプロジェクトを深く理解する、強力な開発基盤となりました。

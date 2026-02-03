---
marp: true
theme: default
paginate: true
header: 'Git サブモジュール運用マニュアル'
footer: '© 2026 Your Name'
---

<!-- _class: lead -->

# Git サブモジュール運用マニュアル
`git_guide.md` の内容まとめ

---

## 本日の内容

1.  **基本操作**: サブモジュールを含むリポジトリの扱い方
2.  **更新フロー**: サブモジュールを更新した際の操作手順
3.  **URL切り替え**: リモートURLを変更する方法
4.  **推奨される改善**: より良く使うためのTips

---

<!-- _class: invert -->

## 1. 基本操作
サブモジュールを含むリポジトリの日常的な扱い方

---

### **新規クローン**
リポジトリを初めてローカルに持ってくる場合

サブモジュールを含めて一度にクローンするには `--recurse-submodules` を付けます。

```bash
git clone --recurse-submodules <repository_url>
```

---

### **既存リポジトリでの更新**
すでにクローン済みのリポジトリでサブモジュールを取得・更新する場合

`submodule update` コマンドで、サブモジュールを初期化・更新します。

```bash
# サブモジュールの初期化と取得
git submodule update --init --recursive
```
---

<!-- _class: invert -->

## 2. 更新フロー
サブモジュール（子）を更新し、親リポジトリに反映する手順

---

### **Step 1: サブモジュール内での開発**

まず、サブモジュール（子リポジトリ）のディレクトリに移動し、通常通り開発と`push`を行います。

```bash
# 1. サブモジュールのディレクトリへ移動
cd motor-control

# 2. 開発、コミット、プッシュを行う
git checkout -b feat/new-feature
git commit -m "feat: 新機能を追加"
git push origin feat/new-feature
```

---

### **Step 2: 親リポジトリでの参照更新**

次に、親リポジトリに戻り、更新されたサブモジュールの「参照（どのコミットを指すか）」をコミットします。

```bash
# 1. 親リポジトリへ戻る
cd ..

# 2. サブモジュールの変更をステージング
git add motor-control

# 3. 参照が更新されたことをコミット＆プッシュ
git commit -m "chore: bump motor-control submodule"
git push
```

---

<!-- _class: invert -->

## 3. URL切り替え
サブモジュールのリモートURLを変更する方法

---

### **URL切り替え手順 (HTTPS ⇄ SSH)**

`git config`でURLを変更し、`submodule sync`で設定を反映します。

```bash
# 1. .git/config の URL を書き換える
git config submodule.motor-control.url <new_url>

# 2. .gitmodules の変更をワーキングツリーに反映
git submodule sync --recursive

# 3. .gitmodules ファイルの変更をコミット
git add .gitmodules
git commit -m "Switch motor-control submodule to SSH"
```
---

<!-- _class: invert -->

## 4. 推奨される改善
より快適な運用のためのTips

---

### **推奨項目リスト**

- **READMEに注意書きを追記**
  - クローン時は `--recurse-submodules` が必須である旨を記載します。

- **`.gitignore` の再確認**
  - `venv/` や `__pycache__/` など、不要なファイルが追跡されないようにします。

- **GitHub Actions の設定**
  - CI/CDでサブモジュールをチェックアウトする設定を追加します。
    ```yaml
    - uses: actions/checkout@v4
      with:
        submodules: recursive
    ```
- **VSCodeの監視対象から除外**
  - `.code-workspace` ファイルで、データセットなどの重いディレクトリを検索・監視対象から除外（`search.exclude`）し、動作を軽量化します。

---

<!-- _class: lead -->

# まとめ

- **クローン時**: `--recurse-submodules`
- **更新時**: `git submodule update --init --recursive`
- **開発フロー**: 「子で開発 → 親で参照更新」

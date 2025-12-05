# qiita-content

My Qiita articles is here.

https://qiita.com/hiiragi_en17

## 📚 目次

- [基本的なコマンド](#基本的なコマンド)
- [記事管理ワークフロー](#記事管理ワークフロー)
  - [画像の追加方法](#-画像の追加方法)
- [注意点とベストプラクティス](#注意点とベストプラクティス)
- [トラブルシューティング](#トラブルシューティング)

## 基本的なコマンド

USAGE:
```bash
qiita <COMMAND> [<OPTIONS>]
```

COMMAND:
  - `init` - 記事をGitHubで管理するための初期設定
  - `login` - Qiita APIの認証認可
  - `new [<basename>] ...` - 新しい記事を追加
  - `preview` - コンテンツをブラウザでプレビュー
  - `publish <basename> ...` - 記事を投稿、更新
  - `publish --all` - 全ての記事を投稿、更新
  - `pull` - 記事ファイルをQiitaと同期
  - `version` - Qiita CLIのバージョンを表示
  - `help` - ヘルプを表示

OPTIONS:
  - `--credential <credential_dir>` - Qiita CLIの認証情報を配置するディレクトリを指定
  - `--root <root_dir>` - 記事ファイルをダウンロードするディレクトリを指定
  - `--verbose` - 詳細表示オプションを有効
  - `--config` - 設定ファイルを配置するディレクトリを指定

詳細については[Qiita CLI公式ドキュメント](https://github.com/increments/qiita-cli)をご覧ください

## 記事管理ワークフロー

### 📝 新しい記事を作成する場合

#### ローカルで作成する方法（推奨）

```bash
# 新しい記事を作成
npx qiita new 記事のタイトル

# または
npx qiita new "新しい記事のタイトル"
```

これにより `public/` ディレクトリに新しいマークダウンファイルが作成されます。最初は `id: null` の状態です。

#### 作成後の流れ

1. ローカルで記事を編集
2. `git add` → `git commit` → `git push`
3. mainブランチにマージすると、GitHub Actionsが自動でQiitaに投稿
4. 投稿後、Qiita CLIが記事IDを自動的に付与してファイル名を更新

### 🔄 パターン1: ローカルで編集 → Qiitaへ反映（通常の流れ）

```bash
# 1. 記事を作成・編集
npx qiita new "新しい記事"
# または既存のファイルを編集

# 2. Gitで管理
git add .
git commit -m "add: 新しい記事を追加"
git push

# 3. mainブランチにマージ
# → GitHub Actionsが自動でQiitaに投稿
```

### 🔄 パターン2: Qiitaで編集 → ローカルへ反映

Qiitaのエディタで記事を直接編集した場合の同期方法：

#### 自動同期（推奨）

1. Qiitaで記事を編集して保存
2. GitHub Actionsを手動実行（またはmainブランチに何かプッシュ）
3. ワークフローが自動的にQiitaの変更をGitHubに反映
4. `git pull` でローカルに取り込む

```bash
git pull origin main
```

#### 手動同期

```bash
# Qiitaから最新版を取得
npx qiita pull

# 変更を確認
git diff

# コミット
git add .
git commit -m "sync: Qiitaから変更を同期"
git push
```

### 📸 画像の追加方法

Qiita CLIには**公式の画像アップロード機能が含まれていません**。以下の方法で画像を記事に追加できます。

#### 方法1: Qiitaエディタで直接アップロード（推奨）

この方法が最も簡単で確実です。

1. **ローカルで記事をマークダウンで作成**
   ```markdown
   # 記事タイトル

   ![画像の説明](画像URL)  ← 一時的にプレースホルダーを入れておく
   ```

2. **GitHub Actions経由でQiitaに投稿**
   ```bash
   git add .
   git commit -m "add: 新しい記事を追加"
   git push
   # mainブランチにマージすると、Qiitaに記事が下書きとして作成される
   ```

3. **Qiita上で画像をアップロード**
   - Qiitaのエディタを開く
   - 画像をドラッグ&ドロップ、またはクリップボードから貼り付け
   - 自動的に画像URLが生成される
   ```markdown
   ![画像の説明](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/xxx/xxx.png)
   ```

4. **変更をローカルに同期**
   ```bash
   npx qiita pull
   git add .
   git commit -m "sync: 画像を追加"
   git push
   ```

#### 方法2: 外部画像ホスティングサービスを使用

外部サービスに画像をアップロードして、そのURLをマークダウンに直接記述する方法です。

**推奨サービス:**
- **Gyazo**: スクリーンショット共有に便利
- **Imgur**: 汎用的な画像ホスティング
- **Cloudinary**: 開発者向けの高機能な画像管理サービス

```markdown
![画像の説明](https://i.imgur.com/xxxxx.png)
```

#### 方法3: GitHubリポジトリに画像を保存

リポジトリ内に画像を保存し、GitHubのrawコンテンツURLを使用する方法です。

```bash
# imagesディレクトリを作成
mkdir -p images

# 画像を配置
cp path/to/image.png images/

# マークダウンで参照
![画像の説明](https://raw.githubusercontent.com/hiiragi17/qiita-content/main/images/image.png)
```

**注意点:**
- この方法は画像が公開リポジトリに保存されます
- 機密情報を含む画像には使用しないでください
- Qiitaの月間アップロード制限（100MB）にカウントされません

#### 画像アップロードの制限

Qiitaには以下の制限があります：
- **月間アップロード上限**: 100MB
- **画像の上書き**: 不可（新規アップロードのみ）

制限に達した場合の対処法：
1. 外部画像ホスティングサービスを使用
2. Qiitaサポートに連絡して古い画像を削除依頼

**参考リンク:**
- [画像のアップロード・削除方法 | Qiita ヘルプ](https://help.qiita.com/ja/articles/qiita-image-upload)
- [Qiitaの記事に画像を貼る](https://qiita.com/sitaya/items/d27bd286c3cda5332b8c)

## 注意点とベストプラクティス

### 1. ファイル名は常に英数字またはID形式を使用

```
✅ public/my-new-article.md
✅ public/abc123def456.md
❌ public/新しい記事.md
```

**重要**: 日本語のファイル名を使用すると、Qiita CLIとの同期時に重複や競合が発生する可能性があります。

### 2. private設定の確認

新規記事はデフォルトで `private: false` になることがあるので、公開前に確認：

```yaml
---
title: タイトル
private: true  # 下書きの場合はtrue
---
```

### 3. 競合を避ける

- **基本ルール**: ローカルで編集するか、Qiitaで編集するか、どちらか一方に統一
- 両方で編集してしまった場合は、`qiita pull` してからローカルの変更をマージ

### 4. 定期的な同期

```bash
# 作業開始前に必ずpull
git pull origin main
npx qiita pull
```

## トラブルシューティング

### 同期がうまくいかない場合

```bash
# 1. Qiitaの最新状態を取得
npx qiita pull

# 2. 差分を確認
git status
git diff

# 3. 問題があれば手動でマージ
git add .
git commit -m "sync: Qiitaとの同期"
git push
```

### ファイル名の重複が発生した場合

日本語ファイル名と記事IDベースのファイル名が重複した場合：

```bash
# 記事IDベースのファイル名に統一
git mv "public/日本語ファイル名.md" public/記事ID.md
git commit -m "fix: ファイル名を標準形式に修正"
git push
```

### GitHub Actionsのワークフローが失敗する場合

1. `.github/workflows/publish.yml` の設定を確認
2. リポジトリの Secrets に `QIITA_TOKEN` が設定されているか確認
3. `permissions: contents: write` が設定されているか確認

## 📖 参考リンク

- [Qiita CLI公式ドキュメント](https://github.com/increments/qiita-cli)
- [Qiita CLI GitHub Action](https://github.com/increments/qiita-cli/tree/main/actions/publish)
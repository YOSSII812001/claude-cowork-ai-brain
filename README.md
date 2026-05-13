# claude-cowork-ai-brain

Karpathy式「AI外部脳」をObsidian vault上に構築する、Claude Cowork / Claude Code 向けスキル。

> 「使うほど賢くなるパーソナルナレッジベース」を、3層構造（raw / wiki / CLAUDE.md）と
> 4操作サイクル（Ingest / Compile / Query / Lint）で実装します。
> vault肥大化時のコンテキスト圧迫を防ぐ3段戦略（Index → Grep → Read offset）と
> サブエージェント隔離を組み込み済み。

## 特徴

- **3層構造**：`raw/`（素材・読み取り専用） / `wiki/`（AI管理ノート層） / `CLAUDE.md`（80行以内の目次層）
- **4操作サイクル**：取込 → 構築 → 質問 → 健康診断 を `/wiki-*` スラッシュコマンドで完結
- **肥大化に強い**：Index → Grep → Read offset の3段戦略で、5,000ノート規模でもコンテキストを節約
- **サブエージェント活用**：重い横断探索を親コンテキストから隔離
- **既存ノートとの共存**：既存フォルダは一切移動せず、`/wiki-ingest` で個別に取込
- **ループ運用対応**：`/loop /wiki-compile` ・ `/loop /wiki-lint` でセルフペース自動化

## 動作環境

- **Claude Cowork**（推奨）または **Claude Code**
- **Obsidian** が手元にインストールされていること
- vault を1つ用意（既存vaultでもOK）
- （Claude Code利用時のみ）`obsidian-cli` または同等の操作層

## インストール

### Claude Cowork の場合

1. このリポジトリをローカルにcloneまたはダウンロード
   ```bash
   git clone https://github.com/YOSSII812001/claude-cowork-ai-brain.git
   ```
2. `claude-cowork-ai-brain` フォルダごと、Coworkのskillsディレクトリ配下に配置
   ```
   <Cowork skills root>/ai-brain/
   ```
3. Cowork側でskillが認識されることを確認

### Claude Code の場合

`~/.claude/skills/ai-brain/` 配下にコピー、または プラグイン化して導入。

## 初回設定（必須）

このリポジトリはサニタイズ済みです。**ご自身の環境に合わせて以下のプレースホルダーを置換してください。**

### 1. SKILL.md 内の置換

| プレースホルダー | 置換例 |
|----------------|--------|
| `<YOUR_USERNAME>` | Windowsユーザー名（例：`taro`） |
| `<YOUR_VAULT_NAME>` | Obsidian vault名（例：`my-vault`） |

具体的には `## 環境情報` セクションの以下3行：

```markdown
- **Vault名**: <YOUR_VAULT_NAME>
- **Vaultパス**: `C:\Users\<YOUR_USERNAME>\Documents\Obsidian\<YOUR_VAULT_NAME>`
- **Obsidian CLI**: `"/c/Users/<YOUR_USERNAME>/Downloads/Obsidian/Obsidian.com"`
```

および `## obsidian-cliとの連携` のbash例。

### 2. commands/*.md 内の置換

5つのコマンド定義ファイル冒頭の `対象vault: <YOUR_VAULT_NAME>` を実際のvault名に変更。

### 3. vault-CLAUDE-template.md を vault rootへ

このファイルを **vault rootの `CLAUDE.md`** としてコピー配置し、同様にプレースホルダーを置換。

### 4. 初期化

スキル導入後、Cowork（またはClaude Code）から：

```
/wiki-init
```

を実行すると、vault内に `raw/` と `wiki/` のスキャフォールドが作成されます。
**既存ファイルは一切移動されません。**

## 使い方

### スラッシュコマンド

| コマンド | 用途 |
|---------|------|
| `/wiki-init` | フォルダ構造のスキャフォールド（初回のみ） |
| `/wiki-ingest <URL or path>` | ソース素材を取り込み、要約を生成 |
| `/wiki-compile [対象]` | wiki全体の整合性維持・概念昇格・index再構築 |
| `/wiki-query <質問>` | ナレッジベースを横断検索→引用付き回答 |
| `/wiki-lint [対象]` | ヘルスチェック＋自動修正 |

### ループ運用（自動化）

```
/loop /wiki-compile
/loop /wiki-lint
```

セッションをバックグラウンドで回し、新しい取込・変更を自動検出して継続的にメンテナンス。

## アーキテクチャ概要

```
vault/
├─ CLAUDE.md            ← Layer 3: 目次層（80行以内）
├─ raw/                  ← Layer 1: 素材層（読み取り専用）
│   ├─ articles/
│   ├─ papers/
│   └─ ...
├─ wiki/                 ← Layer 2: 整理層（AI管理）
│   ├─ index.md
│   ├─ log.md
│   ├─ concepts/
│   ├─ sources/
│   └─ outputs/
└─ <既存フォルダはそのまま維持>
```

詳細は `SKILL.md` の各セクション、および `references/` 配下の手順書を参照。

## コンテキスト節約戦略

vault が数千ノート規模に成長してもAIが詰まらないよう、以下のルールがスキルに組み込まれています。

1. **Index**：まず `CLAUDE.md` と `wiki/index.md` だけ読む
2. **Grep**：`files_with_matches` モードで候補ファイルパスだけ取得
3. **Read offset**：ヒットファイルの該当行±数行だけ抽出

加えて、全件走査や横断分析のような重いタスクは **サブエージェントに丸投げ**して、
結果サマリだけを親コンテキストに戻します。

詳細は `SKILL.md` の `## Cowork環境での検索戦略` を参照。

## ディレクトリ構成

```
claude-cowork-ai-brain/
├─ SKILL.md                       # スキル本体（必ず最初に読まれる）
├─ vault-CLAUDE-template.md       # vault root に置く CLAUDE.md 雛形
├─ commands/                      # スラッシュコマンド定義
│   ├─ wiki-init.md
│   ├─ wiki-ingest.md
│   ├─ wiki-compile.md
│   ├─ wiki-query.md
│   └─ wiki-lint.md
└─ references/                    # 必要時にRead される詳細手順
    ├─ ingest-workflow.md
    ├─ compile-workflow.md
    ├─ query-workflow.md
    ├─ lint-workflow.md
    ├─ init-workflow.md
    ├─ schema-overview.md
    ├─ wiki-layer-structure.md
    ├─ raw-layer-rules.md
    ├─ migration-strategy.md
    ├─ naming-conventions.md
    ├─ page-threshold.md
    ├─ quality-standards.md
    ├─ frontmatter-template.md
    ├─ concept-template.md
    ├─ source-template.md
    ├─ index-template.md
    └─ log-template.md
```

## ライセンス

[MIT](LICENSE)

## 参考

- Andrej Karpathy: ["AI external brain" のコンセプト](https://karpathy.ai/) （提唱者）
- [Obsidian](https://obsidian.md/) — ローカル優先のMarkdownナレッジベース
- [Claude Cowork](https://claude.ai/) — Anthropic Claude のデスクトップ協働モード

## Contributing

Issue / PR 歓迎。改善案・運用例・他vault構成への対応など、お気軽にどうぞ。


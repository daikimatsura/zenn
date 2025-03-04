---
title: "CursorとAIエージェントを活用した開発効率化術"
emoji: "🤖"
type: "tech"
topics: ["Cursor", "AI駆動開発", "AIエージェント"]
published: true
published_at: 2025-03-04 09:00
---

# はじめに

[Cursorエディタ](https://docs.cursor.com/get-started/welcome)のようなAIコーディングアシスタントを活用すると、開発効率が大幅にアップします。でも、AIとの協業にはいくつか課題もありますよね。この記事では、AIとの協業をもっと効率的にするための工夫をした方法を紹介します。

## この記事で学べること

この記事を読むと、以下のことが学べます：

- AIコーディングアシスタントとの効率的な協業方法
- プロジェクト固有のルールをAIに理解させる仕組み
- Cursorのプロジェクトルールを活用したAIとの文脈維持の方法
- AIとの協業で生産性を向上させる方法

私自身、この方法でポートフォリオサイトを開発し、基本的な構造（0→1）は自分の手で、その後の機能拡張（1→10）はほとんどAIに指示を出すだけで実装できました。

今回実装したコードは以下からご覧になれます。

https://github.com/daikimatsura/portfolio

サイトURL：[https://daikimatsuura.vercel.app/](https://daikimatsuura.vercel.app/)

## AIとの協業における課題

AIコーディングアシスタントと協業する際、こんな悩みはありませんか？

1. **文脈の維持が難しい**：セッションが切れると、それまでの会話内容や決定事項が失われてしまいます。
2. **コーディングルールの一貫性**：プロジェクト固有のルールをAIに毎回説明するのは大変です。
3. **プロンプト管理**：AIへの指示を標準化し、一貫性を保つのが難しいことがあります。
4. **更新の自動化**：ルールや記憶を手動で更新するのは面倒で、忘れがちです。

これらの課題を解決するために、AIへの指示を効率よくする方法をご紹介いたします

## 解決策：AIへの指示ルールを効率よく更新していく方法

`.cursor`ディレクトリを作成し、以下のファイルを管理することで、AIとの協業をスムーズにしました：

```
.cursor/
├── prompt.md    # AIへの基本指示
├── rules.md     # コーディングルール
├── memory.md    # プロジェクトの記憶
└── sh/          # 自動化スクリプト
    ├── generate_rules.sh # ルールファイルを自動生成するスクリプト
    ├── update_rules.sh # ルールファイルを自動更新するスクリプト
    └── setup_git_hooks.sh # Gitのフックを自動設定するスクリプト
└── rules
    └── cursorrules.mdc # 自動生成されるCursorのルールファイル
```

これらのファイルから自動生成される`.cursor/rules/cursorrules.mdc`ファイルをAIが読み込むことで、プロジェクト固有のルールや文脈を理解してくれるようになります。

> **Note**: Cursorエディタでは以前は`.cursorrules`ファイルを使用していましたが、現在は`.cursor/rules/*.mdc`形式が推奨されています。この記事では新しい形式に対応した方法を紹介しています。詳細は[Cursor公式ドキュメント](https://docs.cursor.com/context/rules-for-ai)で確認できます。

### 各ファイルの役割

#### `prompt.md`

AIアシスタントへの基本的な指示を記述します。役割や重要な指示、開発プロセスなどを定義します。

```markdown
# シニアソフトウェアエンジニアとしてのポートフォリオサイト開発プロンプト

## あなたの役割

あなたはNext.js、React、Tailwind CSS、Shadcn UI、Radix UIに精通したシニアソフトウェアエンジニアです。ポートフォリオサイト開発のサポートを担当し、高品質なコードと最適なソリューションを提供します。

## 重要な指示

1. **プロジェクトコンテキストの理解**:

   - あなたは一般的な言語やフレームワークの扱いに長けていますが、個々のプロジェクトの背景やコンテキストを正確に読み取ることは苦手です
   - そのため、必ず [rules.md](mdc:.cursor/rules.md) と [memory.md](mdc:.cursor/memory.md) を読み、プロジェクト固有のルールと過去の実装パターンを理解してから作業を開始してください
   - これらのファイルにはプロジェクトの重要な記憶と実装パターンが記録されています

2. **正確性の確保**:

   - 絶対に嘘の情報を出力しないでください
   - ハルシネーション（実際には存在しない情報の生成）に注意してください
   - 不確かな情報は提供せず、代わりにユーザーに質問をしてください

3. **質問の活用**:

   - 情報が不足している場合や不明点がある場合は、遠慮なく質問してください
   - 推測よりも確認を優先してください

4. **コミュニケーション**:
   - 常に日本語で応答してください
   - 技術的な説明も日本語で行ってください
   - 明確で簡潔な説明を心がけてください

## 開発プロセス

(以下省略)
```

#### `rules.md`

プロジェクト固有のコーディングルールを記述します。ファイル構造、コンポーネント設計、スタイリングなどのルールを定義します。

```markdown
Next.js App Router with React, Shadcn UI, Radix UI, and Tailwind
ポートフォリオサイト用のコーディングガイドライン

1. もしあなたがLLMなら以下のガイドラインに従ってください:

- このガイドラインを必ず守る
- 必ず [memory.md](mdc:.cursor/memory.md) ファイルを読み、プロジェクトの記憶と過去の実装パターンを理解してから作業を開始する
- 着手前に既存のソースコードを読み込む
- ソースコードを全て読み込んだら次にこれから実装するプランを考えそのプランが正しいかを確認する
- タスクが細分化されていないと判断したら必ずタスクを細分化しプランを再考する
- そのプランが正しいと判断したらテストコードをまず最初に実装します
- テストコードを実装した後に実際のコードを実装します
- コードを実装したら必ずそのコードが正しいかを確認する
- コード実装し終えた後にコードが正しいものであってもリファクタリングの余地がある場合はリファクタリングを行う
- 特にコードの重複がある場合や一般化できる場合は必ず共通化を行う
- 指摘があればそれに従い、その指摘事項を繰り返し指摘されないように[rules.md](mdc:.cursor/rules.md) を更新してナレッジを貯めていく
- 常に日本語で応答し、技術的な説明も日本語で行う
- [memory.md](mdc:.cursor/memory.md) に記載されている実装パターンや解決策を参考にし、一貫性のあるコードを実装する
- 新しい知見や重要な実装パターンを発見した場合は、[memory.md](mdc:.cursor/memory.md) への追加を提案する

2. ファイル構造:

(以下省略)
```

#### `memory.md`

プロジェクトの重要な記憶や実装パターンを記録します。過去に解決した問題や重要な決定事項などを記録しておくことで、AIがプロジェクトの文脈を理解しやすくなります。

```markdown
# ポートフォリオサイト開発の重要な記憶

このファイルはポートフォリオサイト開発における重要な記憶を保存するためのものです。LLMセッションが切れた後も、プロジェクトの知識と実装パターンを引き継ぐために使用されます。新しい開発を始める前に必ずこのファイルを参照し、プロジェクトの一貫性を保つようにしてください。新しい知見や重要な実装パターンを発見した場合は、このファイルに追加することを検討してください。

## LLMとしての対応方

1. **コード実装前の準備**

   - 既存のソースコードと [rules.md](mdc:.cursor/rules.md) , [memory.md](mdc:.cursor/memory.md) , [prompt.md](mdc:.cursor/prompt.md)を読み込み、プロジェクトの構造と規約、あなたの役割を理解
   - 実装プランを立て、タスクを細分化

2. **コード品質の確保**

   - 実装後にコードの正確性を確認
   - リファクタリングの余地がある場合は改善を提案
   - コードの重複を避け、共通化を推進

3. **コミュニケーション**

   - 常に日本語で応答
   - 技術的な説明も日本語で行う
   - 指摘事項は [rules.md](mdc:.cursor/rules.md) に反映してナレッジを蓄積

4. **記憶の管理**
   - あなたのタスクが完了するごとに必ず [memory.md](mdc:.cursor/memory.md) を更新する
   - 記憶を読み込んだ後に不要な記述や重複箇所があれば記憶を整理しても良いが必ずコンテキストが損なわれる変更はしない
   - 記憶を管理する際に日付を入れない（具体的な年月日を記載しない）
   - これ以降の記述は古い順に書いてあるものである（更新する際も必ず後ろに追加する）
   - セッションが切れそうだと感じたら現在のタスクを終了し [memory.md](mdc:.cursor/memory.md) を更新し整理して次のセッションに移行することをユーザーに提案してください

## プロジェクト構造に関する記憶

(以下省略)
```

新しい知見や実装パターンがあったらこのmemory.mdを更新してくださいという指示をプロンプトに入れるのも重要です。
時折守らないこともあるのでその場合は直接指示しましょう。

人間と同じですね。

全文が気になる方は以下からご覧ください。

https://github.com/daikimatsura/portfolio/tree/main/.cursor

### 自動化スクリプト

`.cursor`ディレクトリ内の`sh/`フォルダには、以下の自動化スクリプトを配置します：

#### `generate_rules.sh`

このスクリプトは、`prompt.md`、`rules.md`、`memory.md`の内容を結合して`.cursor/rules/cursorrules.mdc`ファイルを生成します：

````sh

#!/bin/bash

# スクリプトの説明
echo "=== .cursor/rules/cursorrules.mdcの自動生成スクリプト ==="
echo "このスクリプトは.cursor配下のmdファイル（prompt.md、rules.md、memory.md）を読み込み、.cursor/rules/cursorrules.mdcファイルを自動生成します。"
echo ""

# 作業ディレクトリを取得
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
CURSOR_DIR="$(dirname "$SCRIPT_DIR")"
PROJECT_ROOT="$(dirname "$CURSOR_DIR")"

# 出力ディレクトリとファイルのパス
OUTPUT_DIR="$CURSOR_DIR/rules"
OUTPUT_FILE="$OUTPUT_DIR/cursorrules.mdc"

# 一時ファイル
TMP_FILE="$CURSOR_DIR/tmp_rules"

# 出力ディレクトリの存在確認と作成
if [ ! -d "$OUTPUT_DIR" ]; then
  echo "出力ディレクトリが存在しないため作成します: $OUTPUT_DIR"
  mkdir -p "$OUTPUT_DIR"
fi

# ファイルの存在確認
if [ ! -f "$CURSOR_DIR/rules.md" ]; then
  echo "エラー: $CURSOR_DIR/rules.md が見つかりません。"
  exit 1
fi

if [ ! -f "$CURSOR_DIR/memory.md" ]; then
  echo "エラー: $CURSOR_DIR/memory.md が見つかりません。"
  exit 1
fi

# prompt.mdの存在確認
PROMPT_EXISTS=false
if [ -f "$CURSOR_DIR/prompt.md" ]; then
  PROMPT_EXISTS=true
else
  echo "警告: $CURSOR_DIR/prompt.md が見つかりません。プロンプトなしで生成します。"
fi

# 一時ファイルを初期化
> "$TMP_FILE"

# YAML frontmatterを最初に追加
echo "---" > "$TMP_FILE"
echo "description: 以下のプロンプトに絶対遵守すること" >> "$TMP_FILE"
echo "globs: " >> "$TMP_FILE"
echo "alwaysApply: true" >> "$TMP_FILE"
echo "---" >> "$TMP_FILE"

# 次にヘッダーを追加
echo "# 自動生成された.cursor/rules/cursorrules.mdc" >> "$TMP_FILE"
echo "# 生成日時: $(date)" >> "$TMP_FILE"
echo "# このファイルは.cursor/sh/generate_rules.shによって自動生成されています。" >> "$TMP_FILE"
echo "# 直接編集せず、.cursor/prompt.md、.cursor/rules.md、.cursor/memory.mdを編集してください。" >> "$TMP_FILE"
echo "" >> "$TMP_FILE"

# prompt.mdの内容を最初に追加（存在する場合）
if [ "$PROMPT_EXISTS" = true ]; then
  echo "## プロンプト" >> "$TMP_FILE"
  echo "" >> "$TMP_FILE"
  cat "$CURSOR_DIR/prompt.md" >> "$TMP_FILE"
  echo "" >> "$TMP_FILE"
fi

# rules.mdの内容を追加
echo "## コーディングルール" >> "$TMP_FILE"
echo "" >> "$TMP_FILE"
cat "$CURSOR_DIR/rules.md" >> "$TMP_FILE"
echo "" >> "$TMP_FILE"

# memory.mdの内容を追加
echo "## プロジェクトの記憶" >> "$TMP_FILE"
echo "" >> "$TMP_FILE"
cat "$CURSOR_DIR/memory.md" >> "$TMP_FILE"

# 出力ファイルに移動
mv "$TMP_FILE" "$OUTPUT_FILE"

# 実行権限を付与
chmod 644 "$OUTPUT_FILE"

echo "✅ .cursor/rules/cursorrules.mdcファイルが正常に生成されました: $OUTPUT_FILE"
echo "ファイルサイズ: $(du -h "$OUTPUT_FILE" | cut -f1)"
echo ""
echo "注意: このファイルは自動生成されています。変更する場合は元のmdファイル（prompt.md、rules.md、memory.md）を編集してください。" ```

#### `update_rules.sh`

このスクリプトは、Gitのpre-commitフックとして使用され、コミット前に`.cursor/rules/cursorrules.mdc`ファイルを自動更新します：

```sh
#!/bin/bash

# スクリプトの説明
echo "=== .cursor/rules/cursorrules.mdcの更新スクリプト ==="
echo "このスクリプトはGitのpre-commitフックとして使用され、コミット前に.cursor/rules/cursorrules.mdcを自動更新します。"
echo ""

# 作業ディレクトリを取得
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(dirname "$(dirname "$SCRIPT_DIR")")"
CURSOR_DIR="$(dirname "$SCRIPT_DIR")"
OUTPUT_FILE="$CURSOR_DIR/rules/cursorrules.mdc"

# generate_rules.shの存在確認
if [ ! -f "$SCRIPT_DIR/generate_rules.sh" ]; then
  echo "エラー: $SCRIPT_DIR/generate_rules.sh が見つかりません。"
  exit 1
fi

# generate_rules.shを実行
bash "$SCRIPT_DIR/generate_rules.sh"

# 更新されたファイルをGitに追加
git add "$OUTPUT_FILE"

echo "✅ .cursor/rules/cursorrules.mdcファイルが更新され、Gitに追加されました。"
````

#### `setup_git_hooks.sh`

このスクリプトは、Gitのpre-commitフックを設定します：

```sh
#!/bin/bash

# スクリプトの説明
echo "=== Gitフック設定スクリプト ==="
echo "このスクリプトはGitのpre-commitフックを設定し、コミット前に.cursor/rules/cursorrules.mdcを自動更新します。"
echo ""

# 作業ディレクトリを取得
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(dirname "$(dirname "$SCRIPT_DIR")")"

# .gitディレクトリの確認
if [ ! -d "$PROJECT_ROOT/.git" ]; then
  echo "エラー: .gitディレクトリが見つかりません。Gitリポジトリ内で実行してください。"
  exit 1
fi

# hooksディレクトリの確認
HOOKS_DIR="$PROJECT_ROOT/.git/hooks"
if [ ! -d "$HOOKS_DIR" ]; then
  echo "エラー: $HOOKS_DIR ディレクトリが見つかりません。"
  exit 1
fi

# pre-commitフックの作成
PRE_COMMIT_HOOK="$HOOKS_DIR/pre-commit"
cat > "$PRE_COMMIT_HOOK" << 'EOF'
#!/bin/bash

# pre-commitフック
# コミット前に.cursor/rules/cursorrules.mdcを自動更新します

# スクリプトのパスを取得
SCRIPT_PATH="$(git rev-parse --show-toplevel)/.cursor/sh/update_rules.sh"

# スクリプトの存在確認
if [ ! -f "$SCRIPT_PATH" ]; then
  echo "エラー: $SCRIPT_PATH が見つかりません。"
  exit 1
fi

# スクリプトを実行
bash "$SCRIPT_PATH"

# スクリプトの実行結果を確認
if [ $? -ne 0 ]; then
  echo "エラー: .cursor/rules/cursorrules.mdcの更新に失敗しました。"
  exit 1
fi

# 正常終了
exit 0
EOF

# 実行権限を付与
chmod +x "$PRE_COMMIT_HOOK"

echo "✅ pre-commitフックが正常に設定されました: $PRE_COMMIT_HOOK"
echo "これにより、コミット前に.cursor/rules/cursorrules.mdcファイルが自動的に更新されます。"
```

これらのスクリプトにより、`prompt.md`、`rules.md`、`memory.md`の内容が自動的に`.cursor/rules/cursorrules.mdc`ファイルに反映され、Gitコミット時に常に最新の状態が維持されます。また、後方互換性のために`.cursorrules`ファイルも生成するオプションが含まれています。

ちなみにこれらのスクリプトもAIに出力させました。便利ですね。

## 実践例：AIアシスタントとの効率的な協業

### コード生成と編集

例えば、こんな感じでブログカードコンポーネントを生成できます：

```
アトミックデザインのmoleculesカテゴリに、ブログ記事カードコンポーネントを実装してください。Tailwindのスタイリングパターンに従い、画像、タイトル、タグを表示できるようにしてください。
```

AIアシスタントは、`.cursor/rules/cursorrules.mdc`ファイルに定義されたルールに基づいて、以下のようなコードを生成してくれます：

```tsx
import Image from "next/image";
import Link from "next/link";
import { motion } from "framer-motion";
import { ExternalLink } from "lucide-react";

interface Blog {
  id: number;
  title: string;
  tags: string[];
  link: string;
  image?: string;
}

interface BlogCardProps {
  blog: Blog;
}

const BlogCard = ({ blog }: BlogCardProps) => {
  return (
    <Link href={blog.link} target="_blank" className="block h-full">
      <div className="group relative h-full bg-gradient-to-br from-card to-card/90 rounded-xl overflow-hidden border border-border">
        <div className="absolute inset-0 bg-gradient-to-br from-blue-600/5 to-purple-600/5 dark:from-blue-500/5 dark:to-purple-500/5 opacity-0 group-hover:opacity-100 transition-opacity duration-500" />

        <div className="flex flex-col h-full">
          {/* 画像部分 */}
          <div className="aspect-video bg-muted relative overflow-hidden">
            {blog.image ? (
              <Image
                src={blog.image}
                alt={blog.title}
                width={500}
                height={300}
                className="object-cover w-full h-full transition-transform duration-500 group-hover:scale-110"
              />
            ) : (
              <div className="w-full h-full bg-gradient-to-br from-blue-600/20 to-purple-600/20 dark:from-blue-800/20 dark:to-purple-800/20 flex items-center justify-center">
                <span className="text-4xl">📝</span>
              </div>
            )}

            {/* オーバーレイグラデーション */}
            <div className="absolute inset-0 bg-gradient-to-t from-background to-transparent opacity-60" />

            {/* タグ */}
            {blog.tags.length > 0 && (
              <div className="absolute top-3 left-3 flex flex-wrap gap-2">
                {blog.tags.map((tag, index) => (
                  <motion.span
                    key={index}
                    initial={{ opacity: 0, y: -10 }}
                    animate={{ opacity: 1, y: 0 }}
                    transition={{ delay: index * 0.1 + 0.2 }}
                    className="bg-blue-100 backdrop-blur-sm text-blue-800 dark:bg-blue-600/20 dark:text-blue-300 text-xs px-2 py-1 rounded-full border border-blue-300 dark:border-blue-600/30"
                  >
                    {tag}
                  </motion.span>
                ))}
              </div>
            )}
          </div>

          {/* コンテンツ部分 */}
          <div className="p-6 flex flex-col flex-grow">
            <h3 className="text-lg font-bold mb-3 line-clamp-2 text-foreground group-hover:text-blue-700 dark:group-hover:text-blue-400 transition-colors">
              {blog.title}
            </h3>

            <div className="mt-auto pt-4">
              <span className="inline-flex items-center text-sm text-blue-700 dark:text-blue-400 group-hover:text-blue-600 dark:group-hover:text-blue-300 transition-colors">
                <span>記事を読む</span>
                <ExternalLink className="ml-1 h-3 w-3" />
              </span>
            </div>
          </div>
        </div>

        {/* ホバーエフェクト用の装飾 */}
        <div className="absolute inset-0 border border-blue-500/0 group-hover:border-blue-600/30 rounded-xl transition-all duration-300" />
      </div>
    </Link>
  );
};

export default BlogCard;
```

このように、AIアシスタントはプロジェクトのコーディングルールに従って、高品質なコードを生成してくれます。

他にも、例えばこんな指示を出すだけで、機能を追加できます：

```
スキルセクションにスクロールアニメーションを追加してください。Framer Motionを使用して、要素が画面に入ったときにフェードインするエフェクトを実装してください。
```

```
コンタクトフォームにバリデーションを追加し、送信中と送信完了時の状態を表示するようにUIを改善してください。
```

AIアシスタントは、プロジェクトの文脈を理解した上で、これらの機能を適切に実装してくれます。

## 私のポートフォリオ開発体験：0→1から1→10へ

ここでは、この手法を活用して開発したポートフォリオサイトについて、私自身の体験を共有します。

### 0→1：基盤構築は自分の手で

ポートフォリオサイトの開発を始めた当初、基本的な構造やコアとなる部分は自分自身の手でコーディングしました。もちろん、Cursorエディタのコード補完機能（タブキーでのサジェスト）は活用していましたが、基本的には自分でコードを書いていました。

具体的には：

- プロジェクトの初期設定（Next.js, TypeScript, Tailwind CSSの導入）
- 基本的なディレクトリ構造の設計
- コアとなるコンポーネント（ヘッダー、フッター、レイアウト）の実装
- アトミックデザインの基本構造の構築
- テーマ設定やグローバルスタイルの定義

これらの「0→1」の部分は、自分の手で丁寧に構築することで、プロジェクトの基盤をしっかりと作ることができました。

### 1→10：AIアシスタントとの協業で加速

基本的な構造ができた後の「1→10」の部分、つまり機能拡張やUI改善、コンポーネントの追加などは、ほとんどCursorのAIアシスタントに指示を与えるだけで実装することができました。

特に`.cursor/rules/cursorrules.mdc`の仕組みを導入してからは、AIアシスタントがプロジェクトの文脈を理解し、一貫性のあるコードを生成してくれるようになりました。例えば：

- 「このサイトで使用している技術と工夫した点について紹介してください」
- 「コンタクトフォームを実装してください」
- 「テストコードを実装してください」

といった指示だけで、プロジェクトのコーディングルールに従った高品質なコードを生成してくれました。また、バグ修正やリファクタリングも、AIアシスタントに任せることができました。

### 生産性向上

この方法で開発を進めた結果、生産性の向上を実感しました：

- AIが出力するコードの品質と一貫性が向上
- 試行錯誤のサイクルが高速化

特に印象的だったのは、テストコードを実装した後自分でテストコマンドを実行しエラーが出た箇所をどんどん自分で修正していってくれたことです。(Yoloモードにする必要あり)

## 大LLM時代における開発者の次のステージ

大規模言語モデル（LLM）の時代において、開発者の役割は大きく変化しているように実感しています。従来の「すべてのコードを自分で検索したりして書く」というアプローチから、「AIに指示を出してコードを生成させる」という新しいパラダイムへの移行が進んでいます。

これらはもう避けようがなく、以下に効率的にAIを活用するかがキーになってきているように感じます。

### AIとの協働による学習と成長

AIとの協働は単なる効率化だけでなく、開発者自身の学習と成長にも寄与します。
今回実感したのですがAIが提案する実装パターンやライブラリ、テクニックの中には絶対に自分だけで実装していたら気づかなかったり見逃していたものがあると思います。
そのため自分の学習を加速させる手段としても非常に有効です。

ただし、AIはあくまでも強力な協力者であり、最終的な責任は開発者にあることを忘れてはなりません。AIが生成したコードを盲目的に採用するのではなく、その内容を理解し、プロジェクトの要件や品質基準に合致しているかを常に評価する姿勢が重要です。

## まとめ：AIアシスタントとの効率的な協業に向けて

この記事では、Cursorに搭載されたAIアシスタントとの効率的な協業方法として、「AIの記憶管理」と「コーディングルールの自動化」について紹介しました。これらの手法を活用すると、こんな効果が期待できます。

1. **AIの文脈理解の向上**：`.cursor/memory.md`ファイルによって、AIは過去のセッションで行ったことや次回以降気をつけるべきことや重要な実装パターンを理解できるようになります。

2. **一貫性のあるコード生成**：`.cursor/rules.md`ファイルによって、AIはプロジェクト固有のコーディングルールに従ったコードを生成できるようになります。

3. **AIへの指示の標準化**：`.cursor/prompt.md`ファイルによって、AIへの指示を一貫して維持できるようになります。

4. **知識の蓄積と共有**：解決した問題や重要な実装パターンを記録しアップデートを続けていくことで、AIもPJに合わせたコードを生成してくれます。

5. **自動化による効率化**：Gitフックを活用した自動更新により、ルールの管理が効率化されます。

AIアシスタントは単なるコード生成ツールではなく、プロジェクトの文脈を理解したパートナーとして活用することで、その真価を発揮します。ぜひ自分のプロジェクトに合わせてカスタマイズして、AIとの協業をさらに効率化してみてください。

## 追記：Cursorのルールシステムのアップデートについて

Cursorのルールシステムは常に進化しています。この記事を執筆した時点では、`.cursor/rules/*.mdc`形式が推奨されており、従来の`.cursorrules`ファイルは後方互換性のために引き続きサポートされていますが、将来的には完全に非推奨となる可能性があります。

新しいルールシステムでは、複数の`.mdc`ファイルを`.cursor/rules/`ディレクトリに配置することで、より柔軟にAIの動作をカスタマイズできるようになっています。これにより、プロジェクトの異なる側面に対して個別のルールファイルを作成することが可能になりました。

最新の情報については、[Cursor公式ドキュメント](https://cursor.sh/docs/rules)を参照することをお勧めします。

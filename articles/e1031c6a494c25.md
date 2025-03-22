---
title: "Next.jsでマークダウン記事を表示する方法"
emoji: "📝"
type: "tech"
excerpt: "Next.jsプロジェクトにマークダウンファイルを組み込み、ブログ記事として表示する方法を解説します。"
topics: ["Next.js", "Markdown", "React", "TypeScript"]
published: true
published_at: "2025-03-22"
---

今回はNext.jsでマークダウンベースのブログシステムを構築する際の実装テクニックを解説します。

ZennやQiitaなどのブログサービスに投稿するのもいいですが開発者ならば一度は自分のブログを持ってみたくなるはずです。

そんな時にこの記事でも使っているマークダウンファイルをNext.jsで作られているサイトに組み込む方法が役にたつはずです。

## なぜマークダウンベースのブログを構築するのか

マークダウン形式でのブログコンテンツ管理には、いくつかの重要なメリットがあります：

- **シンプルな執筆体験**: 余計なツールなしで執筆に集中できる
- **バージョン管理の容易さ**: Gitで履歴管理がしやすいプレーンテキスト
- **将来の移行のしやすさ**: 特定のCMSに依存しないデータ形式
- **カスタマイズの自由度**: 独自の表示や処理を柔軟に実装可能

もちろん、[Contentlayer](https://contentlayer.dev/)や[MDX](https://mdxjs.com/)など他の選択肢もありますが、純粋なマークダウンとNext.jsの組み合わせは、学習コストと実装の自由度のバランスに優れています。

実際にこの記事自体もNext.jsとMarkdownを使用して作成していますので以下のリポジトリをご覧ください。

[https://github.com/daikimatsuura/portfolio](https://github.com/daikimatsuura/portfolio)

## この記事で学べること

- マークダウン処理の効率化手法
- App Router環境でのサーバー/クライアント分離パターン
- IntersectionObserverを活用した目次実装
- Tailwind Typographyのカスタマイズ
- パフォーマンス最適化テクニック

## ディレクトリ構造

マークダウンコンテンツを管理するためのこのサイトでは以下のようなディレクトリ構造を採用しています：

```
project-root/
├── article/
│   └── contents/         # マークダウン記事ファイル
├── src/
│   ├── components/
│   │   ├── molecules/
│   │   │   └── TableOfContents/ # 目次コンポーネント
│   │   └── organisms/
│   │       └── BlogPostContent/ # ブログ記事表示コンポーネント
│   ├── lib/
│   │   └── markdown/     # マークダウン処理ユーティリティ
│   ├── types/
│   │   └── markdown.d.ts # 型定義
│   └── app/
│       └── blog/
│           ├── page.tsx  # ブログ一覧ページ
│           └── [slug]/
│               ├── page.tsx      # サーバーコンポーネント
│               └── page.client.tsx # クライアントコンポーネント
```

この構造では、コンテンツとロジックを明確に分離し、アトミックデザインパターンに基づいてコンポーネントを整理しています。特にサーバーコンポーネントとクライアントコンポーネントを分離することで、パフォーマンスとインタラクティビティの両立が可能になります。

また、CMS連携に移行する際も記事コンテンツとUIロジックが分離されているため、マイグレーションが容易です。

## マークダウン処理の実装

マークダウン処理の核となる部分を詳しく見ていきましょう。実装にあたっては、以下の課題に対処する必要があります：

1. マークダウンファイルからメタデータと本文を分離する
2. 本文をHTMLに変換し、目次用のIDを自動生成する
3. 日本語文字を含む見出しへの適切なID付与
4. 異なるフロントマターフォーマットの互換性を確保する

これらの課題に対処するコードを見ていきましょう：

```typescript
import fs from "fs";
import path from "path";
import matter from "gray-matter";
import { remark } from "remark";
import html from "remark-html";
import { MarkdownPost, MarkdownPostMeta } from "@/types/markdown";

const postsDirectory = path.join(process.cwd(), "article", "contents");

export async function getPostBySlug(slug: string): Promise<MarkdownPost> {
  const fullPath = path.join(postsDirectory, `${slug}.md`);
  const fileContents = fs.readFileSync(fullPath, "utf8");

  // メタデータと本文を分離
  const { data, content } = matter(fileContents);

  // Markdownから安全なHTMLへ変換
  const processor = remark().use(html, {
    sanitize: false, // HTML属性を維持（重要）
  });

  const processedContent = await processor.process(content);
  const htmlContent = processedContent.toString();

  // 見出しに自動的にIDを付与（目次機能用）
  const headingRegex = /<h([1-6])>(.*?)<\/h\1>/g;
  const htmlWithIds = htmlContent.replace(
    headingRegex,
    (match, level, text) => {
      // 日本語文字を保持したまま、特殊文字を適切に処理
      const id = text
        .toLowerCase()
        .trim()
        .replace(/[\s\t\n]+/g, "-") // 空白をハイフンに
        .replace(/[!"#$%&'()*+,./:;<=>?@[\\\]^`{|}~]/g, "") // 記号を削除
        .replace(/-+/g, "-") // 連続ハイフンを単一に
        .replace(/^-|-$/g, ""); // 先頭・末尾のハイフンを削除

      const finalId = id || `heading-${level}-${Date.now()}`;
      return `<h${level} id="${finalId}">${text}</h${level}>`;
    }
  );

  // 型安全な返却値
  return {
    slug,
    content: htmlWithIds,
    title: data.title,
    published_at: data.published_at || data.date, // 互換性
    excerpt: data.excerpt,
    topics: data.topics || data.tags || [], // 互換性
    coverImage: data.coverImage,
    emoji: data.emoji,
    published: data.published,
  };
}
```

### コード解説と実装上の注意点

このコードは単純に見えますが、いくつかの重要なポイントがあります：

1. **`sanitize: false`の使用**: デフォルトではHTMLタグなどはサニタイズされますが、これを無効にしています。セキュリティ上のリスクがあるため、信頼できる執筆者のみがコンテンツを作成する環境でのみこの設定を使用してください。

2. **正規表現による見出し処理**: HTMLに変換された後に正規表現で見出しを検出しています。この方法はシンプルですが、ネストされたタグを含む複雑なHTML構造には対応できない制限があります。より堅牢な実装が必要な場合は、DOMパーサーを使用するアプローチも検討すべきです。

3. **ID生成ロジック**: 日本語対応のID生成は意外と複雑です。URLに使用できない文字を除去しつつ、日本語自体は保持するロジックになっています。ただし、完全に同じテキストの見出しが複数ある場合、IDの衝突が発生するため注意が必要です。

フロントマターは互換性のためZennのマークダウン記法を参考にしました。

### 記事メタデータの効率的な取得

ブログ一覧表示など、多数の記事のメタデータのみを取得する場合は、本文の変換を行わないことで処理を効率化できます：

```typescript
export function getAllPostsMeta(): MarkdownPostMeta[] {
  const fileNames = fs.readdirSync(postsDirectory);

  const allPostsData = fileNames
    .filter((fileName) => fileName.endsWith(".md"))
    .map((fileName) => {
      const slug = fileName.replace(/\.md$/, "");
      const fullPath = path.join(postsDirectory, fileName);
      const fileContents = fs.readFileSync(fullPath, "utf8");

      // コンテンツは変換せず、メタデータのみを取得
      const { data } = matter(fileContents);

      return {
        slug,
        title: data.title,
        published_at: data.published_at || data.date,
        excerpt: data.excerpt,
        topics: data.topics || data.tags || [],
        coverImage: data.coverImage,
        emoji: data.emoji,
        published: data.published,
      };
    })
    // 非公開記事と未来の記事を除外
    .filter((post) => post.published)
    .filter((post) => {
      if (!post.published_at) return true;
      return new Date(post.published_at) <= new Date();
    });

  // 公開日降順でソート
  return allPostsData.sort((a, b) =>
    a.published_at < b.published_at ? 1 : -1
  );
}
```

このアプローチのパフォーマンス上の利点は3つあります：

1. **選択的処理**: 必要なメタデータのみを抽出し、本文のマークダウン変換という重い処理をスキップ
2. **メモリ効率**: 大量の記事がある場合でも、メモリ使用量を抑えられる
3. **フィルタリングの最適化**: 公開状態や公開日のフィルタリングをメタデータ段階で行うことで、不要な処理を回避

注意点として、ファイルシステムの読み取りは同期的に行われているため、記事数が非常に多い場合はパフォーマンスに影響が出る可能性があります。そのような場合は、キャッシュ機構（後述）の導入を検討すべきです。

## App Routerでの実装パターン

Next.jsのApp Routerを活用した動的ルーティングの実装について詳しく見ていきます。サーバーコンポーネントとクライアントコンポーネントを分離する構成は、パフォーマンスとインタラクティビティを両立するために重要です。

まず、サーバーコンポーネントの実装を見てみましょう：

```typescript
// src/app/blog/[slug]/page.tsx (サーバーコンポーネント)
export async function generateStaticParams() {
  const slugs = getAllPostSlugs();
  return slugs.map((slug) => ({ slug }));
}

export async function generateMetadata({
  params,
}: BlogPostProps): Promise<Metadata> {
  try {
    const resolvedParams = await params;
    const post = await getPostBySlug(resolvedParams.slug);

    return {
      title: post.title,
      keywords: post.topics,
      description: post.excerpt || `${post.title}の詳細記事です。`,
      ...(post.coverImage && {
        openGraph: {
          images: [post.coverImage],
        },
      }),
    };
  } catch (error) {
    return {
      title: "ブログ記事",
      description: "ブログ記事のページです。",
    };
  }
}

export default async function BlogPostPage({ params }: BlogPostProps) {
  try {
    const resolvedParams = await params;
    const post = await getPostBySlug(resolvedParams.slug);

    if (post.published === false) {
      notFound();
    }

    // データをクライアントコンポーネントに渡す
    return <BlogPostClient post={post} />;
  } catch (error) {
    notFound();
  }
}
```

次に、クライアントコンポーネントを見てみましょう：

```typescript
// src/app/blog/[slug]/page.client.tsx (クライアントコンポーネント)
"use client";

import { useRef } from "react";
import { MarkdownPost } from "@/types/markdown";
import { BlogPostContent } from "@/components/organisms/BlogPostContent";

interface BlogPostClientProps {
  post: MarkdownPost;
}

export function BlogPostClient({ post }: BlogPostClientProps) {
  const contentRef = useRef<HTMLDivElement>(null);
  return <BlogPostContent post={post} contentRef={contentRef} />;
}
```

### 実装ポイントと注意点

この設計には次のような特徴があります：

1. **サーバー/クライアント分離**: データ取得とメタデータ生成はサーバーで、インタラクティブ要素はクライアントで処理します。これにより、初期ページロードのパフォーマンスが向上します。

2. **静的生成の活用**: `generateStaticParams`メソッドによってビルド時に静的ページを生成します。これにより、訪問者はサーバー処理を待つことなく瞬時にページを閲覧できます。

3. **SEO対応**: `generateMetadata`メソッドにより、各記事に適したメタデータとOGPタグを動的に生成します。これはSEOにおいて重要なファクターです。

4. **エラーハンドリング**: 記事が存在しない場合や非公開の場合は`notFound()`関数を使用して404ページにリダイレクトします。

実装時の注意点として、以下の点に留意する必要があります：

- **データの受け渡し**: サーバーコンポーネントからクライアントコンポーネントへのデータ受け渡しはシリアライズ可能な値に限られます。複雑なオブジェクトや関数は渡せない点に注意してください。

- **実行タイミング**: `generateStaticParams`はビルド時のみ実行されるため、新しい記事を追加した場合はビルドし直すか、後述するISRを設定する必要があります。

- **代替アプローチ**: App Router以前のPages Routerを使用する場合は、`getStaticPaths`と`getStaticProps`を使って同様の機能を実現できます。また、Contentlayerなどのライブラリを使用すると、型安全性がさらに向上します。

## IntersectionObserverを活用した目次実装

記事内の見出しを自動検出し、スクロール追従する目次機能の実装について詳しく解説します。このコンポーネントは次の3つの主要機能を持ちます：

1. 記事コンテンツから見出し（h1〜h6）を検出して階層構造化
2. 現在のスクロール位置に応じて、表示中の見出しをハイライト
3. 目次項目クリック時に対応する見出しへスムーズにスクロール

実装コードを詳しく見ていきましょう：

```typescript
export const TableOfContents = ({ contentRef }: TableOfContentsProps) => {
  // 見出し情報と現在アクティブな見出しIDの状態
  const [headings, setHeadings] = useState<Heading[]>([]);
  const [activeId, setActiveId] = useState<string>("");

  useEffect(() => {
    // contentRefが設定されていない場合は処理しない
    if (!contentRef.current) return;

    const contentElement = contentRef.current;
    // すべての見出し要素を取得
    const headingElements = Array.from(
      contentElement.querySelectorAll("h1, h2, h3, h4, h5, h6")
    );

    // 見出し情報を抽出し、必要に応じてIDを自動生成
    const allHeadingsData = headingElements.map((heading) => {
      if (!heading.id) {
        // IDがない場合は自動生成（詳細コードは省略）
        // ...
      }

      return {
        id: heading.id,
        text: heading.textContent?.trim() || "",
        level: parseInt(heading.tagName.charAt(1)),
      };
    });

    // フィルタリングして不要な見出しを除外
    const finalHeadings = allHeadingsData
      .filter((heading) => heading.id && heading.text)
      .filter((heading) => !heading.text.match(/^目次$/i));

    setHeadings(finalHeadings);

    // IntersectionObserverの設定
    // この設定は実際の使用環境（ヘッダーの高さなど）に応じて調整する必要がある
    const observerOptions = {
      rootMargin: "-80px 0px -70% 0px", // 上部に80px、下部に70%のマージン
      threshold: [0.1, 0.5, 0.9], // 複数のしきい値で正確な検出
    };

    // スクロール位置の検出と現在の見出し更新
    const headingObserver = new IntersectionObserver((entries) => {
      // 画面内に表示されている見出しを抽出
      const visibleHeadings = entries
        .filter((entry) => entry.isIntersecting)
        .map((entry) => entry.target.id);

      if (visibleHeadings.length > 0) {
        // 最初の可視見出しをアクティブとして設定
        setActiveId(visibleHeadings[0]);
      }
    }, observerOptions);

    // 各見出し要素を監視
    headingElements.forEach((heading) => {
      if (heading.id) {
        headingObserver.observe(heading);
      }
    });

    // クリーンアップ関数（コンポーネントアンマウント時に実行）
    return () => {
      headingElements.forEach((heading) => {
        if (heading.id) {
          headingObserver.unobserve(heading);
        }
      });
    };
  }, [contentRef]); // contentRefが変更されたときのみ実行

  // 見出しがない場合は何も表示しない
  if (headings.length === 0) return null;

  return (
    <nav className="w-full">
      <ul className="space-y-1">
        {headings.map((heading) => (
          <li
            key={heading.id}
            className={cn(
              "transition-colors",
              heading.level === 1 && "mt-3 font-semibold",
              heading.level === 2 && "font-medium",
              heading.level === 3 && "pl-3",
              heading.level === 4 && "pl-5 text-sm",
              heading.level >= 5 && "pl-6 text-xs"
            )}
          >
            <a
              href={`#${heading.id}`}
              className={cn(
                "block py-1.5 border-l-2 pl-3 hover:text-primary transition-colors",
                activeId === heading.id
                  ? "border-primary text-primary font-medium bg-primary/5"
                  : "border-transparent text-muted-foreground"
              )}
              onClick={(e) => {
                e.preventDefault();
                const element = document.getElementById(heading.id);
                if (element) {
                  // スクロール制御
                  window.scrollTo({
                    top: element.offsetTop - 100, // ヘッダー高さ考慮
                    behavior: "smooth",
                  });
                  // URLハッシュ更新（ブックマーク対応）
                  history.pushState(null, "", `#${heading.id}`);
                  setActiveId(heading.id);
                }
              }}
            >
              {heading.text}
            </a>
          </li>
        ))}
      </ul>
    </nav>
  );
};
```

### 実装の詳細と潜在的な課題

このIntersectionObserver実装には、いくつかの重要な考慮点があります：

1. **rootMarginの調整**: `-80px 0px -70% 0px`というrootMarginは画面上部の固定ヘッダーと、下部70%を見出し検出から除外するための設定です。この値はサイトのレイアウトに応じて調整する必要があります。

2. **複数のthreshold値**: 単一の閾値ではなく複数（`[0.1, 0.5, 0.9]`）を設定することで、スクロール中の検出精度を向上させています。これにより、特に短い見出しの検出が改善されます。

3. **パフォーマンスへの影響**: headingElementsの数が多い場合（非常に長い記事など）、多数のIntersectionObserverがアクティブになり、パフォーマンスに影響を与える可能性があります。必要に応じて、監視する見出しの数を制限する方法も検討してください。

4. **URLハッシュとの同期**: 目次項目クリック時にURLハッシュを更新しますが、ブラウザのバック/フォワードボタンでの移動時にアクティブ項目と同期されないケースがあります。完全な対応には、`popstate`イベントリスナーの追加が必要です。

代替アプローチとして、以下の方法も考慮できます：

- スクロール位置計算ベースの実装：IntersectionObserverの代わりに、スクロールイベントと各見出しの位置計算による実装も可能です。ただし、パフォーマンス面でIntersectionObserverより劣ります。

- ライブラリの利用：`react-scrollspy`などのライブラリを使用すると、実装が簡略化できる可能性があります。ただし、カスタマイズ性は低下します。

この実装のもう一つの特徴は、見出しレベルに応じた視覚的階層構造（インデント、フォントサイズ変更）を提供している点です。これにより、目次が記事の構造を視覚的に表現し、ユーザーの理解を助けます。

## レスポンシブ対応の記事レイアウト

デスクトップとモバイルの両方で使いやすい表示を提供するレイアウト実装について解説します。記事レイアウトでは、可読性とナビゲーション（目次）の両立が重要な課題です。

```tsx
export const BlogPostContent = ({ post, contentRef }: BlogPostContentProps) => {
  return (
    <div className="container max-w-screen-xl mx-auto px-4 py-12">
      <div className="flex flex-col lg:flex-row lg:gap-8">
        {/* サイドバー（デスクトップ表示時） */}
        <div className="lg:w-64 xl:w-72 flex-shrink-0">
          <div className="lg:sticky lg:top-24 space-y-6">
            <Link
              href="/blog"
              className="inline-flex items-center text-primary hover:underline"
            >
              <ArrowLeft className="w-4 h-4 mr-2" />
              ブログ一覧に戻る
            </Link>

            {/* デスクトップ用目次 */}
            <div className="hidden lg:block">
              <h2 className="text-xl font-bold mb-4">目次</h2>
              <TableOfContents contentRef={contentRef} />
            </div>
          </div>
        </div>

        {/* メインコンテンツ */}
        <article className="flex-1 max-w-3xl">
          {/* モバイル用目次 */}
          <div className="lg:hidden mb-8">
            <h2 className="text-xl font-bold mb-4">目次</h2>
            <TableOfContents contentRef={contentRef} />
          </div>

          {/* 記事メタデータ */}
          <div className="mb-8">
            <h1 className="text-3xl md:text-4xl font-bold mb-4">
              {post.title}
            </h1>
            <time className="text-muted-foreground">{post.published_at}</time>

            {/* カバー画像または絵文字 */}
            {post.coverImage ? (
              <div className="mb-8 aspect-video w-full relative rounded-lg overflow-hidden">
                <Image
                  src={post.coverImage}
                  alt={post.title}
                  fill
                  className="object-cover"
                  priority
                />
              </div>
            ) : post.emoji ? (
              <div className="mb-8 flex justify-center items-center py-12 bg-muted/20 rounded-lg">
                <span className="text-9xl">{post.emoji}</span>
              </div>
            ) : null}

            {/* トピックタグ */}
            {post.topics && post.topics.length > 0 && (
              <div className="flex flex-wrap gap-2 mb-8">
                {post.topics.map((topic, index) => (
                  <span
                    key={index}
                    className="px-3 py-1 text-sm rounded-full bg-primary/10 text-primary"
                  >
                    {topic}
                  </span>
                ))}
              </div>
            )}
          </div>

          {/* 記事本文 */}
          <div
            ref={contentRef}
            className="prose prose-lg dark:prose-invert max-w-none"
            dangerouslySetInnerHTML={{ __html: post.content }}
          />
        </article>
      </div>
    </div>
  );
};
```

### レイアウト実装の重要ポイントと課題

このレイアウト実装では、いくつかの重要なテクニックを使用しています：

1. **レスポンシブなフレックスレイアウト**: モバイルでは縦方向（`flex-col`）、デスクトップでは横方向（`lg:flex-row`）に配置を切り替えています。これにより、画面サイズに応じた最適なスペース利用が可能になります。

2. **コンテンツの条件付き表示**: 目次が2箇所（モバイル用とデスクトップ用）に記述されていますが、`hidden lg:block`と`lg:hidden`の組み合わせにより、画面サイズに応じて適切な方だけが表示されます。

3. **スティッキーポジショニング**: デスクトップ表示では、サイドバーに`lg:sticky lg:top-24`を適用し、スクロール時も目次が画面内に留まるようにしています。

4. **画像表示の最適化**: `priority`属性によるLCP（Largest Contentful Paint）の改善や`aspect-video`による比率維持を実装しています。

実装時に直面する可能性のある課題：

1. **目次の重複コード**: 同じTableOfContentsコンポーネントを2回レンダリングしているため、特に長い記事では初期ロード時のパフォーマンスに影響する可能性があります。状況によっては、単一のコンポーネントをCSSで位置変更する方法も検討可能です。

2. **スティッキーポジショニングの互換性**: 一部の古いブラウザでは`position: sticky`のサポートが限定的です。必要に応じてフォールバックを実装するか、ポリフィルの使用を検討してください。

3. **アクセシビリティの考慮**: この実装では次の点に注意が必要です：

   - 目次が2つあるため、スクリーンリーダーでは重複して読み上げられる可能性
   - 適切な`aria-hidden`属性の追加を検討すべき
   - トピックタグが装飾目的のみの場合は、適切なセマンティックマークアップに変更すべき

4. **コンテンツ幅の制御**: 記事が非常に長い段落や幅広いテーブルを含む場合、モバイル表示で横スクロールが発生する可能性があります。`overflow-x-auto`などの追加対策が必要になることがあります。

5. **画像最適化のトレードオフ**: `fill`と`object-cover`の組み合わせは美観を保つ一方で、重要な画像部分が切れる可能性があります。コンテンツによっては、`object-contain`の使用や固定サイズの設定が適切な場合もあります。

## Tailwind Typographyのカスタマイズ

マークダウンコンテンツのスタイリングを効果的に行うためのTailwind Typography設定について解説します。この設定により、一貫性のあるタイポグラフィとデザイン要素を実現できます。

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      typography: {
        DEFAULT: {
          css: {
            maxWidth: "100%",
            color: "hsl(var(--foreground))",
            a: {
              color: "hsl(var(--primary))",
              "&:hover": {
                color: "hsl(var(--primary))",
              },
            },
            h1: { color: "hsl(var(--foreground))" },
            h2: { color: "hsl(var(--foreground))" },
            h3: { color: "hsl(var(--foreground))" },
            h4: { color: "hsl(var(--foreground))" },
            code: {
              color: "hsl(var(--foreground))",
              backgroundColor: "hsl(var(--accent))",
              borderRadius: "0.25rem",
              padding: "0.15rem 0.3rem",
            },
            blockquote: {
              borderLeftColor: "hsl(var(--primary))",
              backgroundColor: "hsl(var(--accent) / 0.2)",
              color: "hsl(var(--foreground))",
            },
            pre: {
              backgroundColor: "hsl(var(--accent) / 0.8)",
              color: "hsl(var(--foreground))",
              overflow: "auto",
            },
          },
        },
      },
    },
  },
  plugins: [require("tailwindcss-animate"), require("@tailwindcss/typography")],
};
```

### カスタマイズの特徴と実装上の考慮点

この設定には以下のような特徴があります：

1. **CSS変数の活用**: `hsl(var(--foreground))`のような形式でCSS変数を使用しています。これにより、テーマ変更（ダークモード/ライトモードなど）に容易に対応できます。

2. **要素別のスタイル定義**: 見出し、リンク、コードブロック、引用などの要素ごとに個別のスタイルを定義しています。これにより、マークダウンの各要素が一貫したデザインで表示されます。

3. **ダークモード対応**: コンポーネントの使用時に`dark:prose-invert`クラスを追加することで、ダークモードに対応できます。

実装時に考慮すべき点：

1. **カスタマイズの範囲**: Tailwind Typographyは多くの要素に対するデフォルトスタイルを提供しますが、この例では一部の要素のみをカスタマイズしています。実際のプロジェクトでは、リスト（`ul`, `li`）やテーブル（`table`, `th`, `td`）なども必要に応じてカスタマイズするとよいでしょう。

2. **フォントサイズの調整**: この例では`prose-lg`クラスを使用していますが、コンテンツやデザインに応じて`prose-sm`や`prose-base`などの異なるサイズを検討することも重要です。

3. **印刷スタイルの考慮**: 記事を印刷する可能性がある場合は、`@media print`クエリを使用して印刷時のスタイルも定義すると良いでしょう。

4. **アクセシビリティの考慮**: コントラスト比が十分であることを確認し、特にリンク色やコードブロックの背景色などは、WCAGのガイドラインに準拠するよう注意してください。

5. **オーバーライドの問題**: Tailwind Typographyのスタイルは優先度が高く設定されているため、一部のカスタマイズには`!important`が必要になる場合があります。これは最終手段として使用し、できるだけ避けるべきです。

以上のポイントを考慮することで、読みやすく美しいマークダウンコンテンツ表示を実現できます。

## マークダウン処理の拡張

基本実装をさらに強化するための実践的なテクニックを紹介します。ここでは、マークダウン処理のカスタマイズと拡張について、実際のプロジェクトで役立つアプローチを解説します。

### remarkプラグインエコシステムの活用

remarkは豊富なプラグインエコシステムを持っており、様々な機能拡張が可能です。以下は一般的に便利なプラグインの組み合わせ例です：

```typescript
import { remark } from "remark";
import html from "remark-html";
import gfm from "remark-gfm";
import prism from "remark-prism";
import externalLinks from "remark-external-links";

const processedContent = await remark()
  .use(gfm) // GitHub Flavored Markdown
  .use(prism, { plugins: ["line-numbers"] }) // シンタックスハイライト
  .use(externalLinks, { target: "_blank", rel: ["nofollow", "noopener"] }) // 外部リンク処理
  .use(html, { sanitize: false })
  .process(content);
```

各プラグインの役割と利点：

- **remark-gfm**: テーブル、取り消し線、タスクリストなどGitHub Flavored Markdownの機能を追加
- **remark-prism**: コードブロックにシンタックスハイライトを適用（代替としてremark-highlight.jsも選択肢）
- **remark-external-links**: 外部リンクに自動的に`target="_blank"`と適切なセキュリティ属性を追加

プラグイン導入時の注意点：

1. **バンドルサイズ**: 多くのプラグインを追加するとバンドルサイズが増加します。必要なものだけを選択しましょう。
2. **互換性**: プラグイン間の互換性に注意し、必要に応じてバージョンを固定してください。
3. **実行順序**: プラグインの順序によって結果が変わる場合があります。特にHTML変換前後での処理には注意が必要です。

### AST操作によるカスタムトランスフォーマー

より高度なカスタマイズが必要な場合、AST（抽象構文木）を直接操作するカスタムトランスフォーマーを作成できます：

```typescript
import { visit } from "unist-util-visit";

function customTransformer() {
  return (tree) => {
    visit(tree, "paragraph", (node, index, parent) => {
      // 特定のパターン（例: ::note::）を探す
      if (
        node.children &&
        node.children.length === 1 &&
        node.children[0].type === "text"
      ) {
        const text = node.children[0].value;
        const match = text.match(/^::(\w+)::(.*)$/);

        if (match) {
          // マッチした場合、ノードを変換
          const [, type, content] = match;
          node.type = "html";
          node.children = undefined;
          node.value = `<div class="custom-block custom-block-${type}">${content}</div>`;
        }
      }
    });
  };
}
```

このアプローチでは、特定のマークダウンパターンを独自のHTMLに変換できます。例えば、`::note:: これは注意書きです`というテキストを、スタイル付きの注意書きブロックに変換できます。

カスタムトランスフォーマー実装のヒント：

1. **テスト重視**: 複雑なASTの変換は予期せぬ結果を生むことがあるため、十分なテストが必要です。
2. **パフォーマンス考慮**: 大量のマークダウンに対しては処理が重くなる可能性があるため、パフォーマンスに注意してください。
3. **段階的実装**: 一度にすべての機能を実装するのではなく、段階的に機能を追加していくことをおすすめします。

### メタデータバリデーションの強化

マークダウンのフロントマターデータに対するバリデーションを実装することで、コンテンツの一貫性を確保できます：

```typescript
function validateFrontMatter(data) {
  // 必須フィールド検証
  const requiredFields = ["title", "published_at"];
  const missingFields = requiredFields.filter((field) => !data[field]);

  if (missingFields.length > 0) {
    throw new Error(
      `Missing required front matter fields: ${missingFields.join(", ")}`
    );
  }

  // 日付フォーマット検証
  if (data.published_at && !/^\d{4}-\d{2}-\d{2}/.test(data.published_at)) {
    throw new Error("Invalid date format in published_at. Expected YYYY-MM-DD");
  }

  // トピック配列検証
  if (
    data.topics &&
    (!Array.isArray(data.topics) ||
      data.topics.some((topic) => typeof topic !== "string"))
  ) {
    throw new Error("Topics must be an array of strings");
  }
}
```

バリデーション実装の利点：

1. **早期エラー検出**: 開発・執筆段階でのエラー検出により、本番環境での問題を防止
2. **一貫性確保**: すべての記事が一定のフォーマットに従うことを保証
3. **型安全性の向上**: TypeScriptと組み合わせることで、エディタでの自動補完やエラー検出が可能に

### 代替アプローチとの比較

純粋なマークダウン処理以外にも、いくつかの代替アプローチがあります：

1. **MDX**: JSXをマークダウン内に記述できるため、React コンポーネントを直接埋め込めます。より複雑なインタラクティブコンテンツに適していますが、学習曲線が高く、処理も複雑になります。

2. **Contentlayer**: マークダウンデータに対する型安全なアクセスを提供し、開発体験が向上します。ただし、追加の依存関係とビルド設定が必要です。

3. **CMS連携**: HeadlessCMSを使用する場合、APIからのデータ取得に変更する必要がありますが、執筆体験とコンテンツ管理が向上します。

それぞれのアプローチには長所と短所があり、プロジェクトのニーズに応じて選択すべきです。純粋なマークダウン処理は、シンプルさと柔軟性のバランスが優れています。

## パフォーマンス最適化テクニック

マークダウンベースのブログシステムを運用する上で、特に記事数が増えてきた場合に考慮すべきパフォーマンス最適化手法を紹介します。

### LRUキャッシュによる処理効率化

マークダウン処理は比較的重い処理のため、結果をメモリ内にキャッシュすることで繰り返しの変換処理を回避できます：

```typescript
import LRUCache from "lru-cache";

// 設定可能なLRUキャッシュ
const postCache = new LRUCache({
  max: 50, // 最大50記事をキャッシュ
  ttl: 1000 * 60 * 5, // 5分間キャッシュ
  allowStale: true, // ttl後も削除されるまで古い値を返す
  updateAgeOnGet: true, // 取得時に有効期限をリセット
});

export async function getPostBySlug(slug: string): Promise<MarkdownPost> {
  const cacheKey = `post:${slug}`;

  // キャッシュチェック
  const cached = postCache.get(cacheKey);
  if (cached) {
    return cached as MarkdownPost;
  }

  // 通常の処理
  const post = await processMarkdownFile(slug);

  // キャッシュに保存
  postCache.set(cacheKey, post);
  return post;
}
```

LRUキャッシュ実装の注意点：

1. **メモリ使用量**: `max`パラメータを適切に設定し、メモリ使用量を制御する必要があります。
2. **TTL（Time To Live）**: 更新頻度に応じて適切なキャッシュ時間を設定します。
3. **キャッシュ無効化**: コンテンツ更新時にキャッシュを適切に無効化する仕組みが必要です。

### インクリメンタル静的再生成（ISR）

Next.jsのISR機能を活用して、サーバー側でもキャッシュを効果的に管理できます：

```typescript
// App RouterでのISR設定
export const revalidate = 3600; // 1時間ごとに再検証

// または動的なrevalidate設定
export async function generateStaticParams() {
  const allPosts = getAllPostsMeta();

  return allPosts.map((post) => ({
    slug: post.slug,
    // 古い記事ほど長い再検証間隔を設定
    revalidate: isRecentPost(post.published_at) ? 3600 : 86400,
  }));
}

// Revalidate On-Demandの活用
// pages/api/revalidate.ts
export default async function handler(req, res) {
  const { slug, token } = req.query;

  // シークレットトークン検証
  if (token !== process.env.REVALIDATION_TOKEN) {
    return res.status(401).json({ message: "Invalid token" });
  }

  try {
    // 特定のパスを再検証
    await res.revalidate(`/blog/${slug}`);
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send("Error revalidating");
  }
}
```

ISR実装の実践的なヒント：

1. **コンテンツによる差別化**: 頻繁に更新される記事と静的な記事で異なるrevalidate値を設定
2. **On-Demand Revalidation**: コンテンツ更新時に特定のページだけを再検証する仕組みを導入
3. **フォールバック戦略**: 新しい記事へのアクセス時の初回レンダリングをどう扱うかを検討

### ストリーミングSSRとSuspense活用

Next.js 13以降では、ReactのSuspenseを活用したストリーミングSSRが可能になり、初期ロード体験を向上できます：

```tsx
import { Suspense } from "react";
import { BlogPostSkeleton } from "@/components/molecules/BlogPostSkeleton";

export default function BlogLayout({ children }) {
  return (
    <div className="blog-layout">
      <Suspense fallback={<BlogPostSkeleton />}>{children}</Suspense>
    </div>
  );
}
```

このアプローチの利点：

1. **体感速度の向上**: ユーザーは完全なページの読み込みを待たずにスケルトンUIを見ることができる
2. **段階的なレンダリング**: 重要なコンテンツから順に表示され、TTI（Time to Interactive）が向上
3. **異なるデータソースの分離**: 遅いデータソースが他の部分の表示をブロックしない

実装時の考慮点：

1. **スケルトンUIの設計**: ローディング状態のUIが本番コンテンツとサイズやレイアウトが大きく異なると、表示が不安定になる可能性があります（レイアウトシフト）。
2. **Suspenseの粒度**: 細かく分割しすぎると複雑性が増し、大きすぎるとパフォーマンス向上の効果が減少します。
3. **SEOへの影響**: ストリーミングSSRでも適切なメタデータが初期レスポンスに含まれるよう注意してください。

## 実運用から得た知見と実践的なアドバイス

数ヶ月間のマークダウンブログ運用から得た実践的な知見を共有します：

1. **一貫したフロントマター設計**: 最初にフロントマターの仕様をしっかり設計することが重要です。後からの変更は既存コンテンツすべての修正が必要になり、大変な作業になります。

2. **画像の最適化**: 記事内の画像は自動最適化の仕組みを導入するか、事前に最適化しておくことをおすすめします。Next.jsのImageコンポーネントを活用しても、元画像が大きいとビルド時間やストレージに影響します。

3. **コンテンツの分割**: 非常に長い記事は、複数のファイルに分割し、シリーズとして公開する方が読者体験が向上します。また、ビルド時のメモリ使用量も抑えられます。

4. **リファクタリング戦略**: コードベースが成長するにつれて、マークダウン処理ロジックのリファクタリングが必要になることがあります。その際は、ユニットテストを充実させ、既存コンテンツに対するリグレッションテストを実施しましょう。

5. **エラーハンドリング**: 本番環境では予期せぬマークダウン構文に遭遇することがあります。堅牢なエラーハンドリングとフォールバック表示を実装することで、サイト全体がクラッシュする事態を防げます。

## まとめ

Next.jsとマークダウンを組み合わせたブログシステムは、開発者にとって多くの利点をもたらします：

1. **効率的なコンテンツ管理**: Git管理可能なプレーンテキストによるバージョン管理と差分確認の容易さ
2. **パフォーマンス**: 静的サイト生成とISRによる表示速度と更新頻度のバランス
3. **拡張性**: remarkプラグインエコシステムによる機能拡張の柔軟性
4. **型安全性**: TypeScriptとの統合による開発時のエラー検出
5. **開発体験**: マークダウンによる執筆の手軽さと、プログラマティックな制御の両立

「一長一短」の視点からは、以下の点も考慮すべきです：

### あわせて読みたい

他にも技術ブログをあげているのでそちらもよろしければ見ていってください。

[私のブログ記事一覧](https://daikimatsuura.vercel.app/blog)

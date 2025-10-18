# Vue＋Nuxt経験者が「Next.js Foundations course App Router」を終えての雑感
## 前置き
* 雑感：まとまりのない思いついたままの感想。 とりとめのない感想。
* Vue + Nuxt 経験
    * ４年ほど。SPA中心。
    * Vue3への破壊的アップデートの被害経験あり。

## 概要
* Next.js 公式　Foundations courseの [App Router](https://nextjs.org/learn/dashboard-app) を一通り終えたので、その感想を纏めた。
* 公式 Foundation courseのApp Routerでは、全十七章構成で以下のアプリケーションを開発する。
    * A public home page.
    * A login page.
    * Dashboard pages that are protected by authentication.
    * The ability for users to add, edit, and delete invoices.

##  開発環境
* node.js環境
    * vscodeのDev Container(node.js)環境
        ```
        node ➜ /workspaces/next_app/nextjs-dashboard (main) $ node --version
        v22.16.0
        ```

* NextJS環境（15.4.6）
    * ビルド環境は指定された手順通り公式CLIを使用。
        * Viteは現時点で公式未対応
        * 内部ビルドエンジンは既定でTurbopackが使われるが訳ありで外すことに。（後述）

   ```
    node ➜ /workspaces/next_app $ npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm
    Need to install the following packages:
    create-next-app@15.4.6
    Ok to proceed? (y) y
    ・・・ snip ・・・
    Success! Created nextjs-dashboard at /workspaces/next_app/nextjs-dashboard
    ・・・ snip ・・・
    cd nextjs-dashboard
    pnpm run dev
    ```

##　Next.jsに感じた戸惑いや懸念
### JSX内のコメントでHydration Error発生
    ```
    In HTML, whitespace text nodes cannot be a child of <html>. Make sure you don't have any extra whitespace between tags on each line of your source code. This will cause a hydration error.
    ```
* JSX内のコメントアウト記述が上記エラーの原因であった。
* Next.jsのApp Routerや、React Server Componentsや、strict hydratonでは、html,head,body要素直後のJSXコメント内の改行、空白もtextノードとして出力されるためhydration mismatchが発生する。
* 上記差分は、Nuxt/Vueでは発生しない。Vueのテンプレート内コメントはSSR時に削除されるため。（ただし、Nuxt/Vueでも状態差分、非同期差分によりHydration Errorが発生するエラー実装はある。）
* Nextの実運用上、JSX内のコメントは避けた方が無難。（return外部でコメントするなどの回避方法はあるが・・・）


### client apiとserver apiの使い分け
* Server ComponentやServer Actionのおかげで、クライアントと同レベルにサーバサイド実装ができるとは言っても、apiは明確に使い分ける必要がある。シームレスに実装できるわけではない。個人的には逆に混乱する。
    * SPA的なページ遷移と、サーバ側によるページ遷移の例
        * client apiによるページ遷移API
            ```
            const { replace } = useRouter();
            ```
        * server apiによるページ遷移API
            ```
            import { redirect } from 'next/navigation';
            ```
    * クライアントのみで実行可能なAPI
        * DOMやURLを直接操作するもの
        * React Hooks(useXXX)
        * 例
            * useState, useEffect, useRouter（next/navigation）
            * usePathname, useSearchParams
            * window, document を触る処理
    * サーバのみで実行可能なAPI
        * DB, FS, 外部 API との通信、キャッシュ操作など
        * 例
            * redirect, notFound（next/navigation）
            * revalidatePath, revalidateTag
            * 直接fetchを呼ぶ（SSR/SSGの場合）

### use client/use serverの宣言文の意味の非対称性
* use client/use serverはその意味が対称では無いため混乱する。!(use client) == (use server)ではない
    * use client
        * このファイル内のコンポーネントの実行環境がクライアントであることを定義
        * App Router環境ではデフォルトはサーバコンポーネントであるため、クライアントでの実行を明示的にするためには、この宣言が必要。
    * use server
        * そのファイルでexportされた関数がサーバアクションであることの宣言。このファイル内のコンポーネントがサーバコンポーネントという意味ではない。
        * フォームやボタンのaction={fn} から呼び出されるサーバ側RPMエンドポイントであることの宣言。

* GitHub/Discussionsでも同様の混乱が話題になっています。[Why do we use 'use server' in the server action component? #58752](https://github.com/vercel/next.js/discussions/58752)

* インラインサーバアクションを利用するソースコードにおいては、use client/use serverが同居してしまう！！！
    * 果たしてこれは便利なのか？
    * stack overflowで「サーバアクションの利点がわからん」というトピック (https://stackoverflow.com/questions/77883318/trouble-understanding-the-benefit-point-of-server-actions-in-next-js)

    ```
    'use client'

    export default function InlineActionForm() {
    async function action(formData: FormData) {
        'use server'
        console.log('Form submitted on server:', formData.get('name'))
    }

    return (
        <form action={action}>
        <input name="name" placeholder="Your name" />
        <button type="submit">Send</button>
        </form>
    )
    }
    ```

### サーバコンポーネント内でのSQL実行への違和感
* DBMSへの依存実装をコンポーネントレイヤに持ち込むことが果たして便利なのか？ 下記サンプルは１例であるが、ダッシュボードコンポーネントレイヤ内にSQL文がある。
    * 「use clientが無い == サーバーコンポーネントである」から、DBアクセスしても問題ない。「やった。便利！！」となりますか？
    * 私には、SQLが複雑化して、DBMS依存の実装がコンポーネントレイヤにまで発展、DBMSリプレース段階で後悔する未来しか見えないです。

    ```
    // app/dashboard/page.tsx
    import { sql } from '@vercel/postgres';

    export default async function DashboardPage() {
    const result = await sql`SELECT * FROM invoices LIMIT 5`;
    return (
        <ul>
        {result.rows.map((row) => (
            <li key={row.id}>{row.customer_name} - {row.amount}</li>
        ))}
        </ul>
    );
    }
    ```

* 本チュートリアルでもPostgreSQL依存コードが、app/lib/data.tsに多数あるが、DBMSを変更する場合のコード改変を鑑みると、API層で分離できていた時の方が可搬性が高いのでは？　DBMS依存コードの分離については、「リポジトリパターン」で抽象化という方法があるらしいが。
* 実際のDBアクセスでは、複数テーブルの結合、複合条件など、ビジネスロジックが多数入り込むことになり、これらのためにサービス層を設け、サーバコンポーネントから分離することになる。
* 結果、可搬性を鑑みると「リポジトリパターン＋サービス層」という設計が必須。
* サービス層が複雑（データ結合、ソートなど）になると、シングルスレッドであるNode.jsでのパフォーマンスへの懸念が生じる。
* 外部APIサービスとの連携が最初から必須なシステムにおいて、API層を意識したシステム設計の方に分があるように感じる。

### errorページのReact Error Boundaryへの依存
* Next.jsはerror.tsxがエラーページとしてハンドリングされる仕組みを提供。
* error.tsx自体はReact Error Boundaryへのfallback UIであり、そのため、client componentでなければならないが、そこへの連想は初心者には掴みづらい。


## Middleware ランタイムの制限
* 前述した開発環境で「Turbopack」を外した理由
* 本チュートリアルをずっとdevモードで進めていたが、buildを試したところ以下のエラーに遭遇。ビルドは成功しているように見えるが、アプリケーション起動後のアクセスが空白ページが応答される。
    ```
    > @ build /workspaces/next_app/nextjs-dashboard 
    > next build 
    ▲ Next.js 15.3.2 
    - Environments: .env 
    Creating an optimized production build ... 
    ⚠ nodejs runtime support for middleware requires experimental.nodeMiddleware be enabled in your next.config 
    ⚠ nodejs runtime support for middleware requires experimental.nodeMiddleware be enabled in your next.config 
    ⚠ nodejs runtime support for middleware requires experimental.nodeMiddleware be enabled in your next.config 
    ✓ Compiled successfully in 5.0s
    ```
* 上記エラーメッセージの意味
    * middlewareにてnodejsランタイムを利用するためには、next.config.js に experimental.nodeMiddleware: true を追加しなさい。
    * middlewareにてnodejsランタイムを利用する理由は、bcrypt、postgresを使うため。これらはまだEdgeランタイムでは扱えない。

* ２つのランタイム
    * Node.jsランタイム（既定）
    * Edgeラインタイム　（軽量、高速）
        * Edgeランタイムの制限
            *  https://nextjs.org/docs/app/api-reference/edge?utm_source=chatgpt.com

            ```
            The Edge Runtime does not support all Node.js APIs. Some packages may not work as expected.
            ```

            * fetch, Request, Response などの Web 標準 API をサポートしていますが、 fs, http, cryptoなどのNode.jsネイティブAPIは使えない。
            　（https://www.gaji.jp/blog/2025/03/28/22690/）

* middlewareの制限
    * Next v15.2以前：Edgeランタイムのみ対応。
    * Next v15.2以降：Edge/Node.js ランタイムの選択可
    * middlewareのランタイム指定（def.Edge） (https://nextjs.org/docs/app/api-reference/file-conventions/middleware#runtime)

* 仕方なく、next.configにxperimental.nodeMiddleware: trueを指定すると、今度はbuildエラーが発生
    ```
    The experimental feature "experimental.nodeMiddleware" can only be enabled when using the latest canary version of Next.js.
    ```
    * どうやら、Next.js 15.3.2 (stable) では experimental.nodeMiddleware がまだ入っていない。canary ビルド（next@canary）でのみ有効化できる
        * [公式ドキュメントに記載](https://nextjs.org/docs/app/api-reference/file-conventions/middleware#runtime)されている設定であるにも関わらず


* NextAuth.js v5 (beta)の導入で実行時エラー発生

    ```
    Uncaught Error: Cannot find the middleware module
    DevServer.runMiddleware ...
    ```

    * Next.js 15 + Turbopackで、middlewareビルドにおいて、Edge用の制約が適用され、Node runtime設定が無視されてしまうバグがあるらしい。  
    [Node.js runtime support for Next.js Middleware #71727](https://github.com/vercel/next.js/discussions/71727)

## 雑感まとめ
* Server Action, Server Componentによる、frontend/backendのシームレスな開発について、私は馴染めそうにない。
    * そもそも、frontend/backendの分離・分担は開発体制や、カバーする技術領域分担の面から理にかなっている。
    * 複雑なビジネスロジックやDBMS依存コードがNextJS上に実装されることの弊害が大きいのでは？
* Middleware周りの、ランタイム整合については不完全で、ドキュメントも不親切と感じた。
* Vercelへのロックイン傾向は今後強まる懸念あり。
* OSS的分散貢献モデルを実践している[void(0)](https://voidzero.dev/posts/announcing-voidzero-inc) にも期待したいが。
* とはいえ、最低限のキャッチアップはしていこう。
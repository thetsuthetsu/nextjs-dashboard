# Learn Next.js

* 参考：　https://nextjs.org/learn

## Getting Started
### Creating a new project

* pnpm (it's faster and more efficient than npm or yarn)

    ```
    node ➜ /workspaces/next_app $ npm install -g pnpm

    changed 1 package in 3s

    1 package is looking for funding
    run `npm fund` for details
    ```
    
    ```
    node ➜ /workspaces/next_app $ pnpm -v
    10.14.0
    ```

* create-next-app (nextjs-dashboard)

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

* install packagea
    ```
    node ➜ /workspaces/next_app/nextjs-dashboard (main) $ pnpm i
    Lockfile is up to date, resolution step is skipped
    Already up to date
    Done in 385ms using pnpm v10.14.0
    ```

* run dev
    ```
    node ➜ /workspaces/next_app/nextjs-dashboard (main) $ pnpm dev

    > @ dev /workspaces/next_app/nextjs-dashboard
    > next dev --turbopack

    ▲ Next.js 15.3.2 (Turbopack)
    - Local:        http://localhost:3000
    - Network:      http://172.18.0.2:3000

    ✓ Starting...
    ```

### CSS Styling
* Tailwind
    * CSSスタイルはグローバルに共有されますが、各クラスは各要素に個別に適用
* CSS Modules
    * 一意のクラス名を自動的に作成して CSS をコンポーネントに適用できるため、スタイルの衝突についても心配する必要がありません。
* clsxライブラリ　(https://www.npmjs.com/package/clsx)
    * 動的条件によって適用するクラスを変更する。

### Optimizing Fonts and Images    
*　フォントの最適化
    * [Cumulative Layout Shift](https://vercel.com/blog/how-core-web-vitals-affect-seo) により、ページロード後に、システムフォント（またはフォールバックフォント）からカスタムフォントに切り替わることによる表示上のずれが発生する。
    * next/fontにより、ビルド時にフォントのダウンロード、ホストが最適化され（他の静的アセットと一緒にホストされる）、パフォーマンスの影響を取り除く。

    * app/ui/fonts.ts
        * primaryフォントの指定
    * app/layout.tsx
        * primaryフォントの利用
    * app/ui/fonts.ts
        * secondaryフォントの指定
    * app/page.tsx
        * secondaryフォントの利用
        * <Acme />のコメントアウトを外す。（secondaryフォントを使用しているため）

* 画像の最適化  
    * 通常の手動設定
        ```
        <img
            src="/hero.png"
            alt="Screenshots of the dashboard project showing desktop version"
            />
        ```
        * 以下は手動で設定する必要がある。  
            * (responsive対応) 画像がさまざまな画面サイズに応答することを確認します。
            * さまざまなデバイスの画像サイズを指定します。
            * 画像の読み込み時にレイアウトがシフトするのを防ぎます。
            * ユーザーのビューポート外にある画像を遅延読み込みします。    

    * <Image> コンポーネント   
        * 以下を自動で最適化する
            * 画像の読み込み時にレイアウトシフトが自動的に発生するのを防ぎます。
            * 大きな画像がビューポートの小さいデバイスに送信されないように、画像のサイズを変更します。
            * デフォルトでは画像を遅延読み込みします（画像はビューポートに入ると読み込まれます）。
            * ブラウザがサポートしている場合、WebP, AVIFなどのモダンフォーマットに対応します。
        * hero-desktop.png/hero-modile.pngをデバイスによって切り替える
            * app/page.tsx
                * <Image>コンポーネントのclassName(hidden/block md:block/md:hidden)にて切り替え可能
            * DevToolの「Toggle device toolbar」にて切り替え可能

### Creating Layouts and Pages
* Nested routing
* layout.tsx/page.tsxによるUI分離
* フォルダ改装によるルーティング
    ```
    app/dashboard
                 /layout.tsx (<SideNav>)
                 /page.tsx
    app/dashboard/customers
                           /page.tsx
    app/dashboard/invoices
                          /page.tsx
* ルートレイアウト(app/layout.tsx)からの子ページへの共有
    * フォント、メタデータ

### Navigating Between Pages
* NavLink (nextjs-dashboard/app/ui/dashboard/nav-links.tsx)
    * next/linkの利用
        * <a> -> <Link>
        * 自動コード分割
        * Linkページのプリフェッチ
    * usePathname(Reactフック)の利用
        * clientコンポーネントへの切り替えが必要　(use client)
    * clsxの利用
        * カレントパスの状態からリンク表示を動的変更

### Setting Up Your Database
* vercelアカウント：既存
* Storage Provider: Neon
* .env.localをシークレット解除し、プロジェクトの.envにコピー
    * .gitignoreに含めるように注意
* localhost:3000/seedにアクセスし、以下の正常応答を待つ
    ```
    {"message":"Database seeded successfully"}
    ```

* シーディング確認
    * nextjs-dashboard/app/seed/route.tsを編集
    * http://localhost:3000/queryにアクセスし、以下の応答を待つ

        ```
        [{"amount":666,"name":"Evil Rabbit"}]
        ```

### Fetching Data
* APIs
* React Server Components (default)
* SQL
    * app/lib/data.ts

* Request Waterfalls
    ```
    const revenue = await fetchRevenue();
    const latestInvoices = await fetchLatestInvoices();
    const {
        numberOfInvoices,
        numberOfCustomers,
        totalPaidInvoices,
        totalPendingInvoices,
    } = await fetchCardData();
    ```

### Static and Dynamic Rendering
* static rendering
    * データの取得とレンダリングはビルド時 (デプロイ時) またはデータの再検証時にサーバー上で行われます
    * パフォーマンス、SEOで有利
* dynamic redering
    * コンテンツはリクエスト時（ユーザーがページにアクセスした時）に各ユーザーのサーバー上でレンダリングされます
    * リアルタイムデータ、ユーザ固有データの表示
* 現時点でdashboardはdynamic rendering
    * アプリケーションの速度は、最も遅いデータ フェッチと同じになります。

## Streaming
* チャンク：段階的ストリーミングを行うのコンテンツブロックの単位
* ストリーミングの２方法
    1. pageレベルでloading.tsxファイルを利用
    2. componentレベルで、より細かくReact Suspense（<Suspense>）を定義
* スケルトン
    * nextjs-dashboard/app/ui/skeletons.tsx
    * loading.tsxから静的にロードされる。
* root group
    * /dashboard/loading.tsxは、子フォルダ invoices/page.tsx, customers/page.tsxにも影響を与える。
    * /dashboard/(overview)フォルダを作成し、loading.tsx, page.tsxを移動する。

## Pertial PreRendering (PPR)
* experimental feature（https://nextjs.org/docs/app/getting-started/partial-prerendering?utm_source=chatgpt.com）

## Adding Search and Pagination
* URL検索パラメータを使う利点
    * ブックマークおよび共有可能なURL 
    * サーバー側レンダリングが容易になる
    * 分析と追跡: URLに検索クエリとフィルターを直接含めることで、追加のクライアント側ロジックを必要とせずにユーザーの行動を追跡しやすくなります。
1. Capture the user's input.
    * /app/ui/search.tsx
    * use client
        * hookを利用：useSearchParams, usePathname, useRouter
        * Web APIを利用：URLSearchParams
2. Update the URL with the search params
    * usePathname + URLSearchParam(searchParams)
    * replace(usePathname + input query) 
        * ex: /dashboard/invoices?query=lee 
3. Keeping the URL and input in sync
4. Updateing the table

* Debouncing: 関数の実行頻度を制限するプログラミング手法
    * use-debounce

* Pagination
    * nextjs-dashboard/app/ui/invoices/pagination.tsx
        * Link hrefによるURL遷移でページ移動

## Mutating Data
### Server Actions
    * サーバー上で直接非同期コードを実行できます
    * Progressive Enhancement
        * forms work even if JavaScript has not yet loaded on the client. 
1. Create a new route and from
    * invoices/create/page.tsx
2. Create a Server Action
    * nextjs-dashboard/app/lib/actions.ts
        * use server
            * you mark all the exported functions within the file as Server Actions. 
            * These server functions can then be imported and used in Client and Server components. 
            * Any functions included in this file that are not used will be automatically removed from the final application bundle.
        * You can also write Server Actions directly inside Server Components by adding "use server" inside the action. We recommend having a separate file for your actions.
    * nextjs-dashboard/app/ui/invoices/create-form.tsx
        * form action={createInvoice}
        * 通常、HTMLではfrom属性URLによりAPIエンドポイントを渡すが、サーバーアクションはバックグラウンドでPOSTAPIエンドポイントを作成します。そのため、サーバーアクションを使用する際にAPIエンドポイントを手動で作成する必要はありません。
3. Extract the data from formData
    * nextjs-dashboard/app/ui/invoices/create-form.tsx
        * form.get("form-key")
4. Validate and prepare the data
    * [data definitions] nextjs-dashboard/app/lib/definitions.ts
    * TypeScriptによる型検証
        * Zod (https://zod.dev/)
5. Inserting the data into your database
    * createInvoice(actions.ts)からsql実行
6. Revalidate and redirect
    * client-side router cache
        * ルートセグメントをユーザーのブラウザに一定期間保存するクライアント側ルーターキャッシュがあります。
        * revalidatePath: invlicesルートで、表示データを更新するためキャッシュをクリアし、新規リクエストがサーバに要求されるようにする。
    * redirect
        * invoices登録後、/dashboard/invoicesに強制遷移する。
            * clientコンポーネント：　replace(useRouter)
            * serverアクション、serverコンポーネント：　redirect

### Updating a invoice
1. Create a new dynamic route segment with the invoice id.
    * Dynamic Route Segments
        * invoices/[id]/edit/page.tsx
        * URL: /dashboard/invoices/${id}/edit
    * UpdateInvoice (nextjs-dashboard/app/ui/invoices/buttons.tsx)
2. Read the invoice id from page parama
    * nextjs-dashboard/app/dashboard/invoices/[id]/edit/page.tsx
3. Fetch the specific invoice
    * URL include UUID
        * ex: http://localhost:3000/dashboard/invoices/eda5555d-6bfb-4242-bac8-746e6a6c2095/edit

4. Pass the id to the Server Action
    * Using JS bind to pass id
        ```
        const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
        return <form action={updateInvoiceWithId}>
        ```

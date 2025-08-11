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
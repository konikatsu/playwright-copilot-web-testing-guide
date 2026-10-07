# プロキシ経由でCopilotにPlaywrightの準備とテスト作成を任せる手順

- Webアプリの一覧画面で、先頭の項目にあるリンクをクリックするテストを作成します。
- 利用者が行う操作を、最初の導入・サインイン・設定値の入力と、必要な承認に絞っています。
- Node.js、npm、プロキシ、Playwright、MCPの準備は、Copilotへの指示文でまとめて依頼します。
- Windows 11以降を想定しています。導入済みの工程は省いてください。
- プロキシの記入例は `http://proxy.example.com:8080` です。実際のホスト名とポート番号に置き換えてください。この例は認証不要のHTTPプロキシ用です。
- 更新日：2026年10月7日。変更内容は[修正履歴](CHANGELOG.md)に記載しています。

## 1 VS Codeをインストールする

- 組織のプロキシ設定を使うブラウザで、[VS Codeのダウンロードページ](https://code.visualstudio.com/download)を開きます。
- ブラウザが接続できない場合は、Windowsの **設定 → ネットワークとインターネット → プロキシ** で、担当者から案内された設定を確認します。管理済みの設定があれば、その設定を使います。
- Windows向けの **User Installer** を選びます。通常のIntel・AMD搭載PCはx64、Arm搭載PCはArm64を選びます。
- ダウンロードしたインストーラーを開き、画面の案内に従ってインストールし、VS Codeを起動します。
- 証明書エラーが表示される場合は、組織から配布されたCA証明書のWindowsへの導入について、担当者の案内に従います。
- 詳細は[Windows版の導入手順](https://code.visualstudio.com/docs/setup/windows)と[VS Codeのネットワーク設定](https://code.visualstudio.com/docs/setup/network)を参照してください。

## 2 Copilotへ指示できる状態にする

- `Ctrl + Shift + P` で **Preferences: Open User Settings (JSON)** を開きます。
- 既存の設定を残したまま、次の項目を追加・変更します。プロキシURLは実際の値に置き換えます。

  ```json
  {
    "http.proxy": "http://proxy.example.com:8080",
    "http.proxyStrictSSL": true
  }
  ```

- VS Codeを開き直し、Copilotアイコンから **Use AI Features** または **Sign in to use Copilot** を選び、GitHubアカウントでサインインします。すでに利用できる場合は、この操作を省きます。
- Copilotが表示されない場合は、`Ctrl + Shift + X` で拡張機能画面を開き、GitHub発行の **GitHub Copilot** を確認・インストールします。
- エクスプローラーで任意の作業場所に `web-test` フォルダーを作り、VS Codeの **ファイル → フォルダーを開く** で開きます。すでにテスト用フォルダーがあれば、それを使います。
- 自分の作業フォルダーの信頼確認が表示されたら、内容を確認して信頼する操作を選びます。
- `Ctrl + Alt + I` でCopilotのチャットを開き、コマンドの実行とファイルの編集ができる **Agent** を選びます。Session Targetが表示される場合は、ローカルPCで作業する **Local** または **Copilot** のセッションを使います。
- 詳細は[Copilotの導入手順](https://code.visualstudio.com/docs/setup/copilot)と[プロキシ設定](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/set-up-copilot/configure-network-settings?tool=vscode)を参照してください。

## 3 環境の準備をCopilotへまとめて依頼する

- 次の指示文の `［ ］` を自分の環境に合わせて記入し、Copilotへ送信します。
- 固定のプロキシURLはネットワーク担当者へ確認します。PACのURLをnpmや `HTTP_PROXY` の設定値として使うことはできません。
- CA証明書が必要な場合は、担当者から受け取ったPEM形式のファイルをローカルに保存し、そのパスを記入します。不要なら「なし」と記入します。
- npmのプロキシ設定を行っても `SELF_SIGNED_CERT_IN_CHAIN` や `UNABLE_TO_GET_ISSUER_CERT_LOCALLY` などが出る場合は、HTTPSを中継するプロキシのCA証明書をNode.jsが信頼できていない可能性があります。Copilotにエラーを確認させ、必要なCA証明書を担当者へ確認します。
- パスワードは指示文に含めません。認証が必要な場合は、その方式の確認と利用者による認証を依頼します。

  ```text
  このWindows PCと、VS Codeで開いている作業フォルダーに、
  Playwrightのテスト作成環境を準備してください。
  コマンドの説明だけで終わらず、実行できる操作はターミナルとファイル編集で進めてください。
  利用者の操作が必要な場合だけ、具体的な操作を案内してください。

  プロキシURL：［http://proxy.example.com:8080を実際の値に変更］
  プロキシ認証：［不要／必要。必要なら認証方式を記入］
  環境変数NO_PROXYの除外先：［localhost,127.0.0.1など］
  ブラウザのプロキシ除外先：［localhost,127.0.0.1など］
  社内CA証明書のPEMファイル：［ファイルパス／なし］

  - 既存のNode.js、npm、Playwright、MCP設定を確認し、使えるものを再利用してください。
  - 接続エラーが出る場合は、認証エラー、証明書エラー、接続タイムアウトを切り分けてください。
  - npmのプロキシ設定後も証明書エラーが出る場合は、必要な組織CAの設定を確認してください。
  - CAが必要なのに未提供の場合は、担当者から受け取るファイルと必要な情報だけを案内してください。
  - Node.jsはPlaywrightの現行要件に合うLTS版を選び、可能なら24系を使ってください。
  - Node.jsが未導入なら、WinGetの有無と winget --info の管理者設定を確認してください。
  - WinGetのProxyCommandLineOptionsが有効なら、--proxy で指定されたプロキシを明示してください。
  - WinGetを使う場合は、OpenJS.NodeJS.LTS を --exact --source winget --installer-type wix で導入してください。
  - プロキシ指定が無効なら、既存のWindowsプロキシ設定で取得できるか確認してください。
  - 管理者設定の変更や手動導入が必要な場合だけ、必要な操作を案内してください。
  - インストール直後にPATHが反映されていない場合は、導入先を確認して実行中のプロセスへ反映してください。
  - HTTP_PROXY、HTTPS_PROXY、NO_PROXYをユーザー環境変数と、以降のコマンドを実行するプロセスへ設定してください。
  - HTTPS_PROXYの値も、指定されたHTTPプロキシURLを使ってください。
  - CA証明書が指定されている場合だけ、NODE_EXTRA_CA_CERTSを設定してください。
  - npmのproxyとhttps-proxyをユーザー設定に保存し、strict-sslはtrueにしてください。
  - CA証明書が必要なら、npmのcafileもユーザー設定へ追加してください。
  - 証明書の検証を無効にする設定は使わず、組織から配布されたCAを使ってください。
  - npm.cmd ping でレジストリへの接続を確認してください。
  - package.jsonが未作成なら npm.cmd init -y で作成してください。
  - Playwrightが未導入なら npm.cmd install -D @playwright/test で導入してください。
  - TypeScriptのplaywright.config.tsとtestsフォルダーを用意してください。
  - 新規構成ではtestDirをtestsにし、Chromiumのプロジェクトを用意してください。
  - プロキシと必要なCAの設定を適用して npx.cmd playwright install chromium を実行してください。
  - ブラウザ取得がタイムアウトする場合はPLAYWRIGHT_DOWNLOAD_CONNECTION_TIMEOUTを120000にして再試行してください。
  - 作業フォルダー直下の .mcp.json に、mcpServers形式でPlaywright MCPを設定してください。
  - MCPのパッケージは @playwright/mcp@latest を使い、同じサーバーを重複登録しないでください。
  - MCPが操作するブラウザは、導入済みのMicrosoft Edgeを --browser msedge で指定してください。
  - MCPのブラウザ通信には --proxy-server と --proxy-bypass を設定してください。
  - MCPの起動プロセスにもenvでプロキシと必要なCAを明示してください。
  - npmやブラウザ取得用の設定と、MCPのブラウザ通信の設定をそれぞれ確認してください。
  - .gitignoreにローカル設定、認証状態、.env、.npmrc、.mcp.json、証明書を除外する設定を追加してください。
  - プロキシ認証が必要なら、方式に対応するローカル設定を用意し、必要な認証操作を案内してください。
  - 準備が終わったら、Node.jsとnpmのバージョン、作成したファイル、接続確認の結果を日本語で報告してください。
  - VS Codeの再起動やMCPの起動が必要なら、そのタイミングと操作をまとめて案内してください。
  ```

- Copilotは、この指示を受けて導入・設定・ファイル作成を進めます。PCの権限やプロキシの認証方式によっては、手順4の利用者操作が必要です。
- WinGetのプロキシ指定は、PCの管理者設定で制御されます。詳細は[MicrosoftのWinGet導入説明](https://learn.microsoft.com/en-us/windows/package-manager/winget/)と[インストールコマンド](https://learn.microsoft.com/en-us/windows/package-manager/winget/install)を参照してください。
- npm・Playwright・MCPの設定は[npmの設定一覧](https://docs.npmjs.com/cli/v11/using-npm/config/)、[Playwrightの設定](https://playwright.dev/docs/test-configuration)、[プロキシ経由のブラウザ取得](https://playwright.dev/docs/browsers#install-behind-a-firewall-or-a-proxy)、[MCPの設定](https://playwright.dev/mcp/configuration/options)を参照してください。

## 4 必要な確認と承認だけ操作する

- コマンド実行やファイル編集の確認が表示されたら、実行内容を確認して許可します。
- Windowsの管理者権限確認、利用規約への同意、プロキシ認証は、利用者または管理者が対応します。
- 組織CAが必要なのに用意されていない場合は、担当者から指定された証明書を受け取ります。Copilotは、組織で信頼する証明書を判断して代わりに発行することはできません。
- Copilotから再起動を案内された場合は、VS Codeをすべて閉じ、同じ作業フォルダーで開き直します。再起動後、次の短い指示を送れば準備を続けられます。

  ```text
  セットアップの続きを進めてください。
  既存の設定を使ってNode.js、npm、Playwright、Playwright MCPの準備状況を確認し、
  完了していない作業だけ実行してください。
  利用者の操作が必要な場合だけ、具体的に案内してください。
  ```

- MCPの起動を利用者に求められた場合だけ、`Ctrl + Shift + P` で **MCP: List Servers** を開き、`playwright` の **Start** または再起動を選びます。
- Playwrightのツールが使えない場合だけ、**Configure Tools** または **Chat: Open Customizations → Tools** で有効化します。
- Copilotから環境の準備完了が報告されたら、次の指示へ進みます。

## 5 Copilotにテストスクリプトの作成を指示する

- 一覧画面のURL、対象となる一覧、クリックするリンク、クリック後の成功条件を確認します。
- 次の指示文の `［ ］` を記入し、Copilotへ送信します。実際のURLや画面情報は、自分の作業環境だけで入力します。

  ```text
  PlaywrightとTypeScriptで、次のテストスクリプトを作成してください。
  今回はテストスクリプトの作成まで行ってください。

  一覧画面URL：［一覧画面のURL］
  対象の一覧：［一覧の名前や位置］
  表示条件：［検索条件や並べ替え条件。なければ「初期表示」］
  クリックするリンク：［先頭項目の商品名リンク、詳細リンクなど］
  クリック後の成功条件：［表示される見出しや項目番号など］
  ログインの必要性：［必要／不要］
  対象画面への通信経路：［プロキシ経由／社内サイトへ直接接続］

  - セットアップ済みのPlaywright設定、環境変数、ローカル設定を使ってください。
  - テストの通信経路に合わせてuse.proxyを設定し、プロキシURLなどは環境変数から読み込んでください。
  - Playwright MCPで対象画面を開き、実際の画面構造を確認してください。
  - 対象の一覧が表示されるまで待つ処理を入れてください。
  - 対象の一覧の中で、表示順の先頭にある項目の指定されたリンクを選んでください。
  - メニュー、一覧の見出し、ページ送りのリンクを対象に含めないでください。
  - 現在表示されている項目名やリンクの文字列は固定しないでください。
  - 実際の画面で確認できたロケーターを使ってください。
  - クリック後の成功条件をexpectで検証する処理を入れてください。
  - 一覧が0件の場合は、理由が分かる形でテストを失敗させてください。
  - 認証が必要なら、必要な設定を準備し、私が行うログイン操作だけ案内してください。
  - MCPのログイン状態やプロキシ設定はテストへ自動で引き継がれないため、テスト側にも必要な設定を用意してください。
  - パスワード、認証状態、社内CA証明書はローカルで管理し、公開用ファイルへ含めないでください。
  - 確認できない画面や不足する情報がある場合だけ、質問してください。
  - tests/first-item.spec.ts に保存してください。
  - 作成したファイルと各処理を日本語で説明してください。
  ```

- ログインが必要な場合は、自分でログイン画面へ入力します。パスワードをチャットへ記入する必要はありません。
- Copilotの報告で `tests/first-item.spec.ts` の作成を確認します。
- 動作を変えたい場合は、「検索結果の先頭行の商品名リンクをクリックするように修正してください」のように、対象と期待する動作を具体的に伝えます。

# プロキシ経由でVS CodeとGitHub Copilotを使いPlaywrightテストを作成する手順

- Webアプリの一覧画面で、先頭の項目にあるリンクをクリックするテストを作成します。
- VS Codeのインストールから、Copilotにテストスクリプトの作成を依頼するところまでを説明します。
- Windows 11以降を想定しています。すでに導入済みのものは、動作を確認して次へ進んでください。
- プロキシの記入例は `http://proxy.example.com:8080` です。`proxy.example.com` と `8080` は、ネットワーク担当者から案内されたホスト名とポート番号に置き換えてください。
- 記入例は認証不要のHTTPプロキシを想定しています。認証が必要な場合は、各ツールが対応する認証方式をネットワーク担当者へ確認してください。
- 更新日：2026年10月7日。VS Codeのバージョンによって、画面の表示やボタンの位置が異なる場合があります。
- 変更内容は[修正履歴](CHANGELOG.md)に記載しています。

## 1 VS Codeをインストールする

- Windowsの **設定 → ネットワークとインターネット → プロキシ** を開き、組織の設定を確認します。すでに管理されている設定があれば、その設定を使います。
- 手動設定が必要な場合は、案内されたホスト名とポート番号を設定します。記入例は、アドレスが `proxy.example.com`、ポートが `8080` です。
- 自動構成スクリプトが指定されている場合は、そのPACのURLを設定します。PACのURLを、この後の `HTTP_PROXY` やnpmの設定値として使うことはできません。固定のプロキシURLを別途確認してください。
- Edgeなどのブラウザから、プロキシ経由で[VS Codeのダウンロードページ](https://code.visualstudio.com/download)を開きます。
- Windows向けの **User Installer** をダウンロードします。通常のIntel・AMD搭載PCはx64を選びます。Arm搭載PCはArm64を選びます。
- ダウンロードしたインストーラーを開き、画面の案内に従ってインストールします。
- インストールが終わったら、VS Codeを起動します。
- 起動できれば、この手順は完了です。
- 証明書エラーが表示される場合は、組織から配布されたCA証明書がWindowsの信頼ストアに導入されているか確認します。必要な導入は、ネットワーク担当者の案内に従ってください。
- 詳しい導入方法は[Windows版の公式手順](https://code.visualstudio.com/docs/setup/windows)、ネットワーク設定は[VS Codeの公式説明](https://code.visualstudio.com/docs/setup/network)を参照してください。

## 2 GitHub Copilotを使えるようにする

- `Ctrl + Shift + P` で **Preferences: Open User Settings (JSON)** を開きます。プロジェクトの設定ではなく、自分のユーザー設定を編集します。
- 既存の設定を残したまま、次の項目を追加・変更します。プロキシURLは実際の値に置き換えてください。

  ```json
  {
    "http.proxy": "http://proxy.example.com:8080",
    "http.proxyStrictSSL": true
  }
  ```

- VS Codeを開き直します。VS Code本体はWindowsのプロキシ設定を使い、Copilotや拡張機能には上記の設定を適用します。npmやPlaywrightには、後の手順で別の設定を行います。
- GitHubアカウントを用意します。アカウントがない場合は[GitHub](https://github.com/)で作成します。
- VS CodeのCopilotアイコンから **Use AI Features** または **Sign in to use Copilot** を選びます。
- 表示された案内に従って、Copilotを利用するGitHubアカウントでサインインします。
- Copilotが利用できない場合は、拡張機能画面を `Ctrl + Shift + X` で開き、発行元がGitHubの **GitHub Copilot** を確認・インストールします。
- チャット画面を `Ctrl + Alt + I` で開き、「日本語で回答してください」と入力します。
- 応答が返れば、Copilotを使う準備は完了です。利用できる機能や利用枠は、アカウントのプラン・組織の設定に従います。
- 接続できない場合は、プロキシ側でGitHub・Copilot・VS Code Marketplaceへの通信が許可されているか、ネットワーク担当者へ確認します。
- 認証付きプロキシでは、WindowsやブラウザでのログインがCopilotへ引き継がれるとは限りません。ユーザー名・パスワードを公開用の設定ファイルに記載しないでください。
- 詳しい設定方法は[Copilotの公式手順](https://code.visualstudio.com/docs/setup/copilot)、プロキシ設定は[Copilotのネットワーク設定](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/set-up-copilot/configure-network-settings?tool=vscode)を参照してください。

## 3 Node.jsをインストールする

- プロキシ設定済みのブラウザで[Node.jsのダウンロードページ](https://nodejs.org/en/download)を開き、Windows用のLTS版をインストールします。
- この手順ではNode.js 24系のLTS版を推奨します。Playwrightの現行要件は、最新の22系・24系・26系です。
- インストール後、VS Codeを閉じて開き直します。
- メニューの **ターミナル → 新しいターミナル** を選びます。
- 次のコマンドを1行ずつ入力し、Enterキーを押します。

  ```powershell
  node --version
  npm.cmd --version
  ```

- それぞれバージョン番号が表示されれば、この手順は完了です。
- `node` が見つからない場合は、Node.jsのインストールとVS Codeの再起動を確認します。
- Windowsの検索欄で「環境変数」を検索し、**環境変数を編集** を開きます。自分の **ユーザー環境変数** に、次の名前と値を追加・変更します。
- `HTTP_PROXY` の値：`http://proxy.example.com:8080`
- `HTTPS_PROXY` の値：`http://proxy.example.com:8080`
- `NO_PROXY` の値：`localhost,127.0.0.1`
- `HTTPS_PROXY` の名前でも、この例のURLは `http://` です。HTTPSの接続先にもHTTPプロキシを経由して接続します。Copilotは `https://` で始まるプロキシURLを現在サポートしていません。
- 社内サイトへの接続をプロキシから除外する場合は、担当者から指定されたホストやドメインを `NO_PROXY` に追加します。
- 社内CA証明書が必要な場合は、担当者からPEM形式のCA証明書を受け取り、ローカルのファイルとして保存します。例は `C:\certs\corp-proxy-ca.pem` です。
- その場合だけ、ユーザー環境変数 `NODE_EXTRA_CA_CERTS` にCA証明書のファイルパスを設定します。これはnpmなどのNode.jsの通信やPlaywrightのブラウザ取得で使用します。Windowsやブラウザの証明書設定は手順1の方法で別に確認します。
- 環境変数を保存したら、VS Codeをすべて閉じて開き直します。すでに起動しているNode.jsやMCPには、新しい設定が自動で反映されません。
- 新しいターミナルで、次のコマンドを実行します。プロキシURLは実際の値に置き換えてください。npmの設定は自分のユーザー設定に保存します。

  ```powershell
  npm.cmd config set proxy "http://proxy.example.com:8080" --location=user
  npm.cmd config set https-proxy "http://proxy.example.com:8080" --location=user
  npm.cmd config set strict-ssl true --location=user
  ```

- 社内CA証明書が必要な場合は、次の設定も行います。例のパスを、自分が保存したCA証明書のパスに置き換えてください。

  ```powershell
  npm.cmd config set cafile "C:\certs\corp-proxy-ca.pem" --location=user
  ```

- 設定が終わったら、次のコマンドでnpmレジストリへの接続を確認します。

  ```powershell
  npm.cmd ping
  ```

- `PONG` が返れば、設定されているnpmレジストリへ接続できています。
- `407 Proxy Authentication Required` が表示された場合は、プロキシ認証の設定を確認します。証明書エラーの場合は、CA証明書の形式と設定したファイルパスを確認します。
- 詳しい設定は[npmの設定一覧](https://docs.npmjs.com/cli/v11/using-npm/config/#https-proxy)と[Copilotのプロキシ・証明書の説明](https://docs.github.com/en/copilot/concepts/security-governance-and-network-settings/network-settings)を参照してください。
- 対応環境は[Playwrightのシステム要件](https://playwright.dev/docs/intro#system-requirements)を参照してください。

## 4 テスト用の作業フォルダーを作る

- エクスプローラーで、任意の作業場所に `web-test` という新しいフォルダーを作ります。
- VS Codeの **ファイル → フォルダーを開く** から、そのフォルダーを開きます。
- 自分で作成したフォルダーの信頼確認が表示されたら、内容を確認して信頼する操作を選びます。
- メニューの **ターミナル → 新しいターミナル** を選びます。
- 次のコマンドを入力します。

  ```powershell
  npm.cmd init playwright@latest
  ```

- パッケージのインストール確認が表示されたら、`y` を入力します。
- 言語は **TypeScript** を選びます。
- テストの保存先は **tests** を選びます。
- GitHub Actionsの追加は **No** を選びます。
- Playwright用ブラウザのインストールは **No** を選びます。次の操作で、まずChromiumだけをプロキシ経由で取得します。
- 左側のファイル一覧に `package.json`、`playwright.config.ts`、`tests` フォルダーが作成されたことを確認します。
- 同じターミナルで次のコマンドを実行します。`HTTPS_PROXY` がブラウザ取得用の通信に使われます。

  ```powershell
  npx.cmd playwright install chromium
  ```

- ダウンロードがタイムアウトする場合は、同じターミナルで待ち時間を120秒に延ばしてから再試行します。

  ```powershell
  $env:PLAYWRIGHT_DOWNLOAD_CONNECTION_TIMEOUT = "120000"
  npx.cmd playwright install chromium
  ```

- 証明書エラーの場合は、手順3の `NODE_EXTRA_CA_CERTS` と、設定後にVS Codeを開き直したかを確認します。
- 既存のテスト用フォルダーにこれらの設定がある場合は、その構成を使用します。初期化を重ねて行う必要はありません。
- 必要に応じて、拡張機能画面でMicrosoft発行の [Playwright Test for VSCode](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright) をインストールします。
- 詳しい設定方法は[Playwrightの導入手順](https://playwright.dev/docs/intro)、ブラウザ取得の設定は[プロキシ経由のインストール手順](https://playwright.dev/docs/browsers#install-behind-a-firewall-or-a-proxy)を参照してください。

## 5 Copilotとブラウザを連携する

- **Playwright MCP** を設定します。Copilotがブラウザを開き、実際の画面構造を確認するための連携機能です。
- VS Codeで開いている `web-test` フォルダーの直下に、`.mcp.json` というファイルを作ります。
- 次の内容を貼り付け、プロキシURLを実際の値へ置き換えて `Ctrl + S` で保存します。この例では、MCPが操作するブラウザに導入済みのMicrosoft Edgeを使います。

  ```json
  {
    "mcpServers": {
      "playwright": {
        "command": "npx",
        "args": [
          "-y",
          "@playwright/mcp@latest",
          "--browser",
          "msedge",
          "--proxy-server",
          "http://proxy.example.com:8080",
          "--proxy-bypass",
          "localhost,127.0.0.1"
        ]
      }
    }
  }
  ```

- すでにPlaywright MCPが登録されている場合は、その登録を使います。同じサーバーを重複して追加する必要はありません。
- `npx` によるMCPパッケージの取得には、手順3のnpm設定と環境変数を使います。`--proxy-server` は、MCPが操作するブラウザの通信に使う別の設定です。
- 社内サイトへ直接接続する必要がある場合は、担当者の指示に従って `--proxy-bypass` の値へ除外先を追加します。`NO_PROXY` だけでは、このブラウザの除外設定にはなりません。
- プロキシ認証が必要な場合は、認証方式に合うMCPのローカル設定を用意します。実際の認証情報を、この公開手順書やGitへ保存しないでください。
- `Ctrl + Shift + P` を押して、**MCP: List Servers** を実行します。
- `playwright` を選び、起動していない場合は **Start** を選びます。
- 設定を変更した場合は、サーバーを再起動します。
- サーバーの信頼確認が表示されたら、Microsoftの公式Playwright MCPであることを確認して許可します。
- チャットを開き、ファイルの編集ができる **Agent** を選びます。Session Targetが表示される場合は、ローカルPCで作業する **Local** または **Copilot** のセッションを使います。
- **Configure Tools** または **Chat: Open Customizations → Tools** で、Playwrightのツールが利用できることを確認します。
- ツール使用の確認が表示された場合は、実行する内容を確認して許可します。
- 起動に失敗したら、**MCP: List Servers → playwright → Show Output** でエラーを確認します。`npx` が見つからない場合は、Node.jsの導入とVS Codeの再起動を確認します。
- MCPは起動するのに対象画面へ接続できない場合は、プロキシURL、除外先、認証、Windows側の社内CA証明書を確認します。
- 登録形式の詳細は[VS CodeのMCP設定](https://code.visualstudio.com/docs/agent-customization/mcp-servers)、ブラウザの設定は[Playwright MCPの設定一覧](https://playwright.dev/mcp/configuration/options)、サーバーの詳細は[MicrosoftのPlaywright MCP](https://github.com/microsoft/playwright-mcp)を参照してください。

## 6 作成したいテストの情報を用意する

- テスト対象の一覧画面のURLを確認します。
- 対象となる一覧の名前・位置を確認します。例は「検索結果の一覧」「注文一覧」です。
- 先頭の項目に複数のリンクがある場合は、どのリンクを押すかを決めます。例は「商品名のリンク」「詳細リンク」です。
- 「先頭」は、一覧が表示されたときの並び順の先頭を意味します。必要な検索条件や並べ替え条件があれば、それも書きます。
- クリックした後に何が表示されれば成功かを決めます。例は「詳細画面の見出しが表示される」「クリックした項目の番号が詳細画面に表示される」です。
- ログインが必要かを確認します。必要な場合は、Copilotに認証方法の確認を依頼します。
- MCPのブラウザと、作成したPlaywrightテストのブラウザは別です。MCP側でログインしても、テスト側のログイン設定は別に必要です。
- パスワードは指示文に書かず、ログイン画面へ自分で入力します。ログイン状態をファイルへ保存する場合は、そのファイルを公開リポジトリに含めません。
- 実際のURLや画面情報は、自分の作業環境で指示文に記入します。
- テストでアクセスする画面もプロキシ経由か、社内ネットワークへ直接接続するかを確認します。MCPのプロキシ設定は、作成したテストには自動で引き継がれません。
- プロキシURL・除外先・認証の必要性を、テスト作成の指示に含めます。認証情報や社内CA証明書はローカル環境で管理します。

## 7 Copilotにテストスクリプトの作成を指示する

- `web-test` フォルダーを開いたまま、Copilotのチャットを開きます。
- 次の指示文の `［ ］` の部分を、手順6で確認した内容に置き換えます。
- 指示文全体をチャットへ貼り付けて送信します。

  ```text
  PlaywrightとTypeScriptで、次のテストスクリプトを作成してください。
  今回はテストスクリプトの作成まで行ってください。

  対象の一覧画面URL：［一覧画面のURL］
  対象の一覧：［一覧の名前や位置］
  一覧の表示条件：［検索条件や並べ替え条件。なければ「初期表示」］
  クリックするリンク：［先頭項目の商品名リンク、詳細リンクなど］
  クリック後の成功条件：［表示される見出しや項目番号など］
  ログインの必要性：［必要／不要］
  テストの通信経路：［プロキシ経由／社内サイトへ直接接続］
  テスト用プロキシURL：［http://proxy.example.com:8080を実際の値に変更。直接接続なら「なし」］
  プロキシの除外先：［localhost,127.0.0.1など。追加がなければ「なし」］
  プロキシ認証の必要性：［必要／不要］

  - 既存のPlaywright設定を確認してください。
  - 指定された通信経路に合わせて、テストのuse.proxy設定を用意してください。
  - プロキシURLなどは環境変数から読み込み、認証情報をコードへ直接記載しないでください。
  - プロキシや認証の設定が不足している場合は、必要な情報を質問してください。
  - Playwright MCPで対象画面を開き、実際の画面構造を確認してください。
  - 対象の一覧が表示されるまで待つ処理を入れてください。
  - 対象の一覧の中で、表示順の先頭にある項目のリンクを選んでください。
  - メニュー、一覧の見出し、ページ送りのリンクを対象に含めないでください。
  - 現在表示されている項目名やリンクの文字列は固定しないでください。
  - 実際の画面で確認できたロケーターを使ってください。
  - クリック後の成功条件をexpectで検証する処理を入れてください。
  - 一覧が0件の場合は、理由が分かる形でテストを失敗させてください。
  - 認証が必要な場合は、実行時の認証方法を確認してください。
  - 画面や認証方法を確認できない場合は、必要な情報を質問してください。
  - tests/first-item.spec.ts に保存してください。
  - 作成したファイルと、各処理が何をしているかを日本語で説明してください。
  ```

- Copilotから質問が返ってきたら、対象の一覧、リンク、成功条件を補足します。
- ファイルの作成確認が表示されたら、内容を確認して許可します。
- 左側のファイル一覧で `tests/first-item.spec.ts` が作成されたことを確認します。
- 指定した一覧、先頭の項目のリンク、クリック後の成功条件がコードに含まれていることを確認します。
- 修正したい場合は、「このコードを、検索結果の先頭行の商品名リンクをクリックするように修正してください」のように、対象と期待する動作を具体的に伝えます。

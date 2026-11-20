# salesforce-agentforce — Agentforce と Claude × Salesforce を試す

Agentforce・Agentforce Vibes・Salesforce の Hosted MCP・Salesforce in Claude(通称 claudeforce)を、Developer Edition の org で試す。

## 使う org

**主に使うのは新しい org(A)。**古い org(B)は Agentforce が無いので、Hosted MCP の確かめにだけ使った。

### A: Agentforce 付きの Developer Edition(2026-11-14 作成)

| 項目 | 値 |
|---|---|
| サービス | Salesforce Developer Edition(Agentforce・Data Cloud 付き) |
| 目的 | Agentforce のエージェント・Claude からの Hosted MCP を試す |
| ログイン ID | `tnakanishi.66a67f5a215d@agentforce.com`(Salesforce CLI のエイリアス `trailhead`) |
| My Domain | `https://orgfarm-0b95a1a8a2-dev-ed.develop.my.salesforce.com` |
| org ID | `00Dbm000019ftAvEAI` |
| 言語・時間帯 | `en_US`・`America/Los_Angeles`(サインアップの既定のまま。ユーザーの表示言語は日本語) |
| 期限 | なし(使っていれば続く) |
| 認証 | 本人が Mac で作って CLI に取り込んだ(2026-11-20) |

最初から有効だったもの: Einstein 生成 AI(`EinsteinGpt.enableEinsteinGptPlatform`)・エージェントの基盤(`AgentPlatform.enableAgentPlatform`)・Agentforce Vibes。
ライセンス: Agentforce (Default) 1・Data Cloud・Agentforce Service Agent Builder/User・Einstein Prompt Templates ほか。

2026-11-20 にしたこと:

| 何を | どうやって | 確かめたこと |
|---|---|---|
| Agentforce を有効化 | 設定 `EinsteinCopilot.enableEinsteinGptCopilot=true` をメタデータでデプロイ | 既定の従業員エージェント「Agentforce (Default)」(`Copilot_for_Salesforce`)が自動でできた |
| エージェントを使う権限 | 権限セット `CopilotSalesforceUser`(Agentforce デフォルトエージェントへのアクセス)を自分に付与。`CopilotSalesforceAdmin` は最初から付いていた | — |
| 既定のエージェントを有効化 | 画面: Setup > Agentforce エージェント > 行のメニュー「ビルダーで開く」>「有効化」。`sf agent activate` は既定のエージェントに使えない(`AgentIsDefaultError`) | `BotVersion.Status=Active`。会話プレビューで「取引先を3件表示して」に「There are no accounts to display.」と返った(データが空のため。応答は英語) |
| Hosted MCP | SObject Reads・SObject All を有効化(画面) | ボタンが「無効化」に変わった |
| Claude 用の OAuth クライアント | `force-app/` を `sf project deploy start -o trailhead` | org から取得し直して、下の表と同じ値を確かめた |

有効化のときに「設定の問題が検出されました」が出た: 「1 つ以上のサブエージェントが重複」「競合するサブエージェントのチェックが未完了」「Data Cloud が有効になっていない」。**自動の操作が「無視 & 有効化」を押して有効にした**(意図して選んだのではない)。気になれば、ビルダーで「無効化」し、チェックリストを見直す。

Claude のサーバー URL(この org): `https://api.salesforce.com/platform/mcp/v1/d/orgfarm-0b95a1a8a2-dev-ed/develop/platform/sobject-all`(読むだけなら末尾を `sobject-reads`)。

PC とスマホ:

- PC: Lightning の画面右上の Agentforce のアイコンからパネルを開く。エージェントを作り直すのは Setup > Agentforce エージェント
- スマホ: Salesforce モバイルアプリで、既定のエージェントが有効でユーザーに権限があれば Agentforce が出る(Trailhead の記述。実機ではまだ確かめていない)。Setup > Salesforce モバイルアプリケーション の「Agentforce 向けモバイルビルダー (ベータ)」は表示のカスタマイズで、ベータの規約が付くのでオフのまま
- Claude(PC・スマホ): 下の「Claude から Hosted MCP を使う」。鍵と秘密はこの org の `Claude MCP` から取る

推論モデル: 既定のエージェントは従来のビルダーで動き、画面に「GPT-4o は 2027-03 に廃止。新しい Agentforce ビルダーへ上げるか、推論モデルを GPT-5.6 Luna に替えて」と出る。
従来のビルダーの「エージェント設定」にはモデルを選ぶ欄が無い。新しいビルダー(Agent Script)ではサブエージェントごとにモデルを替えられ、Claude も選べるという記事がある(二次情報。未確認)。

### B: 古い Developer Edition(2019 年ごろ作成)

| 項目 | 値 |
|---|---|
| サービス | Salesforce Developer Edition |
| 目的 | Agentforce・Hosted MCP・Agentforce Vibes を試す |
| ログイン ID | `tnakanishi@sanei-clover.com.devhub`(Salesforce CLI のエイリアスは取り込み時 `trailhead`、2026-11-20 の作業中に `devhub` へ変わった。コマンドではユーザー名で指す) |
| My Domain | `https://sanei-dev-ed.my.salesforce.com` |
| org ID | `00D2v000002H7frEAC`(エイリアス `devhub` の `.restart` のユーザーも同じ org) |
| 期限 | なし(Developer Edition。長く使わないと停止される) |
| 認証 | Mac の CLI から `sf org auth show-sfdx-auth-url` で書き出し、WSL で `sf org login sfdx-url` で取り込んだ(2026-11-20) |

## B の org で何が使えるか(2026-11-20 に確かめた)

確かめ方: `sf org list metadata-types`・`sf org list metadata -m Settings`・権限セットライセンスの SOQL、Setup の画面(クイック検索)。

| 機能 | この org | 根拠 |
|---|---|---|
| Agentforce のエージェント(Agentforce Builder・従業員エージェント) | **使えない** | メタデータ型 `GenAiPlanner`・`GenAiPlugin`・`Bot` が無く、設定 `EinsteinGpt` も「unknown」。Setup に「Einstein 生成 AI」「エージェント」が無い。Einstein 生成 AI・Data Cloud の権限セットライセンスも無い |
| Salesforce モバイルアプリの Agentforce | **使えない** | 上と同じ(エージェントが作れないため) |
| Agentforce Vibes(VS Code 拡張・Agentforce Vibes IDE) | **有効**(既定のまま) | Setup「Agentforce Vibes拡張機能」。設定 `AgentforceForDevelopers` の `agentforceForDevelopersOptOut=false` |
| Hosted MCP サーバー(Setup > API カタログ > MCP サーバー) | **ある。SObject Reads と SObject All を稼働にした(2026-11-20。本人の了承済み)**。ほかの 11 個は非稼働 | 画面に「Salesforce MCP サーバーの使用に対して請求される場合があります。Flex Credits」と出る。Developer Edition に実際に請求が来るかは確かめていない |
| Agentforce レジストリ(外部の MCP サーバーの登録) | ある。登録 0 件 | Setup > Einstein > Agentforce レジストリ |
| Salesforce in Claude(claudeforce) | **前提を満たさない見込み** | Claude のヘルプが前提に「最新の Sales Cloud Enterprise Edition」と Salesforce のベータの承認を挙げる。Developer Edition への言及は無い |

Hosted MCP サーバーの一覧(標準): SObject All / SObject Mutations / SObject Reads / SObject Deletes / content-write / content-readonly / engagement-interaction / Data Cloud SQL / Comms Offer Agent / metadata-experts / crma-beta / salesforce-api-context / Headless360 (H360) MCP Server。

## Claude から Hosted MCP を使う(PC・スマホ)

Salesforce in Claude(ベータ)の代わりに、Claude の**カスタムコネクタ**で Hosted MCP サーバーへ直接繋ぐ。

### Salesforce 側(済み)

1. Setup > MCP サーバー で `SObject Reads`・`SObject All` を「有効化」(画面で操作。ボタンが「無効化」に変わったのを確かめた)
2. OAuth のクライアント `Claude_MCP`(External Client App)を `force-app/` のメタデータでデプロイ:
   `sf project deploy start -o <エイリアスかユーザー名>`(このディレクトリで。A・B の両方に入れた)

`Claude_MCP` の設定(org から取得し直して確かめた):

| 項目 | 値 | 画面の名前 |
|---|---|---|
| コールバック URL | `https://claude.ai/api/mcp/auth_callback` | Callback URL |
| スコープ | `MCP`・`RefreshToken` | Access MCP servers (mcp_api)・Perform requests at any time (refresh_token) |
| PKCE | 必須 | Require Proof Key for Code Exchange (PKCE) |
| シークレット | Web サーバーフロー・リフレッシュの両方で必須 | Require secret for Web Server Flow / Refresh Token Flow |
| JWT のアクセストークン | 有効 | Issue JSON Web Token (JWT)-based access tokens for named users |
| リフレッシュトークンのローテーション | 有効 | Enable Refresh Token Rotation |
| 許可されているユーザー | 全ユーザーが自己承認可能(`AllSelfAuthorized`) | Permitted Users |
| IP 制限 | 緩和(`Bypass`) | IP Relaxation: Relax IP restrictions |
| リフレッシュトークンの有効期間 | 365 日(作成時の既定) | Refresh Token Policy |

メタデータで詰まったところ:

- スコープの値は画面の名前と違う。`mcp_api` は `MCP`、`refresh_token` は `RefreshToken`(デプロイのエラーが有効な値の一覧を返す)
- JWT・ローテーションの項目は `ExtlClntAppGlobalOauthSettings`(`.ecaGlblOauth`)の `isNamedUserJwtEnabled`・`isRefreshTokenRotationEnabled`。ポリシーの側(`.ecaOauthPlcy`)に書くと「not valid」
- IP 制限の緩和は `ipRelaxationPolicyType` を `Bypass` にする(有効な値は `Enforce`・`Bypass_2factor`・`Bypass`・`Enforce_relaxrefresh`)
- ポリシー(`.ecaOauthPlcy`・`.ecaPlcy`)は初回のデプロイで自動で作られる。それを retrieve して直し、デプロイし直した

資料の食い違い: Trailhead は「Require secret for Web Server Flow」をオンに、Apex Hours はオフにする。Claude のカスタムコネクタにはシークレットを渡すので、Trailhead(Salesforce 公式)に合わせてオンにした。

### 本人が操作すること

1. **コンシューマーの鍵と秘密を取る**(使う org で): Setup > 外部クライアントアプリケーションマネージャー > `Claude MCP` > 設定 > OAuth 設定 > 「コンシューマーの鍵と秘密」。メールの確認コードを入れる。リポジトリには書かない
2. **claude.ai(PC のブラウザか Claude Desktop)でコネクタを足す**: 設定 > コネクタ > カスタムコネクタを追加
   - 名前: `Salesforce agentforce-dev`(A の org)
   - サーバーの URL: A は `https://api.salesforce.com/platform/mcp/v1/d/orgfarm-0b95a1a8a2-dev-ed/develop/platform/sobject-all`、B は `https://api.salesforce.com/platform/mcp/v1/d/sanei-dev-ed/platform/sobject-all`(読むだけなら末尾を `sobject-reads`)。Setup の MCP サーバーの詳細の「サーバー URL」と同じ。My Domain が `/d/<My Domain>/` の所に入る
   - 詳細設定: OAuth Client ID に鍵、OAuth Client Secret に秘密
3. 「連携/Connect」→ Salesforce のログイン画面で その org のユーザー(A は `tnakanishi.66a67f5a215d@agentforce.com`) で入り「許可」
4. スマホ: Claude アプリに同じアカウントでログインし、チャットの「+」> コネクタで `Salesforce agentforce-dev` をオンにする(web で足したコネクタがアプリにも出るかは、まだ確かめていない)

注意: External Client App は、作ってから使えるまで最大 30 分かかるという記事がある(Apex Hours)。「Invalid OAuth client」が出たら、設定を変える前に待つ。

## なぜエージェントが使えないか

Developer Edition に Agentforce と Data Cloud が付いたのは 2025-03(TDX 2025)以降に作った org から。この org は 2019 年ごろの org ID(`00D2v...`)で、付いていない。
エージェントを試すには、[Developer Edition のサインアップ](https://developer.salesforce.com/signup?lang=ja)で新しい org を作る(新しいユーザー名とメールの確認が要る)。
2026-11-20 に本人が新しく作った(上の A)。

## 出典(2026-11-20 に読んだ)

- [Set up Salesforce in Claude for your organization](https://support.claude.com/en/articles/16952184-set-up-salesforce-in-claude-for-your-organization)(Claude ヘルプ。ベータ・有料プラン・Sales Cloud Enterprise・AgentExchange での申込み)
- [Introducing the new Salesforce Developer Edition, now with Agentforce and Data Cloud](https://developer.salesforce.com/blogs/2025/03/introducing-the-new-salesforce-developer-edition-now-with-agentforce-and-data-cloud)(題と検索の抜粋だけ。本文は 403 で読めていない)
- [New in Salesforce Developer Edition: Agentforce Vibes IDE, Claude 4.5, MCP](https://developer.salesforce.com/blogs/2026/04/new-developer-edition-agentforce-vibes-claude-mcp)(同上)
- [Set up Claude connection to your Salesforce org(Trailhead)](https://trailhead.salesforce.com/content/learn/modules/salesforce-in-claude-for-admins/set-up-salesforce-in-claude)
- [How to Connect Claude with Salesforce Hosted MCP Servers(Salesforce Time)](https://salesforcetime.com/2026/07/15/how-to-connect-claude-with-salesforce-hosted-mcp-servers/)
- [Salesforce Headless 360 Hands-On(Apex Hours)](https://www.apexhours.com/salesforce-headless-360-hands-on-run-your-org-from-claude-with-the-hosted-mcp-server/)
- [Agentforce: Easily Switch LLM Models for Each Subagent with Model Override](https://medium.com/@marketingcloudtips/agentforce-easily-switch-llm-models-for-each-subagent-with-model-override-fc83493ee86b)(サブエージェントごとのモデル。二次情報)
- [Claudeforce Explained(Apex Hours)](https://www.apexhours.com/claudeforce-explained-what-salesforce-anthropic-actually-ship/)

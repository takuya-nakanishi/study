# salesforce-agentforce — Agentforce と Claude × Salesforce を試す

Agentforce・Agentforce Vibes・Salesforce の Hosted MCP・Salesforce in Claude(通称 claudeforce)を、Developer Edition の org で試す。

## 使う org

| 項目 | 値 |
|---|---|
| サービス | Salesforce Developer Edition |
| 目的 | Agentforce・Hosted MCP・Agentforce Vibes を試す |
| ログイン ID | `tnakanishi@sanei-clover.com.devhub`(Salesforce CLI のエイリアス `trailhead`) |
| My Domain | `https://sanei-dev-ed.my.salesforce.com` |
| org ID | `00D2v000002H7frEAC`(エイリアス `devhub` の `.restart` のユーザーも同じ org) |
| 期限 | なし(Developer Edition。長く使わないと停止される) |
| 認証 | Mac の CLI から `sf org auth show-sfdx-auth-url` で書き出し、WSL で `sf org login sfdx-url` で取り込んだ(2026-11-20) |

## この org で何が使えるか(2026-11-20 に確かめた)

確かめ方: `sf org list metadata-types`・`sf org list metadata -m Settings`・権限セットライセンスの SOQL、Setup の画面(クイック検索)。

| 機能 | この org | 根拠 |
|---|---|---|
| Agentforce のエージェント(Agentforce Builder・従業員エージェント) | **使えない** | メタデータ型 `GenAiPlanner`・`GenAiPlugin`・`Bot` が無く、設定 `EinsteinGpt` も「unknown」。Setup に「Einstein 生成 AI」「エージェント」が無い。Einstein 生成 AI・Data Cloud の権限セットライセンスも無い |
| Salesforce モバイルアプリの Agentforce | **使えない** | 上と同じ(エージェントが作れないため) |
| Agentforce Vibes(VS Code 拡張・Agentforce Vibes IDE) | **有効**(既定のまま) | Setup「Agentforce Vibes拡張機能」。設定 `AgentforceForDevelopers` の `agentforceForDevelopersOptOut=false` |
| Hosted MCP サーバー(Setup > API カタログ > MCP サーバー) | **ある。13 個とも非稼働** | 画面に「Salesforce MCP サーバーの使用に対して請求される場合があります。Flex Credits」と出る |
| Agentforce レジストリ(外部の MCP サーバーの登録) | ある。登録 0 件 | Setup > Einstein > Agentforce レジストリ |
| Salesforce in Claude(claudeforce) | **前提を満たさない見込み** | Claude のヘルプが前提に「最新の Sales Cloud Enterprise Edition」と Salesforce のベータの承認を挙げる。Developer Edition への言及は無い |

Hosted MCP サーバーの一覧(標準): SObject All / SObject Mutations / SObject Reads / SObject Deletes / content-write / content-readonly / engagement-interaction / Data Cloud SQL / Comms Offer Agent / metadata-experts / crma-beta / salesforce-api-context / Headless360 (H360) MCP Server。

## なぜエージェントが使えないか

Developer Edition に Agentforce と Data Cloud が付いたのは 2025-03(TDX 2025)以降に作った org から。この org は 2019 年ごろの org ID(`00D2v...`)で、付いていない。
エージェントを試すには、[Developer Edition のサインアップ](https://developer.salesforce.com/signup?lang=ja)で新しい org を作る(新しいユーザー名とメールの確認が要る)。

## 出典(2026-11-20 に読んだ)

- [Set up Salesforce in Claude for your organization](https://support.claude.com/en/articles/16952184-set-up-salesforce-in-claude-for-your-organization)(Claude ヘルプ。ベータ・有料プラン・Sales Cloud Enterprise・AgentExchange での申込み)
- [Introducing the new Salesforce Developer Edition, now with Agentforce and Data Cloud](https://developer.salesforce.com/blogs/2025/03/introducing-the-new-salesforce-developer-edition-now-with-agentforce-and-data-cloud)(題と検索の抜粋だけ。本文は 403 で読めていない)
- [New in Salesforce Developer Edition: Agentforce Vibes IDE, Claude 4.5, MCP](https://developer.salesforce.com/blogs/2026/04/new-developer-edition-agentforce-vibes-claude-mcp)(同上)

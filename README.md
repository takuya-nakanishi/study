# study

勉強用のリポジトリ(private)。デモデータ・テストアカウント・試したコードを主題ごとに置き、そこで分かったことを LLM Wiki に貯める。
本物の事実を持つ fire(横断と個人の Wiki)に、架空のデータを混ぜないために分けた。

**規約は [AGENTS.md](AGENTS.md) が正。**この README は全体像の把握用。

## 構成

```
topics/     勉強の主題。1主題=1ディレクトリ(kebab-case)。デモデータ・試したコード・手順
llm-wiki/   LLM Wiki(原本 raw/・Wiki wiki/)と、問いと作業(backlog/)
scripts/    リポジトリの道具(scripts/wiki/ は Wiki の道具。agent-base から配る)
```

## 主題の一覧

| 主題 | 状態 | 概要 |
|---|---|---|
| [salesforce-agentforce](topics/salesforce-agentforce/) | 調査中 | Developer Edition で Agentforce・Hosted MCP・Claude × Salesforce を試す |

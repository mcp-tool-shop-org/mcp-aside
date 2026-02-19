<p align="center">
  <a href="README.md">English</a> | <strong>日本語</strong> | <a href="README.zh.md">中文</a> | <a href="README.es.md">Español</a> | <a href="README.fr.md">Français</a> | <a href="README.hi.md">हिन्दी</a> | <a href="README.it.md">Italiano</a> | <a href="README.pt-BR.md">Português</a>
</p>

<p align="center">
  <img src="logo.png" alt="mcp-aside ロゴ" width="280" />
</p>

<h1 align="center">mcp-aside</h1>

<p align="center">
  会話の途中でAIがメモを取れるMCPサーバー — タスクを中断させることなく。
</p>

<p align="center">
  <a href="#クイックスタート">クイックスタート</a> &middot;
  <a href="#仕組み">仕組み</a> &middot;
  <a href="#ツール">ツール</a> &middot;
  <a href="#設定">設定</a> &middot;
  <a href="#ライセンス">ライセンス</a>
</p>

---

## なぜ

LLMは物事を見失いがちです。ふとした思いつき、半分形になった懸念、「後で戻ろう」と思ったまま二度と戻らないこと。**mcp-aside**はモデルに専用のインボックスを提供します — レート制限、重複排除、自動期限付きなので、インボックス自体が問題になることはありません。

会話の横に置いた付箋のようなものです。モデルがメモを書き、あなた（またはモデル）が適切なタイミングでそれを読みます。

## 仕組み

1. モデルが`aside.push`を呼び出し、優先度付きの思考を送信します。
2. ガードレールが重複、レート制限、TTL上限をチェックします。
3. 通過すると、インジェクションがインメモリのインボックスに保存されます。
4. クライアントは`notifications/resources/updated`で通知を受けます。
5. 誰でも`interject://inbox`リソースを通じてインボックスを読めます。

データベースなし。永続化なし。サーバーが停止すればインボックスは消えます — これは意図的な設計です。

## クイックスタート

```bash
npm install
npm run build
node build/index.js
```

サーバーは**stdio**でMCPを使用します。任意のMCP対応クライアントから接続してください：

```json
{
  "mcpServers": {
    "aside": {
      "command": "node",
      "args": ["build/index.js"]
    }
  }
}
```

## ツール

| ツール | 機能 |
|---|---|
| `aside.push` | インボックスにインジェクションを追加。`text`、`priority`（low/med/high）、`reason`、`tags`、`expiresAt`、`source`、`meta`を指定可能。 |
| `aside.configure` | ガードレールを実行時に調整 — TTL上限、レート制限、重複排除ウィンドウ、通知閾値。 |
| `aside.clear` | インボックスを消去。 |
| `aside.status` | インボックスのサイズと現在のガードレール設定の読み取り専用スナップショット。 |

## リソース

| URI | 説明 |
|---|---|
| `interject://inbox` | 保留中のインジェクションのJSON配列（新しいもの順）。期限切れのアイテムは読み取り時にフィルタリング。 |

## ガードレール

すべて`aside.configure`で設定可能。デフォルト値：

| 設定 | デフォルト | 制御対象 |
|---|---|---|
| `defaultTtlSeconds` | 600（10分） | 明示的な期限が設定されていない場合のインジェクションの寿命 |
| `maxTtlSeconds` | 3600（1時間） | TTLのハードキャップ（呼び出し側がそれ以上を要求しても） |
| `dedupeWindowSeconds` | 300（5分） | 同一の優先度+テキスト+理由 = このウィンドウ内で抑制 |
| `rateLimitWindowSeconds` | 60 | レート制限のスライディングウィンドウ |
| `rateLimitMax` | low: 6, med: 3, high: 1 | 優先度ごと・ウィンドウごとの最大プッシュ数 |
| `notifyAtOrAbove` | high | この優先度以上のアイテムのみログ通知を送信 |

## 設定

### タイマートリガー

5分ごとに低優先度の「ブロッカーはありますか？」チェックインを自動プッシュします。通常のプッシュと同じガードレールを適用（重複排除やレート制限の対象）。`index.ts`の`startTimerTrigger`呼び出しをコメントアウトして無効化できます。

### MCPインスペクター

ローカルテスト用：

```
Transport: STDIO
Command:   node
Args:      build/index.js
```

## 備考

- ログは**stderr**に出力 — stdoutはMCP JSON-RPC用に予約。
- インボックスはエフェメラル。再起動 = クリーンスレート。
- インジェクションは新しいもの順で保存。期限切れのアイテムは読み取りとプッシュのたびに整理。

## ライセンス

[MIT](LICENSE)

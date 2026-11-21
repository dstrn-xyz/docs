# デプロイ

- [デプロイ](#デプロイ)
  - [はじめに](#はじめに)
  - [デプロイ戦略](#デプロイ戦略)
    - [直接ベアメタル（デフォルト）](#直接ベアメタル（デフォルト）)
    - [caddyを使用したリバースプロキシ](#caddyを使用したリバースプロキシ)
    - [cloudflareを使用したエッジデプロイ](#cloudflareを使用したエッジデプロイ)
  - [自動デプロイ](#自動デプロイ)
    - [直接ベアメタルのセットアップ](#直接ベアメタルのセットアップ)
    - [リバースプロキシのセットアップ](#リバースプロキシのセットアップ)
    - [エッジのセットアップ](#エッジのセットアップ)
    - [ドライランプレビュー](#ドライランプレビュー)
    - [システムインストール](#システムインストール)
  - [手動デプロイのセットアップ](#手動デプロイのセットアップ)
    - [手動直接ベアメタル](#手動直接ベアメタル)
    - [手動caddyリバースプロキシ](#手動caddyリバースプロキシ)
  - [tlsおよび証明書管理](#tlsおよび証明書管理)
    - [証明書の自動検出](#証明書の自動検出)
    - [証明書管理コマンド](#証明書管理コマンド)
      - [証明書の取得](#証明書の取得)
      - [証明書の更新](#証明書の更新)
      - [証明書の状態確認](#証明書の状態確認)
    - [マルチドメインとsniルーティング](#マルチドメインとsniルーティング)
    - [httpからhttpsへのリダイレクト](#httpからhttpsへのリダイレクト)
  - [websocketセキュア接続](#websocketセキュア接続)
    - [自動wss昇格](#自動wss昇格)
    - [ネイティブwebsocketアップグレード](#ネイティブwebsocketアップグレード)
  - [systemdサービスセットアップ](#systemdサービスセットアップ)
    - [アプリケーションサービスユニット](#アプリケーションサービスユニット)
    - [証明書更新タイマーユニット](#証明書更新タイマーユニット)
  - [環境設定](#環境設定)
  - [パフォーマンスに関する考慮事項](#パフォーマンスに関する考慮事項)

<a name="はじめに"></a>

## はじめに

dframeworkは生のスループットと低レイテンシの実行のために設計されています。本番環境向けにdframeworkは3つのデプロイアーキテクチャを提供します:

1. ネイティブtlsを使用した直接ベアメタル（デフォルト）: ネイティブトランスポートアドオンがopenssl tls 1.3の終端、httpからhttpsへのリダイレクト、およびルーティングを処理する単一プロセスアーキテクチャ。
2. リバースプロキシ（`--proxy`）: 他のサービスとポートを共有する場合や既存のプロキシスタックと統合する場合のcaddyまたはnginxの背後での従来のデプロイ。
3. エッジデプロイ（`--edge`）: エッジでtls終端が行われるcloudflareなどのcdnの背後でプレーンhttpを実行。

<a name="デプロイ戦略"></a>

## デプロイ戦略

<a name="直接ベアメタル（デフォルト）"></a>

### 直接ベアメタル（デフォルト）

直接ベアメタルモードではdframeworkはネイティブopenssl tlsでポート443をバインドし同時にポート80で軽量なリスナーを実行してhttpトラフィックをhttpsにリダイレクトします。

```
[ client ] --( https:443 )--> [ dframework ssl app (native tls) ]
[ client ] --( http:80 )----> [ dframework redirect app (301) ]
```

メリット:

- リバースプロキシのオーバーヘッドがゼロ（余分なtcpループバックホップやコンテキストスイッチがない）。
- `storage/certs/{domain}/`によるネイティブマルチドメインsniサポート。
- 非root権限でsystemd配下で実行される単一のnodeプロセス。

<a name="caddyを使用したリバースプロキシ"></a>

### caddyを使用したリバースプロキシ

リバースプロキシモードではcaddyなどの専用Webサーバーがポート443でパブリックhttpsトラフィックを受信しローカルループバック（`127.0.0.1:8250`）経由でプレーンhttpリクエストをdframeworkにプロキシします。

```
[ client ] --( https:443 )--> [ caddy ] --( loopback:8250 )--> [ dframework ]
```

リバースプロキシを選択する場合:

- ポート80と443を共有して同一サーバー上で複数の独立したアプリケーションをホストする場合。
- カスタムプロキシミドルウェアや外部wafルールを必要とする既存のインフラストラクチャと統合する場合。

<a name="cloudflareを使用したエッジデプロイ"></a>

### cloudflareを使用したエッジデプロイ

エッジモードではcloudflareがパブリックtls接続を終端しhttpまたはhttps経由でオリジンサーバーにリクエストを転送します。

```
[ client ] --( https )--> [ cloudflare edge ] --( http/https )--> [ dframework ]
```

メリット:

- エッジでのddos保護とグローバルcdnキャッシング。
- オリジンサーバーを内部ポートで実行しcloudflareのipアドレスへのアクセスを制限するファイアウォールルールを適用可能。

<a name="自動デプロイ"></a>

## 自動デプロイ

`dstrn deploy`コマンドは設定生成、証明書プロビジョニング、およびサービスインストールを自動化します。

<a name="直接ベアメタルのセットアップ"></a>

### 直接ベアメタルのセットアップ

ドメイン向けにベアメタルの本番設定を生成するには:

```bash
dstrn deploy --domain=api.example.com --email=admin@example.com
```

マルチドメイン設定の場合はカンマ区切りでドメインを指定します:

```bash
dstrn deploy --domain=api.example.com,admin.example.com --email=admin@example.com
```

このコマンドは以下のアクションを実行します:

1. `.env`内の本番用`APP_KEY`を検証または生成します。
2. `storage/certs/{domain}/`内の既存の証明書を確認し不足している証明書を取得します。
3. アプリケーションがroot権限なしでポート80および443をバインドできるように`AmbientCapabilities=CAP_NET_BIND_SERVICE`を含む`deploy/dframework-<app>.service`を作成します。
4. 毎週の証明書更新を自動化するために`deploy/dframework-<app>-renew.timer`と`deploy/dframework-<app>-renew.service`を作成します。

<a name="リバースプロキシのセットアップ"></a>

### リバースプロキシのセットアップ

caddyおよびsystemdサービス設定を生成するには:

```bash
dstrn deploy --proxy --domain=api.example.com
```

これにより以下が生成されます:

1. アプリケーションポートを指すリバースプロキシのルールを含む`deploy/Caddyfile`。
2. 標準の非特権実行用に設定された`deploy/dframework-<app>.service`。

<a name="エッジのセットアップ"></a>

### エッジのセットアップ

cloudflareガイダンス付きのsystemdサービスファイルを生成するには:

```bash
dstrn deploy --edge --domain=api.example.com
```

<a name="ドライランプレビュー"></a>

### ドライランプレビュー

ディスクに書き込まずにターミナルで生成されたすべてのサービスおよび設定ファイルをプレビューします:

```bash
dstrn deploy --domain=api.example.com --email=admin@example.com --dry-run
```

<a name="システムインストール"></a>

### システムインストール

サーバー上でroot権限（sudo）で実行している場合`dstrn deploy`は`/etc/systemd/system/`に直接書き込みサービスをアクティブ化できます:

```bash
sudo dstrn deploy --domain=api.example.com --email=admin@example.com --install
```

<a name="手動デプロイのセットアップ"></a>

## 手動デプロイのセットアップ

手動でのサーバープロビジョニングを希望する場合や自動化ツールを使用している場合は以下の手順に従ってください。

<a name="手動直接ベアメタル"></a>

### 手動直接ベアメタル

ssl証明書と秘密キーをアプリケーションのstorageディレクトリに配置します:

```
storage/certs/api.example.com/
  cert.pem
  key.pem
```

`.env`で本番環境変数を設定します:

```ini
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.example.com
APP_KEY=your_64_character_hex_key
```

`/etc/systemd/system/dframework.service`を作成します:

```ini
[Unit]
Description=dframework application server
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node dstrn serve
Restart=always
RestartSec=3
Environment=NODE_ENV=production
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

サービスを有効化して起動します:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dframework
sudo systemctl start dframework
```

<a name="手動caddyリバースプロキシ"></a>

### 手動caddyリバースプロキシ

debianまたはubuntuにcaddyをインストールします:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

`/etc/caddy/Caddyfile`を設定します:

```caddy
api.example.com {
    reverse_proxy 127.0.0.1:8250
}
```

caddyをリロードします:

```bash
sudo systemctl reload caddy
```

caddyは`api.example.com`のlet's encrypt証明書を自動的に取得し更新を管理しループバック経由でリクエストをdframeworkにプロキシします。

<a name="tlsおよび証明書管理"></a>

## tlsおよび証明書管理

<a name="証明書の自動検出"></a>

### 証明書の自動検出

dframeworkは起動時に`storage/certs/`内の証明書を自動的に検出します:

```
storage/certs/
  api.example.com/
    cert.pem (or fullchain.pem, certificate.crt, api.example.com.crt)
    key.pem  (or privkey.pem, private.key, api.example.com.key)
    ca.pem   (optional intermediate chain)
  admin.example.com/
    cert.pem
    key.pem
```

有効な証明書が見つかり`APP_URL`が`https://`で始まる場合dframeworkは自動的にポート443でネイティブopenssl tlsを使用して起動しポート80でhttpからhttpsへのリダイレクトリスナーを開始します。

<a name="証明書管理コマンド"></a>

### 証明書管理コマンド

`dstrn cert`コマンドファミリーは証明書のライフサイクル管理を提供します。

<a name="証明書の取得"></a>

#### 証明書の取得

1つ以上のドメインに対して新しいlet's encrypt証明書を取得します:

```bash
dstrn cert:obtain --domain=api.example.com --email=admin@example.com
```

> [!TIP]
> デプロイ検証中にlet's encryptのレート制限に達するのを避けるため本番発行の前に`--staging`を使用して証明書生成をテストしてください。

```bash
dstrn cert:obtain --domain=api.example.com --email=admin@example.com --staging
```

<a name="証明書の更新"></a>

#### 証明書の更新

30日以内に期限切れになるすべての証明書を更新します:

```bash
dstrn cert:renew --domain=api.example.com --email=admin@example.com
```

<a name="証明書の状態確認"></a>

#### 証明書の状態確認

インストールされているすべての証明書の有効期限、発行者情報、および有効性を確認します:

```bash
dstrn cert:status
```

<a name="マルチドメインとsniルーティング"></a>

### マルチドメインとsniルーティング

トランスポートはserver name indication（sni）をサポートしています。`storage/certs/`内に複数のドメインディレクトリが存在する場合tlsハンドシェイク中に各ドメインに対応する証明書が提供されます。

ワイルドカード証明書はアンダースコア始まりのディレクトリに配置できます（例: `*.example.com`用の`storage/certs/_.example.com/`）。ワイルドカード一致は厳密な単一階層サブドメイン規則に従うため、`_.example.com`は`api.example.com`には一致しますが、複数階層サブドメイン（`a.b.example.com`）やapexドメイン（`example.com`）には一致しません。apexドメインも併用するマルチ証明書構成では、apex用の証明書を独立した完全一致ディレクトリ（`storage/certs/example.com/`）に配置してください。

これをアプリケーションルートのドメインルートグループと組み合わせます:

```javascript
Route.domain('api.example.com', (api) => {
  api.get('/status', async () => json({ service: 'api' }));
});

Route.domain('admin.example.com', (admin) => {
  admin.get('/status', async () => json({ service: 'admin' }));
});
```

<a name="httpからhttpsへのリダイレクト"></a>

### httpからhttpsへのリダイレクト

tlsが有効な場合dframeworkは自動的にポート80でネイティブの非sslリスナーを開始します。`http://domain/path`へのリクエストは即座に`https://domain/path`を指す`301 Moved Permanently`レスポンスを受け取ります。

リダイレクトリスナーを無効にするには`config/app.js`で`tls.redirect: false`を設定するか`.env`で`TLS_REDIRECT=false`を設定します。

<a name="websocketセキュア接続"></a>

## websocketセキュア接続

<a name="自動wss昇格"></a>

### 自動wss昇格

ページが`https://`経由でロードされた場合クライアントはwebsocket接続を自動的に`wss://`に昇格させます。

最初のページリクエストがhttpからhttpsにリダイレクトされるためブラウザは常に`https:`経由でロードされ安全な`wss://`接続を確立します。

<a name="ネイティブwebsocketアップグレード"></a>

### ネイティブwebsocketアップグレード

サーバー側ではポート443を介したwebsocket接続はsslイベントループ内で直接処理されます。httpアップグレードハンドシェイクは既存の暗号化されたtls接続上で行われるため平文の露出がゼロになり即座にwebsocketが利用可能になります。

<a name="systemdサービスセットアップ"></a>

## systemdサービスセットアップ

<a name="アプリケーションサービスユニット"></a>

### アプリケーションサービスユニット

本番systemdユニットファイル（`/etc/systemd/system/dframework.service`）:

```ini
[Unit]
Description=dframework application server
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node dstrn serve
Restart=always
RestartSec=3
Environment=NODE_ENV=production
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

> [!NOTE]
> `AmbientCapabilities=CAP_NET_BIND_SERVICE`によりsystemdサービスは非特権ユーザーアカウントで実行しながら特権下位ポート（80および443）をバインドできます。

<a name="証明書更新タイマーユニット"></a>

### 証明書更新タイマーユニット

自動証明書更新には週次のsystemdタイマー（`/etc/systemd/system/dframework-renew.timer`）を使用します:

```ini
[Unit]
Description=certificate renewal timer for dframework

[Timer]
OnCalendar=weekly
RandomizedDelaySec=3600
Persistent=true

[Install]
WantedBy=timers.target
```

対応するoneshot更新サービス（`/etc/systemd/system/dframework-renew.service`）:

```ini
[Unit]
Description=certificate renewal for dframework

[Service]
Type=oneshot
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node dstrn cert:renew --domain=api.example.com --email=admin@example.com
ExecStartPost=/bin/systemctl restart dframework
```

更新タイマーを有効化します:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dframework-renew.timer
sudo systemctl start dframework-renew.timer
```

<a name="環境設定"></a>

## 環境設定

標準的な本番環境の`.env`設定:

```ini
APP_NAME=dframework
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.example.com
APP_KEY=your_generated_64_character_hex_key
ACME_EMAIL=admin@example.com
```

オプションの明示的なtls設定（自動検出規約をオーバーライド）:

```ini
TLS_CERT=storage/certs/api.example.com/cert.pem
TLS_KEY=storage/certs/api.example.com/key.pem
TLS_CA=storage/certs/api.example.com/ca.pem
TLS_REDIRECT=true
```

<a name="パフォーマンスに関する考慮事項"></a>

## パフォーマンスに関する考慮事項

従来のアーキテクチャではアプリケーションランタイムの前にcaddyやnginxなどのリバースプロキシを配置します。ベンチマーク測定では二重ソケットバッファリング、プロセスのコンテキストスイッチ、およびループバックtcpシリアライゼーションによりリバースプロキシレイヤーが単純なルートに対して60%以上のオーバーヘッドを追加する可能性があります。

直接ベアメタルデプロイは中間プロキシレイヤーを排除し自動化されたlet's encrypt証明書管理とマルチドメインサポートを維持しながらアプリケーションのイベントループに最大のスループットを直接提供します。
# ネイティブブリッジとプラグイン

- [ネイティブブリッジとプラグイン](#ネイティブブリッジとプラグイン)
  - [はじめに](#はじめに)
  - [App.nativeファサード](#App.nativeファサード)
    - [メソッド呼び出しの解決方法](#メソッド呼び出しの解決方法)
    - [Promiseとエラー](#Promiseとエラー)
    - [自動登録されるコアプラグイン](#自動登録されるコアプラグイン)
  - [authプラグイン](#authプラグイン)
  - [deviceプラグイン](#deviceプラグイン)
  - [storageプラグイン](#storageプラグイン)
  - [cryptoプラグイン](#cryptoプラグイン)
  - [audioプラグイン](#audioプラグイン)
  - [notificationsプラグイン](#notificationsプラグイン)
  - [独自プラグインの作成](#独自プラグインの作成)
    - [4ファイル構成の規約](#4ファイル構成の規約)
    - [javascriptリファレンス実装](#javascriptリファレンス実装)
    - [プラグインのスキャフォールディング](#プラグインのスキャフォールディング)
    - [信頼できる情報源としてのjavascriptの記述](#信頼できる情報源としてのjavascriptの記述)
    - [ios実装](#ios実装)
    - [android実装](#android実装)
    - [desktop実装](#desktop実装)
    - [フォールバックルール](#フォールバックルール)
    - [エンドツーエンドの例（バーコードスキャナー）](#エンドツーエンドの例（バーコードスキャナー）)
  - [トラブルシューティング](#トラブルシューティング)

<a name="はじめに"></a>

## はじめに

ネイティブブリッジを使用するとウェブスタックから離れることなくios、android、desktop上のデバイスapiにjavascriptコードからアクセスできます。すべてが単一のオブジェクト`App.native`を通じて公開されすべてのプラットフォームで同じように動作します。

各機能は「プラグイン」です。フレームワークには6つのコアプラグインが同梱されており必要に応じて独自のプラグインを自由に追加できます。すべてのプラグインは同じ4ファイル規約（言語ごとに1ファイル）に従うためjavascriptで一度記述したメソッドにはswift、kotlin、rustの対応する実装が存在し必要に応じて実装またはスキップできます。

このドキュメントではその両面を扱います。アプリケーションコードから呼び出す公開`App.native.*`インターフェースと機能をプラットフォーム上でネイティブに実行したい場合に使用する4ファイルの構成です。

<a name="App.nativeファサード"></a>

## App.nativeファサード

`App.native`はブラウザとネイティブwebviewの両方でグローバルスコープに自動的に添付されるオブジェクトです。各プラグインはこのオブジェクトのプロパティであり各プロパティはプラグインのメソッドを直接公開します。

```javascript
// every call is a promise
const pub = await App.native.crypto.generate();
await App.native.storage.set('theme', 'dark');
const id = await App.native.notifications.show('pairing complete', 'tap to open', 'pairing-1');
```

<a name="メソッド呼び出しの解決方法"></a>

### メソッド呼び出しの解決方法

`App.native.<plugin>.<method>(args)`を呼び出すとブリッジはネイティブランタイムが起動時に登録したプラグインごとのメソッドマップを確認します:

- メソッドがネイティブメソッドとして登録されている場合呼び出しはネイティブレイヤー（iosではswift、androidではkotlin、desktopではtauriコマンド）に転送されネイティブの結果がpromiseとして返されます。
- メソッドがネイティブとして登録されていない場合呼び出しはjavascriptリファレンス実装を直接実行します。これはそのプラットフォームにネイティブファイルが存在しないプラグイン（例: desktop audio、desktop notifications、desktop crypto、desktop device）の場合やネイティブレイヤーが応答しない場合のフォールバックとして発生します。

この判定はプラグイン単位ではなくメソッド単位で行われます。同一プラットフォーム上でアプローチを混在させたい場合`audio.play`をネイティブとして登録し（iosでavplayerを使用）、`audio.load`を未登録のままにしておく（html audio要素を使用）ことができます。

<a name="Promiseとエラー"></a>

### Promiseとエラー

`App.native.*`のすべてのメソッドはpromiseを返します。リジェクト時には明確なエラーメッセージが含まれます。真偽値を返すメソッド（ストレージ書き込みの成功など）は例外をスローせず失敗時に`false`を返します。処理を継続できないメソッド（apiが利用できない場合のnotificationsなど）はエラーをスローします。

最もシンプルな呼び出しには`async`関数内で単一の`await`を使用します:

```javascript
const granted = await App.native.notifications.requestPermission();
if (granted !== 'granted') {
  notify('enable notifications to receive pairing events', 'warning');
  return;
}
```

<a name="自動登録されるコアプラグイン"></a>

### 自動登録されるコアプラグイン

フレームワークは6つのコアプラグインを自動的に登録します。各プラグインの詳細は以下のとおりです:

- `auth`（ローカルセッションidと汎用キーバリューストア）
- `device`（プラットフォームおよびモデルのメタデータ）
- `storage`（汎用文字列キーバリューストア）
- `crypto`（ecdsa p 256 sha 256のキーペア生成、署名、検証、ダイジェスト）
- `audio`（メディアセッションメタデータを備えた単一url再生）
- `notifications`（ローカル通知の表示とキャンセル）

<a name="authプラグイン"></a>

## authプラグイン

authプラグインはすべてのデバイスに安定したセッションidと独自データを永続化するための小さなキーバリューストアを提供します。web実装では`localStorage`を使用しios実装では`UserDefaults`、android実装では専用の`SharedPreferences`ファイルを使用します。すべてのプラットフォームにおいてセッションキーは`dstrn.session.id`配下に名前空間が設定されキーバリューストアは`dstrn.db.<key>`配下に名前空間が設定されるため独自キーがセッションと衝突することはありません。

#### `App.native.auth.setSession(id)`

`id`を現在のデバイスセッションとして保存します。文字列に型変換されます。成功時は`true`を返し基礎となるストレージが例外をスローした場合（localStorageへの書き込みがブロックされるプライベートブラウジングモードなど）は`false`を返します。

```javascript
await App.native.auth.setSession(loginResponse.token);
```

#### `App.native.auth.getSession()`

保存されているセッションidを文字列として返します。保存されていない場合や読み取りで例外が発生した場合は`null`を返します。

```javascript
const id = await App.native.auth.getSession();
if (!id) redirect('/login');
```

#### `App.native.auth.clearSession()`

保存されているセッションを削除します。成功時は`true`、ストレージエラー時は`false`を返します。

```javascript
await App.native.auth.clearSession();
redirect('/');
```

#### `App.native.auth.db_get(key)`

以前に`dstrn.db.<key>`配下に保存された値を読み取ります。呼び出し元が保存した生の文字列を返します。キーが設定されていない場合は`null`を返します。

```javascript
const lastDeviceId = await App.native.auth.db_get('last_device_id');
```

#### `App.native.auth.db_set(key, value)`

`dstrn.db.<key>`配下に`value`を保存します。`value`はそのまま保存されるためjson文字列は展開されたり再エンコードされたりすることなく保持されます。成功時は`true`、ストレージエラー時は`false`を返します。

```javascript
await App.native.auth.db_set('last_device_id', deviceId);
await App.native.auth.db_set('user_blob', JSON.stringify(user));
```

#### authプラグインに関する注意点

- `db_set`は値の検証を行わないためパース済みオブジェクトを保存すると`[object Object]`になります。必ず文字列を保存してください。
- db名前空間はstorageプラグインとは意図的に分離されており必要に応じて2つ目の独立したキーバリューストアとして利用できます。
- webでは両方の名前空間が`localStorage`に存在するためサイトデータを消去すると両方とも削除されます。

<a name="deviceプラグイン"></a>

## deviceプラグイン

deviceプラグインは現在のデバイスのプラットフォームとモデルをレポートします。iosではデバイスの`UIDevice.current.name`とモデル識別子を返しandroidでは`Build.MANUFACTURER + Build.MODEL`を返しdesktopではホスト名を返します。webでは3つのメソッドがそれぞれ`navigator.userAgent`、`navigator.platform`、および文字列`'web'`を返します。

#### `App.native.device.name()`

人間が読めるデバイス名を返します。iosでは`設定 > 一般 > 情報 > 名前`で設定された名前（例: "jane's iphone"）になります。androidでは製造元とモデルを結合した文字列（例: "google pixel 8"）になります。desktopではホスト名になります。webではユーザーエージェント文字列になります。常に文字列を返しプラットフォームにアクセスできない場合は`'unknown'`にフォールバックします。

```javascript
const name = await App.native.device.name();
els.deviceBadge.textContent = name;
```

#### `App.native.device.model()`

製造元を含まないモデル識別子を返します。iosでは`'iPhone'`または`'iPad'`、androidではモデル名、desktopではホスト名になります。webでは`navigator.platform`になります。常に文字列を返しフォールバック時は`'unknown'`を返します。

```javascript
const model = await App.native.device.model();
```

#### `App.native.device.platform()`

`'ios'`、`'android'`、`'desktop'`、`'web'`のいずれかを返します。これはランタイムごとに処理を分岐する最もシンプルな方法です。

```javascript
const platform = await App.native.device.platform();
if (platform === 'ios' || platform === 'android') {
  // enable native only features
}
```

<a name="storageプラグイン"></a>

## storageプラグイン

ユーザー設定やui状態のための軽量な汎用キーバリューストアです。webでは`localStorage`、iosでは`UserDefaults`、androidでは`SharedPreferences`、desktopではプラットフォームのアプリケーションデータディレクトリ内のjsonファイルをバックエンドとします。値は常に文字列です（プラグインは書き込み時に非文字列の値を文字列に変換し読み取り時にはそのまま返します）。

storageプラグインとauth dbストアは独立した名前空間です。保存する対象の意味に合わせて使い分けてください。

#### `App.native.storage.set(key, value)`

`key`配下に`value`を保存します。成功時は`true`、失敗時は`false`を返します。非文字列の値は`String(value)`によって変換されるため読み取り時に適切に扱う限り数値、真偽値、json文字列のいずれも動作します。

```javascript
await App.native.storage.set('theme', 'dark');
await App.native.storage.set('fontSize', 16);
await App.native.storage.set('lastSeen', String(Date.now()));
```

#### `App.native.storage.get(key)`

保存されている文字列を返します。キーが設定されていない場合は`null`を返します。存在しないキーと空文字列を区別するには明示的に`null`をチェックしてください。

```javascript
const theme = await App.native.storage.get('theme');
if (theme === null) {
  // first run
}
```

#### `App.native.storage.remove(key)`

保存された値を削除します。値が削除された場合は`true`を返しそれ以外の場合は`false`を返します（キーが既に存在しない場合を含む）。

```javascript
await App.native.storage.remove('theme');
```

#### storageプラグインに関する注意点

- `keys`、`clear`、`list`メソッドはありません。これらが必要な場合は固定キー配下のjson配列として独自に永続化してください。
- 約5メガバイトを超える値は一部のプラットフォームで失敗する可能性があります（`localStorage`にはオリジンごとのクォータがあります）。大きなblobはウェブフレームワークの`Storage.disk()`を使用して保存してください。
- iosおよびandroidではストレージ書き込みは同期的ですがブリッジが解決前に待機するためpromiseパターンは期待どおりに機能します。

<a name="cryptoプラグイン"></a>

## cryptoプラグイン

cryptoプラグインはすべてのデバイスに安定したecdsa p 256 sha 256キーペアを提供し署名、検証、ダイジェストのためのヘルパーを備えています。すべてのプラットフォームでバイト単位で同一のキーと署名が返されるためiosで生成された署名はandroid、web、desktopでも検証できます。

キーペアはすべてのプラットフォームで安定したエイリアスに永続化されます（iosキーチェーンの`com.dstrn.crypto.main_key`、androidキーストアの`dstrn_crypto_main_key`、webのindexeddbの同一エイリアス）。`generate()`の呼び出しはべき等です（初回の呼び出しでキーペアを作成して永続化しそれ以降の呼び出しでは同じ公開キーを返します）。秘密キーが安全なストアから外部に出ることはありません。

#### `App.native.crypto.generate()`

デバイスの公開キーをbase64エンコードされた生の非圧縮ポイント（`0x04`で始まる65バイト）およびアルゴリズム文字列`'ECDSA-P256-SHA256'`として返します。べき等であり同一デバイス上で`generate()`を繰り返し呼び出しても常に同じ公開キーが返されます。

```javascript
const { publicKey, algorithm } = await App.native.crypto.generate();
console.log(algorithm); // 'ECDSA-P256-SHA256'
```

返されるオブジェクトは`{ publicKey: string, algorithm: string }`です。

#### `App.native.crypto.sign(data)`

デバイスの秘密キーを使用して`data`（base64文字列）に署名します。`{ signature: string, algorithm: string }`を返します。ここで`signature`はbase64エンコードされたder asn.1署名です。キーペアが利用できない場合（正常に`generate()`を実行した後は発生しません）や`data`が有効なbase64文字列でない場合はエラーでリジェクトされます。

```javascript
const payload = btoa(JSON.stringify({ op: 'pair', device: deviceName }));
const { signature } = await App.native.crypto.sign(payload);
```

同じペイロードと同じpublicKeyはどのプラットフォームでも検証可能です。

#### `App.native.crypto.verify(data, signature, publicKey)`

`signature`が`publicKey`における`data`の有効な署名であることを検証します。3つの引数はすべてbase64文字列です。成功時は`true`、失敗時（不正な形式のbase64、不正な形式の署名、改ざんされた署名など）は`false`を返します。

```javascript
const ok = await App.native.crypto.verify(payload, signature, senderPublicKey);
if (!ok) throw new Error('signature mismatch');
```

#### `App.native.crypto.digest(data, algo?)`

要求されたアルゴリズムで`data`（base64文字列）をハッシュ化します。`algo`のデフォルトは`'SHA-256'`です。`'SHA-1'`および基礎となるプラットフォームがサポートする任意のアルゴリズムも受け付けます（webとiosは`'SHA-256'`と`'SHA-1'`を受け付け、androidはハイフン付きまたはハイフンなしの同等形式を受け付けます）。ハッシュをbase64文字列として返します。

```javascript
const hash = await App.native.crypto.digest(btoa('abc'), 'SHA-256');
// hash === 'ungWv48Bz+pBQUDeXa4iI7ADYaOWG3Yr0FSxiyEyvg7gq9B3/L1lqg=='
```

#### `App.native.crypto.export()`

署名を行わずにデバイスの現在のキーペアの`{ publicKey, algorithm }`を返します。公開アイデンティティのみをピアと共有する必要がある場合に便利です。`generate()`と同じように動作しますがより短い呼び出しで済みます。

```javascript
const { publicKey } = await App.native.crypto.export();
```

#### `App.native.crypto.import(publicKey)`

外部の公開キーをローカルキーストアにインポートし以降の`verify`呼び出しで再インポートなしで使用できるようにします。成功時は`true`を返します。webではこれはキーの形状を検証して`true`に解決するだけのno opです。

```javascript
await App.native.crypto.import(peerPublicKey);
const ok = await App.native.crypto.verify(data, signature, peerPublicKey);
```

#### cryptoプラグインに関する注意点

- 公開キーはpemやx509 blobではなく65バイトの生のバイト列（0x04プレフィックスとxおよびy座標）です。これはiosの`SecKeyCreateWithData`およびandroidの`ECPublicKeySpec`再構築パスで使用される形式と一致するためトランスコードなしでクロスプラットフォーム検証が機能します。
- 署名はderエンコードされたasn.1（`SHA256withECDSA`）であり各プラットフォームがネイティブに生成する形式です。エントロピーの高いビットに応じてバイト長は約64から72の間で変動します。
- webではキーペアはindexeddbの`dstrn_crypto / keys / dstrn_crypto_main_key`に保存されます。サイトデータを消去するとキーペアが削除され次回の`generate()`でデバイスは新しいキーペアを作成します。デバイス間でキーペアを移行する方法はありません。

<a name="audioプラグイン"></a>

## audioプラグイン

メディアセッションメタデータを備えた単一urlオーディオ再生によりオペレーティングシステムはロック画面や通知領域にタイトル、アーティスト、アルバム、アートワーク、再生コントロールを表示できます。ページごとに1つの共有プレイヤーが存在するため`load()`を呼び出すとソースが切り替わります。キューや複数urlモードはありません（一度に1トラックのみ）。

iosではプラグインは`AVPlayer`を使用しバックグラウンド再生とネイティブの再生中コントロールをサポートします。androidでは同様の効果を得るために`MediaPlayer`と`MediaSessionCompat`を使用します。webおよびdesktopではプラグインは`HTMLAudioElement`と`MediaSession`にフォールバックします。これはアプリ内再生とロック画面コントロールには十分ですがアプリが中断されたときに再生を継続することはできません。

#### `App.native.audio.load(url)`

現在のソースとして`url`を読み込みます。サポートされるurlは1つのみです。再度`load()`を呼び出すと前回のトラックが置き換えられます。成功時は`true`を返しurlが無効な場合やaudio要素が拒否された場合は`false`を返します。

```javascript
await App.native.audio.load(episodeUrl);
await App.native.audio.play();
```

#### `App.native.audio.play()`

現在ロードされているトラックの再生を開始します。`true`を返します。ブラウザが自動再生をブロックした場合（ユーザー操作がまだない場合）基礎となるpromiseのリジェクトは無視され`true`が返されます。`play()`を呼び出す前にユーザーイベントを監視してユーザー操作を確認することをお勧めします。

```javascript
button.addEventListener('click', async () => {
  await App.native.audio.play();
});
```

#### `App.native.audio.pause()`

再生位置をリセットせずに再生を一時停止します。`true`を返します。

#### `App.native.audio.resume()`

`play()`のエイリアスです。システムメディアコントロールからは再生/一時停止として公開されているが独自のコードでは一時停止/再開としてモデル化されている場合に便利です。

#### `App.native.audio.seek(seconds)`

再生ヘッドをトラックの先頭から`seconds`秒の位置に移動します。数値を受け入れフレーム精度のシークのために小数の値もサポートされます。トラックがロードされている場合は`true`を返し要素がまだ存在しない場合（先に`load`を呼び出す必要があります）は`false`を返します。

```javascript
await App.native.audio.seek(120); // jump to two minutes in
```

#### `App.native.audio.stop()`

再生を一時停止しトラックの先頭に巻き戻します。`true`を返します。

#### `App.native.audio.setRate(rate, preservePitch?)`

再生速度を設定します。`rate`は数値です（等倍の半分なら`0.5`、2倍速なら`2`など）。`preservePitch`のデフォルトは`false`です。`false`の場合ピッチは速度に連動し`true`の場合ピッチは一定に保たれ速度のみが変化します。`true`を返します。

```javascript
await App.native.audio.setRate(1.5, true); // 1.5x speed, normal pitch
```

#### `App.native.audio.setMetadata(title, artist, album, duration, artwork)`

システムメディアセッションにメタデータを公開しロック画面や通知領域に再生内容を表示できるようにします。`title`以外のすべての引数はオプションです。`duration`は秒単位で数値である必要があります。`artwork`はurl文字列です。成功時は`true`を返しプラットフォームがメディアセッションをサポートしていない場合は`false`を返します。

```javascript
await App.native.audio.setMetadata(
  'episode 42: pairing flows',
  'dframework weekly',
  'season 2',
  1820,
  'https://cdn.example.com/art/episode-42.jpg'
);
```

#### audioプラグインに関する注意点

- 呼び出し間で同じaudio要素が再利用されます。呼び出しごとのプレイヤーオブジェクトはありません。
- プラグインは個別のurlにまたがる範囲リクエストシークを実装していません。1つのurlにつき1つのソースです。
- `setRate`は任意の正の数値を受け付けますが約`[0.25, 4]`の範囲外の値はすべてのプラットフォームで尊重されるとは限りません。

<a name="notificationsプラグイン"></a>

## notificationsプラグイン

通知ごとに安定したidを持つローカル通知であり表示後に更新またはキャンセルできます。webおよびdesktopではプラグインは`window.Notification`を使用しiosでは`UNUserNotificationCenter`を使用しandroidではandroid 13以降でランタイム権限リクエストを伴う`NotificationManagerCompat`を使用します。

#### `App.native.notifications.requestPermission()`

現在の権限状態を文字列（`'granted'`、`'denied'`、または`'prompt'`）として返します。状態が`'prompt'`の場合呼び出し時にユーザーへのプロンプトも表示されます（それ自体が`'granted'`または`'denied'`に解決される可能性があります）。通知apiのないプラットフォームでは`'denied'`を返します。

```javascript
const status = await App.native.notifications.requestPermission();
if (status !== 'granted') {
  return notify('enable notifications in settings', 'warning');
}
```

#### `App.native.notifications.show(title, body?, id?)`

表示タイトルとして`title`、表示本文として`body`を持つローカル通知を表示します。`id`はオプションです。指定した場合は後で`cancel(id)`を呼び出して通知を閉じることができます。指定しない場合プラグインは`'dstrn-0'`や`'dstrn-1'`のようなidを生成しますが後からその特定の通知を確実にキャンセルすることはできません。

実際に使用された通知id（指定した`id`または生成されたid）を返します。apiが利用できない場合や権限が拒否されている場合はエラーをスローします。

```javascript
const id = await App.native.notifications.show(
  'pairing complete',
  'tap to open the dashboard',
  'pairing-success-1'
);
```

#### `App.native.notifications.cancel(id)`

指定された`id`の通知を閉じます。通知が見つかり閉じられた場合は`true`を返しそのidの通知が認識されない場合は`false`を返します。

```javascript
await App.native.notifications.cancel('pairing-success-1');
```

#### notificationsプラグインに関する注意点

- `show()`を成功させるには事前に権限が付与されている必要があります。付与されていない場合は`permission denied`をスローします。
- 明示的な`id`を付与していない通知は設計上キャンセルの追跡対象外となります。後でキャンセルする必要がある場合は常に`id`を渡してください。
- プラグインは通知を将来の時刻にスケジュールしません。それが必要な場合はプッシュ通知またはジョブキューを使用してサーバーからスケジュールしてください。

<a name="独自プラグインの作成"></a>

## 独自プラグインの作成

すべてのプラグインは言語ごとに最大4つのファイルを含むフォルダです。フレームワークはプロジェクト内の`native/plugins/<name>/`配下の任意のフォルダを自動検出します。マニフェスト、スキーマ、レジストリ、ビルド配線の編集は不要です。ビルド時にフォルダが存在していればプラグインは出荷されます。

<a name="4ファイル構成の規約"></a>

### 4ファイル構成の規約

`barcode`という名前のプラグインは`native/plugins/barcode/`に配置されます:

```
native/plugins/barcode/
├── index.js       # required. javascript reference impl. source of truth for method names.
├── Plugin.swift   # optional. ios native impl. must export the same method names.
├── Plugin.kt      # optional. android native impl. must export the same method names.
└── Plugin.rs      # optional. desktop native impl. must export the same method names.
```

この構成には3つのルールがあります:

> [!IMPORTANT]
> `index.js`はすべてのプラグインで必須です。リファレンス実装、メソッド名の信頼できる情報源、およびネイティブ実装が存在しないか失敗した場合の自動フォールバックとして機能します。

- `Plugin.swift`、`Plugin.kt`、`Plugin.rs`はいずれもオプションです。チームがサポートできるネイティブ言語のものを出荷してください。ファイルが存在しない場合プラグインはそのプラットフォームでjavascript実装にフォールバックします。
- メソッド名は言語間で文字列の一致によって照合されます。`index.js`が`barcode.scan`をエクスポートしている場合`Plugin.swift`には`case "scan":`ブランチが必要であり`Plugin.kt`は`"scan"`にマッチし`Plugin.rs`は`"scan"`にディスパッチする必要があります。ブリッジ経由で送信される値がjsonエンコード可能である限りパラメータの形状と戻り値の形状は各言語独自の問題です。

<a name="javascriptリファレンス実装"></a>

### javascriptリファレンス実装

`index.js`はキーがメソッド名で値が関数であるオブジェクトを返す通常のモジュールです。関数本体はブラウザまたはwebview内で実行され通常のブラウザグローバル（`window`、`localStorage`、`navigator`、`fetch`、`crypto.subtle`、`MediaSession`、`Notification`、`Audio`など）のすべてにアクセスできます。

メソッドはホスト環境の全機能を使って実行されます。通常のフロントエンドjavascriptと同様に扱ってください。ネイティブ側がプラットフォームごとメソッドごとにオーバーライドできるようにプラグインとしてパッケージ化されているだけです。

```javascript
// native/plugins/barcode/index.js
return {
  scan() {
    return { ok: false, error: 'not implemented on this platform' };
  },
};
```

<a name="プラグインのスキャフォールディング"></a>

### プラグインのスキャフォールディング

cliは動作するjavascript実装と入力用のスタブネイティブファイルを含む4つのファイルを生成します:

```bash
dstrn make:plugin barcode
```

これにより`index.js`、`Plugin.swift`、`Plugin.kt`、`Plugin.rs`を含む`native/plugins/barcode/`が作成されます。必要なメソッドを編集すると次回の`dstrn simulate`または`dstrn build`で自動的に検出されます。

<a name="信頼できる情報源としてのjavascriptの記述"></a>

### 信頼できる情報源としてのjavascriptの記述

javascript実装は存在しないネイティブファイルに対するフォールバックとなるためどこでも動作するように設計しネイティブ実装はその上のパフォーマンスや機能のアップグレードとして扱います。機能がweb上でどうしても動作しない場合（カメラキャプチャなど）はjavascript実装から明確なエラーを返します:

```javascript
return {
  async scan() {
    if (typeof navigator === 'undefined' || !navigator.mediaDevices) {
      throw new Error('barcode: camera unavailable in this environment');
    }
    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
    stream.getTracks().forEach((t) => t.stop());
    return { value: 'scanned-value', format: 'qr_code' };
  },
};
```

その後プラグインはwebの動作を損なうことなくiosおよびandroidでネイティブ実装へとアップグレードされます。

<a name="ios実装"></a>

### ios実装

フレームワークはビルド中にiosランタイムプロジェクト内にプラグインごとの`Plugin.swift`を呼び出す`Plugins.swift`ファイルを生成します。`Plugin.swift`内では以下の形式に従います:

```swift
import Foundation

class BarcodePlugin {
    weak var bridge: Bridge?

    init(bridge: Bridge? = nil) {
        self.bridge = bridge
    }

    func handle(method: String, params: [String: Any], callId: String) {
        switch method {
        case "scan":
            scan(callId: callId)
        default:
            bridge?.reject(callId: callId, error: "barcode: unknown method \(method)")
        }
    }

    private func scan(callId: String) {
        // AVCaptureSession setup, delegate callbacks, etc.
        // call bridge?.resolve(callId: callId, result: [...]) on success
        // or bridge?.reject(callId: callId, error: "failed") on failure
    }
}
```

ブリッジは起動時に`barcode.scan`をネイティブメソッドとして登録します。呼び出しは`resolve`に渡したものを返します。

<a name="android実装"></a>

### android実装

フレームワークは各メソッドを`Plugin.kt`にルーティングする`Plugins.kt`ディスパッチャーを生成します。`Plugin.kt`内:

```kotlin
package com.dframework.native

import android.content.Context
import org.json.JSONObject

class BarcodePlugin(private val context: Context, private val bridge: Bridge) {
    fun handle(method: String, args: JSONObject, callId: String) {
        when (method) {
            "scan" -> scan(callId)
            else -> bridge.reject(callId, "barcode: unknown method $method")
        }
    }

    private fun scan(callId: String) {
        // CameraX or ML Kit setup
        // call bridge.resolve(callId, JSONObject().put("value", "payload")) on success
        // or bridge.reject(callId, "data") on failure
    }
}
```

<a name="desktop実装"></a>

### desktop実装

フレームワークは`plugins.rs`ディスパッチャーを生成します。`Plugin.rs`内:

```rust
use serde_json::{json, Value};

pub fn handle(method: &str, params: &Value) -> (bool, Value, Value) {
    match method {
        "scan" => (true, json!({ "ok": false, "error": "no camera in desktop build" }), json!(null)),
        _ => (false, json!(null), json!(format!("barcode: unknown method {}", method))),
    }
}
```

desktop webviewはほとんどのブラウザapi（`fetch`、`crypto.subtle`、`localStorage`など）を使用できるためdesktopプラグインは通常ブラウザにはない通知やファイルシステムアクセスなどtauri固有のapiが必要な場合にのみ作成します。desktopファイルを出荷しない場合ブリッジは自動的にjavascript実装にフォールバックします。

<a name="フォールバックルール"></a>

### フォールバックルール

どの実装を実行するかの決定はプラットフォームごとメソッドごとに行われます:

- `Plugin.swift`がiosで`case "scan":`ブランチを宣言している場合ブリッジは`barcode.scan`をネイティブとして登録しjavascriptからのすべての呼び出しはブリッジ経由で転送されます。
- そのブランチ（またはファイル全体）を削除した場合ブリッジはios上で`barcode.scan`をネイティブとして登録しないためwebview内でjavascript実装が実行されます。これが実際の「フォールバック」の意味です（ネイティブファイルが見つからないために失敗するプラットフォームコードパスはありません）。
- ブリッジがメソッドをネイティブとして登録しているにもかかわらずネイティブ呼び出しがリジェクトされた場合ブリッジはそのリジェクトをpromiseに伝播させます。サイレントにjavascriptにフォールバックすることはありません。ネイティブ実装は成功するか明確なメッセージとともにリジェクトするように設計してください。
- desktop audio、desktop notifications、desktop crypto、desktop deviceは現在すべてjavascriptにフォールバックします（コアフレームワークにはこれらに対応する`Plugin.rs`はありません）。desktopでプラットフォーム固有の動作が必要な場合にのみ記述してください。

<a name="エンドツーエンドの例（バーコードスキャナー）"></a>

### エンドツーエンドの例（バーコードスキャナー）

バーコードをスキャンしてその値を返す完全な最小プラグインです。動作するwebフォールバックとios / androidネイティブスタブを含みます。

```javascript
// native/plugins/barcode/index.js
return {
  async scan() {
    if (typeof navigator === 'undefined' || !navigator.mediaDevices) {
      throw new Error('barcode: camera unavailable');
    }
    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
    try {
      return { value: 'placeholder-decoded-value', format: 'qr_code' };
    } finally {
      stream.getTracks().forEach((t) => t.stop());
    }
  },
};
```

```swift
// native/plugins/barcode/Plugin.swift
import AVFoundation

class BarcodePlugin {
    weak var bridge: Bridge?

    init(bridge: Bridge? = nil) {
        self.bridge = bridge
    }

    func handle(method: String, params: [String: Any], callId: String) {
        switch method {
        case "scan": scan(callId: callId)
        default: bridge?.reject(callId: callId, error: "barcode: unknown method \(method)")
        }
    }

    private func scan(callId: String) {
        // AVCaptureMetadataOutput with .qr type, callback writes
        // bridge?.resolve(callId: callId, result: ["value": "data", "format": "qr_code"])
    }
}
```

```kotlin
// native/plugins/barcode/Plugin.kt
package com.dframework.native

import android.content.Context
import org.json.JSONObject

class BarcodePlugin(private val context: Context, private val bridge: Bridge) {
    fun handle(method: String, args: JSONObject, callId: String) {
        when (method) {
            "scan" -> scan(callId)
            else -> bridge.reject(callId, "barcode: unknown method $method")
        }
    }

    private fun scan(callId: String) {
        // CameraX + ML Kit, callback writes
        // bridge.resolve(callId, JSONObject().put("value", "payload").put("format", "qr_code"))
    }
}
```

```rust
// native/plugins/barcode/Plugin.rs
use serde_json::{json, Value};

pub fn handle(method: &str, params: &Value) -> (bool, Value, Value) {
    match method {
        "scan" => (false, json!(null), json!("barcode: camera not available on desktop")),
        _ => (false, json!(null), json!(format!("barcode: unknown method {}", method))),
    }
}
```

アプリケーションからこのプラグインを呼び出します:

```javascript
try {
  const result = await App.native.barcode.scan();
  console.log(result.value, result.format);
} catch (e) {
  notify('could not scan: ' + e.message, 'error');
}
```

iosおよびandroidではネイティブ実装が実行されます。webではjavascript実装が実行されます。desktopではrustディスパッチが明確なエラーでリジェクトします。1つの呼び出し元に対して4つの実装が存在します。

<a name="トラブルシューティング"></a>

## トラブルシューティング

**ブリッジが呼び出しを解決しない。** テスト対象のプラットフォームでそのメソッドがネイティブとして登録されているか確認してください。フレームワークには各プラットフォームで登録されているメソッドを報告する`dstrn native:status`および`dstrn native:doctor`コマンドが用意されています。

**メソッドがプラットフォーム間で異なる値を返す。** 返すjsonの形状がすべてのプラットフォームで同じであることを確認してください。ブリッジはiosの`JSONSerialization`、androidの`JSONObject`、desktopの`serde_json`を介して値を渡すためワイヤー形式は同じです。相違点は元の言語側にあります。デバッガーで解決された値を比較してください。

**プラグインフォルダが認識されない。** フォルダ名が`<name>/`と一致し`index.js`が存在することを確認してください。`dstrn native:doctor`は検出されたプラグインを報告し`index.js`がないものにフラグを立てます。

**ネイティブ呼び出しがタイムアウトする。** デフォルトのタイムアウトは30秒で`config/native.js`の`native.timeout`で設定可能です。実装にそれ以上の時間が必要な場合（大容量ファイルのアップロード、ネットワーク処理など）はタイムアウトを延ばすか処理をより小さな呼び出しに分割してください。
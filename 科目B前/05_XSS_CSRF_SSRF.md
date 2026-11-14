過去問題（R6春問3、R5秋問1、R5春Ⅱ問1、R4春Ⅱ問1）の出題および解答内容を元に、攻撃の流れの部分を抜き出して整理したもの。

## 0. 3つの違い（最初にここで仕分ける）

|      | 誰の権限で     | 何が動く／送られる                                                                                                                     | 攻撃者は応答を読めるか                | 主な狙い                                                                       |
| ---- | --------- | ----------------------------------------------------------------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------- |
| XSS  | 被害者（ブラウザ） | 攻撃者のスクリプトが、<mark style="background: #FFB8EBA6;">正規サイトのオリジン</mark>で動く                                                          | 読める（スクリプトが読んで外に送る）         | <mark style="background: #FFB8EBA6;">Cookie窃取、被害者権限での任意の操作、偽フォーム</mark>    |
| CSRF | 被害者（ブラウザ） | 攻撃者が<mark style="background: #FFB8EBA6;">決め打ちした1つのリクエスト</mark>が、<mark style="background: #FFB8EBA6;">被害者のCookie付き</mark>で送られる | 読めない（同一オリジンポリシー）           | <mark style="background: #FFB8EBA6;">登録情報の変更、送金、投稿など状態の変更</mark>           |
| SSRF | 脆弱なサーバ    | <mark style="background: #FFB8EBA6;">サーバ自身</mark>が、攻撃者の指定した宛先へリクエストを送る                                                        | 読める（返る場合）／読めない（Blind SSRF） | <mark style="background: #FFB8EBA6;">内部の管理画面、内部API、クラウドのメタデータ（IMDS）</mark> |

## 1. XSS（Cross-Site Scripting）

- Webアプリが<mark style="background: #FFB8EBA6;">利用者の入力をエスケープせずにHTMLへ出力</mark>することが原因。攻撃者のスクリプトが、被害者のブラウザ上で、そのサイトの<mark style="background: #FFB8EBA6;">オリジンの権限</mark>で実行される。
- スクリプトなので、Cookieを読む、画面を書き換える、条件分岐しながら複数のリクエストを送る、応答を読んで次の操作を決める、といった<mark style="background: #FFB8EBA6;">任意の動作</mark>ができる。

### 1.1 種類

| 種類        | 発火の仕方                                              | 誘導の要否                                                               |
| --------- | -------------------------------------------------- | ------------------------------------------------------------------- |
| 反射型       | リクエストの値がそのまま応答に出力される                               | <mark style="background: #FFB8EBA6;">罠リンク・罠サイトで被害者に送らせる必要</mark>がある |
| 格納型（持続型）  | 入力がDBなどに保存され、それを表示する別画面を見た人のブラウザで発火する              | 不要。<mark style="background: #FFB8EBA6;">表示した人全員が対象</mark>           |
| DOM-based | クライアント側のJavaScriptがDOMに書き込むときに発生。サーバの応答には現れないこともある | 種類による                                                               |
### 1.2 反射型の流れ（R5春Ⅱ問1：二つ先の確認画面で出力）

![[05_R5春Ⅱ問1_図3.png]]

1. 攻撃者は罠サイトを用意し、入力画面1の送信先URLあてに、<mark style="background: #FFB8EBA6;">姓（または名）にスクリプトを入れたフォーム</mark>を置く。
2. メールなどで利用者に<mark style="background: #FFB8EBA6;">罠サイトのリンクを踏ませる</mark>。
3. 罠サイトのJavaScriptが、利用者のブラウザからフォームを自動送信する。
4. 利用者のブラウザが正規サイトにスクリプト入りの値を送り、入力画面2が表示される（この時点ではまだ実行されない）。
5. 利用者が「次へ」を押す（または罠サイトが見えないiframeで次の遷移も自動送信させる）と、<mark style="background: #FFB8EBA6;">確認画面でエスケープされずに出力</mark>される。
6. スクリプトが利用者のブラウザで、<mark style="background: #FFB8EBA6;">正規サイトのオリジンとして実行</mark>される。
### 1.3 格納型の流れ（R6春問3：問合せ機能 → 管理者の画面）

![[05_R6春問3_図3.png]]

1. 攻撃者が問合せ機能で、<mark style="background: #FFB8EBA6;">Cookieを攻撃者のサイトへ送るスクリプト</mark>を格納する。
2. 管理者が問合せ管理機能で、その問合せを閲覧する。
3. <mark style="background: #FFB8EBA6;">スクリプトが管理者の権限で動き</mark>、管理者のCookie（SESSIONID）が攻撃者に届く（<mark style="background: #FFB8EBA6;">HttpOnly属性がないため読める</mark>）。
4. 攻撃者はそのCookieで会員管理機能にアクセスし、全会員の情報を取得する（セッションの有効期間内なら成立）。

- 入力した画面ではなく別画面に出力されるため、<mark style="background: #FFB8EBA6;">脆弱性診断ツールでは見つからず、設計書を調べた手動分析で発見</mark>された。

### 1.4 格納型の流れ（R5秋問1：字数制限の回避）

![[05_R5秋問1_図3.png]]

1. 攻撃者（会員A）が、50字制限を回避するためスクリプトを15件に分割して、レビュータイトル欄に投稿する。
2. 各投稿の<mark style="background: #FFB8EBA6;">末尾を「/*」、次の投稿の先頭を「*/」にして、投稿の間のHTML（&lt;/div&gt;など）をJavaScriptのコメント</mark>にする。
3. 新しい順に表示されるため、スクリプトの末尾から先に投稿する。
4. <mark style="background: #FFB8EBA6;">別の会員がレビュー閲覧ページを開く</mark>と、アプリはDBの内容をそのままHTMLに埋め込んで返す。
5. 投稿の間がコメントになり、1つの長いスクリプトとして、<mark style="background: #FFB8EBA6;">その会員のブラウザでサイトのオリジンとして実行</mark>される。
6. スクリプトが<mark style="background: #FFB8EBA6;">トークンを取得</mark>し、<mark style="background: #FFB8EBA6;">document.cookie（セッションID）をアイコン画像（a.png）としてアップロード</mark>する。
7. 攻撃者がその会員の<mark style="background: #FFB8EBA6;">アイコン画像をダウンロード</mark>し、テキストとして開いて<mark style="background: #FFB8EBA6;">セッションIDを取り出す</mark>。
8. そのセッションIDで会員になりすまし、購入やカード情報設定などの機能を使う。

### 1.5 属性値からの抜け出し（R4春Ⅱ問1：meta要素の content 属性）

![[05_R4春Ⅱ問1_図1.png]]

![[05_R4春Ⅱ問1_設問1.png]]
※正解はア（`confirm`までがパス名（scriptName）で`"><img src=1 onerror=alert(1)><"`がクエリ文字列（queryString））

```html
<!-- 出力のひな形 -->
<meta property="og:url" content="https://（serverName）/（scriptName）?【queryString】">

<!-- queryString = "><img src=1 onerror=alert(1)><" のとき（成立する） -->
<meta property="og:url" content="https://（serverName）/confirm?"><img src=1 onerror=alert(1)><"">
```
※serverNameはホスト名、scriptNameはパス名、queryStringはURLデコードされたクエリ文字列以降の値

1. ライブラリMが、`<meta property="og:url" content="https://（serverName）/（scriptName）?`の後にqueryStringをそのまま連結し、最後に`">`を付けて出力する。
2. queryString先頭の「"」で、content属性の値が閉じる。
3. 続く「>」で、metaタグが閉じる。
4. `<img src=1 onerror=alert(1)>`が独立したタグになり、画像の読み込みに失敗してonerrorが実行される。
5. 末尾の`<"`は、コードが付ける`">`と合わさって`<"">`になる。「<」の直後が英字ではないためタグとは解釈されず、ただの文字列として表示される。

- 「"」を `&quot;` にエスケープしていれば、属性値は閉じずにただの文字になる。
- `<img src=1 onerror=...>` は script という文字列を含まないので、<mark style="background: #FFB8EBA6;">「script」を消すだけのフィルタをすり抜ける。</mark>innerHTMLで挿入した `<script>` は実行されないが、onerror は実行される。
- Open Graph（Open Graph protocol、OGP）
	- WebページのURLがSNSやチャットアプリで共有されたときに、どのようなプレビューを表示するかを指定する仕組み
	- HTMLのheadにmetaタグで書く。
	- og:url：そのページの正規のURL
- 仕様で決まっている部分：属性値は次の「"」で終わる、「>」でタグが終わる、onerror は読み込み失敗で動く（HTML Living Standard）、og:url（Open Graph）。

### 1.6 対策（解答の型）

| 対策                                                                                                                                                                                                          | 位置付け                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| <mark style="background: #FFB8EBA6;">出力時のエスケープ</mark>（`< > & " '` を文字参照に変換）。出力先（HTML本文、属性値、JavaScript内、URL）に応じた方式を使う                                                                                        | 根本的解決（IPA「安全なウェブサイトの作り方」）。<mark style="background: #FFB8EBA6;">入力時のサニタイジングは不正解もしくは減点になりやすい</mark> |
| Cookieの<mark style="background: #FFB8EBA6;">HttpOnly属性</mark>                                                                                                                                               | Cookieは読めなくなるが、<mark style="background: #FFB8EBA6;">XSS自体は防げない。</mark>被害者権限でのリクエスト送信は可能           |
| <mark style="background: #FFB8EBA6;">CSP</mark>。script-srcを'self'（<mark style="background: #FFB8EBA6;">自オリジン以外の外部スクリプトと自オリジンでもインラインスクリプトの実行を禁止</mark>）や 'nonce-{ランダム値}'（nonceを指定し、nonceが一致するスクリプトだけを実行）に設定。 | 読み込み・実行してよい範囲を制限して被害を緩和                                                                           |
| 入力値検証                                                                                                                                                                                                       | 補助的な対策                                                                                            |
- 禁止されるインラインスクリプト
	- HTML内に直接書かれた`<script>…</script>`
	- onerrorやonclickなどのイベントハンドラ属性
	- `javascript:`で始まるURL
## 2. CSRF（Cross-Site Request Forgery）

- ログイン中の利用者を罠ページに誘導し、利用者のブラウザから正規サイトへ意図しないリクエストを送らせる。
- <mark style="background: #FFB8EBA6;">ブラウザがCookieを自動で付ける</mark>ため、サーバは本人の操作と区別できない。
- 成立条件：①<mark style="background: #FFB8EBA6;">被害者がログイン中</mark>、②<mark style="background: #FFB8EBA6;">Cookieだけでセッション管理</mark>している、③リクエストの中身を攻撃者が予測できる。

### 2.1 流れ（R6春問3：トークンと利用者の紐付けがない）

![[05_R6春問3_図4.png]]

1. <mark style="background: #FFB8EBA6;">攻撃者が自分のアカウントでログインし、自分の csrf_token を取得</mark>する（他人のトークンを盗む必要はない）。
2. その<mark style="background: #FFB8EBA6;">トークンと書き換え内容を hidden に入れた自動送信フォーム</mark>を、罠サイトに置く。
3. 詐欺メールなどで被害者を罠サイトに誘導する。
4. <mark style="background: #FFB8EBA6;">被害者のブラウザが被害者のCookie付きで `POST /shop/editmember` を送り</mark>、被害者の会員情報が変更される。

- 診断結果の読み方：値が空・パラメータなしはエラー（存在チェックはある）が、別アカウントのトークンが通る（<mark style="background: #FFB8EBA6;">セッションとトークンの紐付けを検証していない</mark>）。
- <mark style="background: #FFB8EBA6;">攻撃者が自分で送っても、被害者のCookieは付かない。</mark>「被害者のブラウザから送らせる」ことが要点。

### 2.2 対策（解答の型）

| 対策                                                     | 要点                                                                                                                                                                                           |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="background: #FFB8EBA6;">CSRFトークン</mark>   | <mark style="background: #FFB8EBA6;">推測困難な値をセッションと紐付け</mark>て発行し、フォームのhiddenに埋め込む。サーバで<mark style="background: #FFB8EBA6;">受信時にセッション側の値と一致するか確認</mark>する。存在チェックだけ、「どこかで発行されたトークンか」の確認だけでは不十分 |
| <mark style="background: #FFB8EBA6;">SameSite属性</mark> | <mark style="background: #FFB8EBA6;">Strict：他サイトからは送らない。Lax：他サイトからのトップレベル遷移のGETだけ送る（POST、iframe、画像読み込みでは送らない）。</mark>None：常に送る（Secure必須）                                                     |
| 再認証、Origin／Refererの確認                                  | 重要な操作の前                                                                                                                                                                                      |

- 「どのような処理を行う機能か、具体的に」と問われたら、処理の中身を書く：トークンに関して<mark style="background: #FFB8EBA6;">「推測困難なトークンの発行、フォームを含むページのHTML（hidden）への埋め込み、セッションとの紐付け、受信時の検証」</mark>。

## 3. SSRF（Server-Side Request Forgery）

- <mark style="background: #FFB8EBA6;">サーバが外部から指定されたURLへアクセスする機能を悪用</mark>し、サーバ自身に攻撃者の意図した宛先へリクエストを送らせる。
- 狙われる機能：<mark style="background: #FFB8EBA6;">URL指定での画像やコンテンツの取得</mark>、Webhook、URLプレビュー、PDF生成など。
- 狙われる宛先：<mark style="background: #FFB8EBA6;">内部の管理画面</mark>、<mark style="background: #FFB8EBA6;">内部API</mark>、localhost、クラウドのメタデータサービス（IMDS）。
- 代表例：AWSのIMDS（169.254.169.254）からIAMロールの一時認証情報を盗む手口（2019年のCapital One事件）。
  - IMDSv1：GETだけで取得できる。
  - IMDSv2：PUTでトークンを取得し、ヘッダに付けてアクセスする。GETしか送れない単純なSSRFへの対策になる。
- <mark style="background: #FFB8EBA6;"> 複数回のリクエストは、その都度攻撃者が送る。curlなどで脆弱なサーバを直接たたくイメージ。</mark>

### 3.1 流れ（R6春問3：pageパラメータで取得先を指定できる）

![[05_R6春問3_図7.png]]

- 前提：WebサーバYは、pageの値で指定されたサイトPのページを、<mark style="background: #FFB8EBA6;">利用者のWebブラウザの代わりに</mark>取得して表示する。pageの値を検証していない。CMSの管理画面は「VPN接続した管理者PCかVPC内から」に制限されているが、SSRFでは<mark style="background: #FFB8EBA6;">リクエストがVPC内のWebサーバYから出る</mark>ため、この制限をすり抜ける。
- 方法F（GET固定）：管理画面の表示（GET）はできるが、<mark style="background: #FFB8EBA6;">ログインはPOSTのみ許可なのでできない。</mark>
- 方式1（IMDSv1相当）：<mark style="background: #FFB8EBA6;">pageにIMDSのクレデンシャル情報を返すURLを指定するだけ</mark>で、クレデンシャルが応答に返る。
- 方法G（<mark style="background: #FFB8EBA6;">メソッドとヘッダを任意に指定できる</mark>）では、方式2（IMDSv2相当）も次の手順で突破される。
  1. トークンを発行するURLにPUTを送り、トークンを入手する。
  2. クレデンシャル情報を返すURLに、トークンをリクエストヘッダに入れてアクセスする。

### 3.2 対策（解答の型）

| 対策                                                   | 要点                                                                                                                                     |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="background: #FFB8EBA6;">宛先の許可リスト</mark> | <mark style="background: #FFB8EBA6;">スキームとホストを固定。</mark>本問では「pageの値がサイトPのURL以外ならエラーにする」。拒否リスト方式は、IPアドレスの別表記、リダイレクト、DNSリバインディングで回避されやすい |
| 内部アドレス宛ての拒否                                          | <mark style="background: #FFB8EBA6;">プライベート、ループバック、リンクローカルのアドレスを拒否</mark>する。判定は名前解決後のIPアドレスで行う                                         |
| ネットワーク側の制限                                           | サーバからの送信先を制限する（<mark style="background: #FFB8EBA6;">egress制御</mark>）                                                                   |
| メタデータの保護                                             | IMDSv2の強制、クレデンシャルの最小権限化                                                                                                                |
- egress制御は<mark style="background: #FFB8EBA6;">ファイアウォール、クラウドのセキュリティグループの送信ルール、送信用プロキシ</mark>などを使用する。R6春問3でZ社が提案した「クラウドW上のネットワークでのアクセス制御の設定」は、この考え方にあたる。
- <mark style="background: #FFB8EBA6;">セッションIDの変更などは無関係で不正解。</mark>

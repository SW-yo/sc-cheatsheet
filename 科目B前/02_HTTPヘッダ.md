# HTTPヘッダ あんちょこ（見直し版）
## 0. メッセージの形
### 0.0 HTTPの基本
- <mark style="background: #FFB8EBA6;">ブラウザ→サーバはHTTPリクエスト、サーバ→ブラウザはHTTPレスポンス</mark>
- 構造：<mark style="background: #FFB8EBA6;">「リクエスト/ステータスライン」と「ヘッダ」と「ボディ」</mark>
- <mark style="background: #FFB8EBA6;">ヘッダとボディの間は空行</mark>
- 仕様で決まっている部分：「メソッド␣パス␣バージョン」「バージョン␣コード␣理由」の形、「名前: 値」、空行でヘッダが終わること、各ヘッダ名と属性名（RFC 9110／9112、Cookieは RFC 6265）。

### 0.1 ヘッダの基本
- 構造：「<mark style="background: #FFB8EBA6;">ヘッダ名: 値</mark>」（全ヘッダ共通。コロンの右全体が値）
- 値の中身（区切り記号を含む）はヘッダごとに定義される
- 値の中身は、<mark style="background: #FFB8EBA6;">単独のもの（`nosniff`、`no-store`、`Secure`）</mark>と、<mark style="background: #FFB8EBA6;">「指定=値」の形のもの（`max-age=0`、`SameSite=Lax`）</mark>がある
- 同名ヘッダを複数行にできるのはリスト型（Cache-Control等）とSet-Cookieだけ。Set-Cookieは1行に結合できない
- 値の書き方

| 型            | 値の中身（区切り）                                                                                     | 例                                                                                                                                                                                |
| ------------ | --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ① 単一         | 1個だけ                                                                                          | `Host: www.example.jp`<br>`X-Content-Type-Options: nosniff`                                                                                                                      |
| ② 指定を「,」で並べる | 指定（ディレクティブ）は単独か「指定=値」                                                                         | `Cache-Control: no-store, max-age=0`                                                                                                                                             |
| ③ 指定を「;」で並べる | 同上                                                                                            | `Content-Type: text/html; charset=utf-8`<br>`Strict-Transport-Security: max-age=31536000; includeSubDomains`<br>`Content-Security-Policy: default-src 'self'; script-src 'self'` |
| ④ cookie     | Set-Cookie：「cookie名=cookieの値」の後ろに「;」で属性を付ける。属性は単独か「属性=値」<br>Cookie：「cookie名=cookieの値」を「;」で並べる | `Set-Cookie: SID=abc123; Secure; HttpOnly; SameSite=Lax`<br>`Cookie: SID=abc123; lang=ja`                                                                                        |

- 仕様で決まっている部分：各ヘッダ名、ヘッダごとの区切り記号、charset・no-store・max-age・includeSubDomains・default-src・script-src・'self'・nosniff・Secure・HttpOnly・SameSite、application/x-www-form-urlencoded のボディ形式（パラメータ名=値を&で連結）

### 0.2 HTTPリクエスト
-  リクエストラインの構造：メソッド␣リクエストターゲット（パス）␣HTTPバージョン 
```
POST /login HTTP/1.1                                    ← リクエストライン
Host: www.example.jp                                    ← ここから空行までがヘッダ
Content-Type: application/x-www-form-urlencoded
Cookie: SID=abc123
                                                        ← 空行（CRLFのみ）
userid=taro&password=xxxx                               ← ボディ
```

### 0.3 HTTPレスポンス
- ステータスラインの構造：HTTPバージョン␣ステータスコード␣理由フレーズ 
```
HTTP/1.1 302 Found                                      ← ステータスライン
Location: https://www.example.jp/mypage
Set-Cookie: SID=def456; Secure; HttpOnly; SameSite=Lax
```

## 1. 混ざりやすい組を先に覚える

#### 1.1 試験問題の注意点
- 呼び方
	- 「ヘッダ名」はコロンの左、「値」はコロンの右全体
	- 「値」は「何の値か」を付けて書く（cookieの値、max-ageの値、パラメータpageの値）
	- <mark style="background: #FFB8EBA6;">「指定」はこのノート内の呼び方（仕様ではディレクティブ他）。答案では `no-store` などの文字列そのものを書く。</mark>詳細は「 6. ヘッダの値の形」
	- 表記は問題文に合わせる（R6春は「ヘッダ」、R6秋は「ヘッダー」）
-  ヘッダの設問は<mark style="background: #FFB8EBA6;">「HTTPヘッダー名」に対して「設定した場合の主な効果」を記述させる形で出題された</mark>（R6秋 問4 設問1(5)：Strict-Transport-Security、X-Content-Type-Options）

| 組                                     | 違い（一言）                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Host と SNI                            | <mark style="background: #FFB8EBA6;">Host はHTTPの要素</mark>（TLSの内側）。<mark style="background: #FFB8EBA6;">SNI はTLSの要素</mark>（ClientHelloに平文で載る）。CDNで比較する。ドメインフロンティングではこの違いを元に別サイトへのリダイレクトを誘発する。                                                                                                                                                                                                                                    |
| Origin と Referer                      | <mark style="background: #FFB8EBA6;">Origin はスキーム・ホスト・ポートだけ</mark>。Referer はパスやクエリまで含む（フルURL）。ただしReferrer-Policyによる制限があり<mark style="background: #FFB8EBA6;">クロスオリジンの場合はスキーム・ホスト・ポートのみ、https→httpの場合はRefererを送らない</mark>のみ）。CSRF対策の検証にはどちらも使える                                                                                                                                                                                  |
| no-cache と no-store                   | <mark style="background: #FFB8EBA6;">no-cache は「保存してよいが使う前に確認」</mark>。no-store は「保存するな」。個人情報のページは no-store                                                                                                                                                                                                                                                                                                                      |
| Secure と HSTS                         | <mark style="background: #FFB8EBA6;">Secure は「このCookieをHTTPSのときだけ送る」</mark>。<mark style="background: #FFB8EBA6;">HSTS は「このサイトには以後HTTPSでしか接続しない」</mark>                                                                                                                                                                                                                                                                          |
| HttpOnly と CSP                        | <mark style="background: #FFB8EBA6;">HttpOnly は「スクリプトからCookieを読めない」（XSSは起きる）</mark>。<mark style="background: #FFB8EBA6;">CSP は「スクリプトの実行や読み込み元を制限」（XSSの被害を緩和）</mark>                                                                                                                                                                                                                                                             |
| SameSite と CORS                       | <mark style="background: #FFB8EBA6;">SameSite は「他サイト発のリクエストにCookieを付けるか」</mark>。<mark style="background: #FFB8EBA6;">CORS は「他オリジンの応答をスクリプトに読ませるか」</mark>。CORS（Cross-Origin Resource Sharing）は同一オリジンポリシー（SOP）に対し異なるオリジンからのアクセスを許可する<mark style="background: #FFB8EBA6;">オリジン間リソース共有</marc>の仕組み。サーバがレスポンスヘッダのAccess-Control-Allow-Origin（どのドメインからのアクセスを許可するかを指定）やAccess-Control-Allow-Credentials（trueだとクロスオリジンでcookieを維持）等で指定する。 |
| X-Frame-Options と frame-ancestors     | どちらもクリックジャッキング対策。frame-ancestors はCSPのディレクティブで、許可するサイトを細かく指定できる                                                                                                                                                                                                                                                                                                                                                                 |
| Content-Type と X-Content-Type-Options | <mark style="background: #FFB8EBA6;">Content-Type でMIMEタイプを宣言</mark>し、 <mark style="background: #FFB8EBA6;">X-Content-Type-Optionsのnosniff でその宣言どおりにしか解釈させない</mark>。nonsniffを指定しない場合、ブラウザがリソースの内容から異なるMIMEタイプで解釈する「MIMEスニッフィング」が起こり得る                                                                                                                                                                                           |
| Authorization と WWW-Authenticate      | WWW-Authenticate は401応答でサーバが方式を示す。Authorization はクライアントが資格情報を送る                                                                                                                                                                                                                                                                                                                                                                 |
| 401 と 403                             | <mark style="background: #FFB8EBA6;">401 は「認証されていない（資格情報を出して）」</mark>。<mark style="background: #FFB8EBA6;">403 は「認証済みでも権限がない」</mark>                                                                                                                                                                                                                                                                                            |

## 2. 主要ヘッダ（例・何が起きるか・解答の型）

### 2.1 リクエスト側

| ヘッダ             | 例                                                 | 何が起きるか                                                                                                                                                        | 解答の型                                                                                                                 |
| --------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Host            | `Host: www.example.jp`                            | 仮想ホスト（同一IPアドレスの1台のWebサーバで複数のWebサイトを動かす仕組み）の振り分けに使う。<mark style="background: #FFB8EBA6;">値をそのままURL生成に使うと、パスワード再設定メールのリンクを攻撃者ドメインに向けられる（Hostヘッダインジェクション）</marc> | <mark style="background: #FFB8EBA6;">「URLの生成にHostヘッダの値を使わず、設定値のドメインを使う」</mark>                                       |
| Cookie          | `Cookie: SID=abc123`                              | <mark style="background: #FFB8EBA6;">ブラウザが自動で付ける。CSRFが成立する理由</marc>                                                                                           | 「ブラウザがCookieを自動で送信するため、サーバは利用者の意図した操作か区別できない」                                                                        |
| Authorization   | `Authorization: Bearer eyJ...`                    | <mark style="background: #FFB8EBA6;">Basic はIDとパスワードをbase64にしただけ。Bearer はトークンを持つ者が誰でも使える</marc>                                                               | 「Basicは復号できるので、TLSで保護する」<br>「Bearerのトークンを<mark style="background: #FFB8EBA6;">PKCE</marc>で認可コードとトークンを交換する際の安全性を確保する」 |
| Origin          | `Origin: https://evil.example`                    | <mark style="background: #FFB8EBA6;">どこのページから送られたか（オリジン）。偽装はブラウザ経由ではできない</marc>                                                                               | 「Originヘッダの値が自サイトのオリジンと一致するか確認する」                                                                                    |
| Referer         | `Referer: https://www.example.jp/reset?token=xyz` | URLに入れた秘密（トークン、セッションID）が外部サイトに漏れる                                                                                                                             | 「秘密情報をURLに含めない」<mark style="background: #FFB8EBA6;">「Referrer-Policyで送信範囲を制限する」</mark>                               |
| X-Forwarded-For | `X-Forwarded-For: 198.51.100.7`                   | プロキシ経由の元のIP。クライアントが自由に書ける                                                                                                                                     | 「偽装可能なので、アクセス制御やログの送信元判定に単独で使わない」                                                                                    |

### 2.2 レスポンス側

| ヘッダ                              | 例                                                                | 何が起きるか                                                                                                                        | 解答の型                                                                                                                                  |
| -------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Location                         | `Location: https://evil.example/`                                | 遷移先をパラメータで受け取ると、正規サイトのURLから攻撃者サイトへ飛ばせる（オープンリダイレクト）                                                                            | 「遷移先を自サイト内の許可リストに限定する」                                                                                                                |
| Set-Cookie                       | ※「3. Set-Cookieの属性」参照                                            | Cookieの保存と送信条件の指示                                                                                                             | 「3. Set-Cookieの属性」の属性ごとの型                                                                                                             |
| Content-Type                     | `Content-Type: text/html; charset=UTF-8`                         | charset がないと別の文字コードとして解釈され、XSSにつながることがある                                                                                      | 「Content-Typeにcharsetを明示する」                                                                                                           |
| X-Content-Type-Options           | `X-Content-Type-Options: nosniff`                                | <mark style="background: #FFB8EBA6;">nonsniffでブラウザが中身から種類を推測しない（MIMEスニッフィング）。画像としてアップロードしたファイルをスクリプトやHTMLとして実行させる攻撃を防ぐ</marc> | 「Content-Typeで指定したMIMEタイプ以外として解釈させない」                                                                                                 |
| Content-Disposition              | `Content-Disposition: attachment; filename="a.pdf"`              | 表示せずダウンロードさせる。アップロードファイルをブラウザ内で開かせない                                                                                          | 「ファイルをインライン表示させずダウンロードさせる」                                                                                                            |
| Strict-Transport-Security        | `Strict-Transport-Security: max-age=31536000; includeSubDomains` | <mark style="background: #FFB8EBA6;">以後、max-ageの間はHTTPで接続しない。</marc>初回のHTTP接続を狙うSSLストリッピングを防ぐ                                 | 「HTTPでのアクセスをブラウザがHTTPSに置き換える」                                                                                                         |
| Content-Security-Policy          | `Content-Security-Policy: script-src 'self'`                     | <mark style="background: #FFB8EBA6;">'self'で許可した読み込み元以外のスクリプトを実行しない。'unsafe-inline'やnonce、ハッシュ値指定をしないとインラインスクリプトも止まる</marc>   | 「外部サイトやインラインのスクリプトの実行を禁止し、XSSの被害を緩和する」<br><mark style="background: #FFB8EBA6;">「default-src 'self'は同一オリジンの外部スクリプトファイルしか実行できない」</marc> |
| X-Frame-Options                  | `X-Frame-Options: DENY`                                          | <mark style="background: #FFB8EBA6;">DENYで他サイトのiframeに表示させない</marc>                                                           | 「フレーム内表示を禁止し、クリックジャッキングを防ぐ」                                                                                                           |
| Cache-Control                    | `Cache-Control: no-store`                                        | <mark style="background: #FFB8EBA6;">no-storeで情報をブラウザや共有キャッシュに残さない</marc>                                                     | 「個人情報を含む応答がキャッシュから漏えいしない」                                                                                                             |
| Access-Control-Allow-Origin      | `Access-Control-Allow-Origin: https://www.example.jp`            | <mark style="background: #FFB8EBA6;">指定オリジンのスクリプトに応答の読み取りを許す。</mark>要求のOriginをそのまま返す設定だと、どのサイトからでも読める                         | 「許可するオリジンを固定のリストと照合して返す」                                                                                                              |
| Access-Control-Allow-Credentials | `Access-Control-Allow-Credentials: true`                         | <mark style="background: #FFB8EBA6;">Cookie付きの要求の応答を読ませる。Allow-Origin の「\*」とは併用できない</mark>                                    | 同上                                                                                                                                    |
| WWW-Authenticate                 | `WWW-Authenticate: Bearer error="invalid_token"`                 | 401応答で方式やエラーを示す                                                                                                               | ―                                                                                                                                     |

- 仕様で決まっている部分：各ヘッダ名、「nosniff」「DENY」「SAMEORIGIN」「no-store」「max-age」「includeSubDomains」「'self'」「Bearer」「Basic」「attachment」「charset」などの値・ディレクティブ名（RFC 9110、6797、7034、Fetch Standard、CSP Level 3、RFC 6750）。

### 2.3 CSPの値（頻出のものだけ）

| 値                                                           | 意味                                        |
| ----------------------------------------------------------- | ----------------------------------------- |
| <mark style="background: #FFB8EBA6;">'self'</marc>          | 自オリジンのみ許可                                 |
| 'none'                                                      | すべて禁止                                     |
| <mark style="background: #FFB8EBA6;">'unsafe-inline'</marc> | インラインのスクリプトを許可（XSS対策が大きく弱まる）              |
| 'unsafe-eval'                                               | eval() などで文字列をコードとして実行するのを許可              |
| 'nonce-ランダム値'                                               | 同じnonce属性をもつscript要素だけ実行を許可。値はレスポンスごとに変える |
| 'sha256-ハッシュ値'                                              | 中身のハッシュが一致するインラインスクリプトだけ許可                |

- 注意：<mark style="background: #FFB8EBA6;">script-src 'self' のとき、攻撃者が自サイト内にファイルをアップロードできると、そのファイルは「自オリジン」なので読み込まれる</mark>（R7秋問1の論点）。<mark style="background: #FFB8EBA6;">nosniff とアップロードファイルのContent-Typeの固定が併せて必要になる。</marc>

## 3. Set-Cookieの属性

```
Set-Cookie: SID=a1b2c3; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=3600
```

| 属性                                   | 付けないと何が起きるか                                                                                        | 解答の型                               |
| ------------------------------------ | -------------------------------------------------------------------------------------------------- | ---------------------------------- |
| Secure                               | HTTP通信のときにCookieが平文で送られ、盗聴される                                                                      | 「HTTPS通信のときだけCookieを送信させる」         |
| HttpOnly                             | XSSでスクリプト（JavaScriptのdocument.cookie）からセッションIDを読まれる                                                | 「スクリプトからCookieを参照させない」（XSS自体は防げない） |
| SameSite=Strict                      | 他サイトからのリクエストには常に付けない                                                                               | 「他サイト経由のリクエストにCookieを付けず、CSRFを防ぐ」  |
| SameSite=Lax                         | <mark style="background: #FFB8EBA6;">他サイトからのリンク遷移（トップレベルのGET）には付ける。POST・iframe・画像読み込みには付けない</marc> | 「他サイトからのPOSTにCookieを付けない」          |
| SameSite=None                        | 常に付ける（Secure必須）                                                                                    | ―                                  |
| Domain                               | 指定するとサブドメインにも送られる。脆弱なサブドメインから盗まれる                                                                  | 「Domain属性を指定しない（発行ホストだけに送る）」       |
| Path                                 | 指定パス配下だけに送るが、セキュリティ境界にはならない                                                                        | ―                                  |
| Max-Age / Expires                    | どちらもないとブラウザ終了で消える。両方あればMax-Age優先                                                                   | ―                                  |
| cookie名の接頭辞 \_\_Host-（最初はアンダースコアが2つ） | Secure必須・Domain禁止・Path=/ のときだけ保存される                                                                | 「サブドメインからのCookie上書き（セッション固定）を防ぐ」   |

- 仕様で決まっている部分：属性名、Strict/Lax/None、\_\_Host- と \_\_Secure- の条件（RFC 6265、改訂中の RFC 6265bis）。
- 実装に依存する部分：SameSite未指定のCookieをLaxとして扱う挙動はChrome系ブラウザの実装（元のPDFの「主要ブラウザは未指定時Lax扱い」はこの範囲に限られる）。
- <mark style="background: #FFB8EBA6;">サイトの同一性（SameSiteの「サイト」）は登録可能ドメイン単位（例：a.example.jp と b.example.jp は同一サイト）。オリジン（スキーム・ホスト・ポート）より広い。</marc>

## 4. メソッド（狙われ方）
- <mark style="background: #FFB8EBA6;">プリフライト</mark>：ブラウザが本番のHTTPリクエストを送る前に、サーバーへ「この通信をしても安全ですか？」と事前確認を行う仕組み

| メソッド         | 安全  | 冪等  | 狙われ方と対策                                                                                                                                  |
| ------------ | --- | --- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| GET          | ○   | ○   | <mark style="background: #FFB8EBA6;">更新処理をGETで受けると、リンクや画像を踏ませるだけでCSRFが成立。クエリはログや Referer に残る</marc>                                      |
| POST         | ×   | ×   | <mark style="background: #FFB8EBA6;">CSRFの主な対象。Content-Type が form-urlencoded、multipart/form-data、text/plain ならプリフライトなしで他サイトから送れる</marc> |
| PUT / DELETE | ×   | ○   | <mark style="background: #FFB8EBA6;">不要なら無効化。DELETEはIDの書き換えで他人のデータを消せる（IDOR）→所有者を毎回確認</marc>                                             |
| PATCH        | ×   | ×   | 送られた項目をそのまま反映すると role などまで変えられる（マスアサインメント）→更新可能な項目を許可リストで限定                                                                              |
| OPTIONS      | ○   | ○   | CORSのプリフライト。Originの反射とCredentials: true の組合せが危険                                                                                          |
| TRACE        | ○   | ○   | 応答にCookieやAuthorizationが含まれる（XST）→サーバで無効化                                                                                                |
| CONNECT      | ×   | ×   | 接続先を制限しないと踏み台になる→接続先ポートを443などに限定                                                                                                         |

## 5. Referrer-Policy の値
- Refererはスキーム、ホスト、ポートに加えてパス、クエリも送信するが、Referer-Policyにより制限できる。
- 標準はstrict-origin-when-cross-origin。
- CSRF対策として最近はスキーム、ホスト、ポートだけを送るOriginヘッダに移行してきている（仕様としてMUST、プライバシー上の問題が少ない）

| 値                                                                           | 外部サイトに送るもの                               |
| --------------------------------------------------------------------------- | ---------------------------------------- |
| no-referrer                                                                 | 何も送らない                                   |
| origin                                                                      | オリジンだけ                                   |
| same-origin                                                                 | 外部には送らない（同一オリジンには完全なURL）                 |
| <mark style="background: #FFB8EBA6;">strict-origin-when-cross-origin</mark> | 外部にはオリジンだけ。HTTPS→HTTPでは送らない（多くのブラウザの既定値） |
| unsafe-url                                                                  | 常に完全なURL                                 |
|                                                                             |                                          |

## 6. ヘッダの値の形（名前を問われたときの呼び方）

| 形式 | 例 | 名前の呼び方 |
|---|---|---|
| 単一の値 | `Host: www.example.jp` | ― |
| パラメータ | `Content-Type: text/html; charset=UTF-8` | charset は「パラメータ」 |
| ディレクティブ | `Cache-Control: private, max-age=0`（区切り「,」）、HSTS・CSP（区切り「;」、CSPは名前と値を空白で区切る） | max-age、script-src は「ディレクティブ」 |
| 属性 | `Set-Cookie: SID=…; Secure; HttpOnly` | Secure、HttpOnly は「属性」 |
| 認証パラメータ | `WWW-Authenticate: Basic realm="Admin"` | realm は「認証パラメータ」 |

- 試験で名前を問われたら仕様上の名前で答える。Cookie名のように仕様で決まっていない名前は、問題文の図表の名前で答える。

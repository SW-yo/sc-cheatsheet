## 概要

```
Host: www.example.com
Content-Type: text/html; charset=UTF-8
```

概要：HTTP共通の規則（RFC 9110）が定めているのは「フィールド名: 値」の形までである。値の内部構造は、フィールドごとの仕様で決まる。

- Host：単一の値をそのまま書くフィールドである。
- Content-Type：値の中に「名前=値」を含むフィールドである。この「名前」の呼び方はフィールドの仕様ごとに異なり、パラメータ名、ディレクティブ名、属性名、認証パラメータ名がある。これらはまとめて説明するための言い方で、仕様上の総称ではない。
- どれも「名前（=値）」の形をとる点は共通している。値を持たず、名前だけを書くもの（フラグ）もある。
- 試験で名前を問われた場合は、仕様上の名前で答える。ただし、Cookie名のように仕様で決まっていない名前は、問題文の図表に書かれた名前で答える。
- 任意の部分：「www.example.com」

### 1. パラメータ（Content-Type、Content-Disposition など）

```
Content-Type: text/html; charset=UTF-8
Content-Type: multipart/form-data; boundary=----XyZ123
Content-Disposition: attachment; filename="report.pdf"
```

概要：主となる値の後ろに、「;」で区切って「パラメータ名=パラメータ値」を付ける形式である。

- charset：文字コードを示すパラメータ名である。指定がないと、ブラウザが文字コードを推測する。その結果、攻撃者が仕込んだバイト列が別の文字コード（UTF-7など）として解釈され、スクリプトとして実行されるおそれがある（XSS）。
- boundary：multipartの各パートを区切る文字列を示すパラメータ名である。ファイルアップロードで使われる。
- attachment：Content-Disposition の主となる値で、パラメータではない。ブラウザで表示させずにダウンロードさせる。filename は保存名を示すパラメータ名である。
- 仕様で決まっている部分：「text/html」「multipart/form-data」「attachment」「charset」「boundary」「filename」
- 任意の部分：「----XyZ123」「report.pdf」

### 2. ディレクティブ（Cache-Control、HSTS、CSP など）

```
Cache-Control: no-store
Cache-Control: private, max-age=0, must-revalidate
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com
```

概要：「ディレクティブ名」または「ディレクティブ名=値」を並べる形式である。区切り記号は、Cache-Control が「,」、HSTS と CSP が「;」である。CSP だけは、名前と値を「=」ではなく空白で区切る。

- no-store：キャッシュへの保存を禁止する。no-cache は「保存は可、ただし利用前に再検証が必須」という意味である。個人情報を含むページに no-cache を指定しても、ブラウザのキャッシュには残る点が問題になる。
- private：共有キャッシュ（プロキシ）への保存を禁止する。指定しないと、ある利用者のページがプロキシに保存され、別の利用者に返されるおそれがある。
- HSTS の max-age：その期間（秒）はHTTPSでの接続を強制する。HSTSがないと、初回のHTTP接続を中間者に狙われ、HTTPSに切り替わらないまま通信される（SSLストリッピング）。includeSubDomains は、この強制をサブドメインにも適用する。
- CSP の script-src：スクリプトの読み込み元を限定する。XSSで外部スクリプトを読み込ませる攻撃を防ぐ。
- 仕様で決まっている部分：各ディレクティブ名、「'self'」
- 任意の部分：「0」「31536000」「cdn.example.com」

### 3. 属性（Set-Cookie）

```
Set-Cookie: SID=a1b2c3; Secure; HttpOnly; SameSite=Lax; Path=/; Max-Age=3600
Cookie: SID=a1b2c3; theme=dark
```

概要：先頭に「Cookie名=値」を置き、その後ろに「;」で区切って「属性名（=属性値）」を並べる形式である。Cookie名には仕様上の規定がなく、サーバーが自由に決める。属性が付くのはレスポンスのSet-Cookieだけで、リクエストのCookieヘッダーは「Cookie名=値」の組を送るだけである。

- SID=a1b2c3：Cookie名が「SID」、値が「a1b2c3」である。値にはセッションIDが入る想定である。
- Secure：HTTPS通信のときだけCookieを送信させる。付けないと、HTTP通信のときに平文で送られ、盗聴される。
- HttpOnly：JavaScriptからCookieを読めなくする。付けないと、XSSでセッションIDを盗まれる。
- SameSite：クロスサイトでの送信を制限する。Strict は一切送らない。Lax はリンクでの遷移（GET）のときだけ送り、他サイトからのPOSTでは送らない。None は常に送る（Secure の併用が必須）。None や未対策の状態はCSRFの原因になる。
- Domain：指定するとサブドメインにも送信される。そのため、脆弱なサブドメインからCookieを盗まれるおそれがある。指定しなければ、発行したホストにだけ送られる。
- 仕様で決まっている部分：各属性名、SameSite の属性値（Strict／Lax／None）
- 任意の部分：「SID」「a1b2c3」「theme」「dark」「3600」

### 4. 認証パラメータ（WWW-Authenticate、Authorization）

```
WWW-Authenticate: Basic realm="Admin Area"
Authorization: Basic dXNlcjpwYXNz
Authorization: Bearer eyJhbGciOi...
```

概要：認証方式名の後ろに、「認証パラメータ名=値」または資格情報を付ける形式である。

- realm：サーバーが認証を要求するときの認証パラメータ名で、保護領域の名前を示す。
- Basic：「ID:パスワード」をbase64でエンコードして送る。エンコードは誰でも元に戻せるため、HTTPSがないと盗聴でそのまま漏れる。
- Bearer：アクセストークンを送る。トークンを持っていれば誰でも使えるため、漏えいするとそのまま不正利用される。
- 仕様で決まっている部分：「Basic」「Bearer」「realm」
- 任意の部分：「Admin Area」、エンコード値（「dXNlcjpwYXNz」は「user:pass」をエンコードした例）、トークン文字列

## OAuth 2.0（RFC 6749）認可コードグラント

### 5. 認可要求

```
GET /authorize?response_type=code&client_id=abcd1234&redirect_uri=https://△△△.com/callback HTTP/1.1
```

（R5春Ⅱ問2の「送信されるデータ」の表 p を元にした例）

概要：クライアントが、ブラウザ経由で認可サーバに認可コードを要求する通信である。ここでいうパラメータは、リクエスト行に含まれるURLのクエリ文字列の「名前=値」（「&」区切り）であり、1〜4のヘッダーフィールドのパラメータ・ディレクティブ・属性とは別物である。ヘッダーフィールド側の名前がHTTP関連の各仕様（RFC 9110、RFC 6265など）で定められているのに対し、ここでのパラメータ名は、OAuthの仕様であるRFC 6749で定められている。

- response_type=code：認可コードを要求する指定で、必須（REQUIRED）である。
- client_id：登録時に発行されたクライアントIDで、必須である。
- redirect_uri：認可後にブラウザを戻す先である。仕様上は任意（OPTIONAL）だが、複数登録している場合や未登録の場合は必須になる。指定された場合、認可サーバは登録済みの値と照合する。照合が甘いと、認可コードが攻撃者のサイトに送られてしまう。
- state：上の例には含まれていないパラメータで、指定が推奨されている。ないと何が問題になるかは8で説明する。
- 仕様で決まっている部分：パラメータ名、値「code」、「&」「=」による形式
- 任意の部分：「/authorize」（エンドポイントのパスは仕様で決まっていない）。「abcd1234」「△△△.com/callback」はR5春Ⅱ問2で設定された値である。

### 6. トークン要求

```
POST /oauth/token HTTP/1.1

grant_type=authorization_code&code=5810f68ad195469d85f59a6d06e51e90&redirect_uri=https://△△△.com/callback
```

（R5春Ⅱ問2の「送信されるデータ」の表8の本文を元にした例）

概要：クライアントが、認可コードとアクセストークンの交換を認可サーバに直接要求する通信である。パラメータは本文に、クエリ文字列と同じ形式で記述する。

- grant_type=authorization_code：認可コードを送りトークンと交換する指定で、必須である。
- code：認可サーバが発行した認可コードで、必須である。
- redirect_uri：認可要求に含めていた場合は必須で、認可要求と同一の値でなければならない。認可サーバがこれを照合することで、別のリダイレクト先で発行されたコードの流用を防ぐ。
- 仕様で決まっている部分：パラメータ名、値「authorization_code」
- 任意の部分：「/oauth/token」、認可コードの値（R5春Ⅱ問2で設定された値）

### 7. クライアント認証

```
Authorization: Basic YWJjZDEyMzQ6UEBzc3dvcmQ=
```

（R5春Ⅱ問2の「送信されるデータ」の表8のヘッダーを元にした例）

概要：トークン要求の際に、クライアント自身を認可サーバに認証させる仕組みである。4のBasic認証の形式で、「クライアントID:クライアントシークレット」を送る。

- クライアントは認可サーバに事前登録し、クライアントIDの発行を受ける。シークレットは、機密クライアントの場合に発行される。
- クライアントIDは「クライアントが提供した登録情報を表す一意の文字列」と定義されている。「不透明（opaque）」とは規定されていない。仕様本文で opaque と書かれているのは、state やアクセストークンである。
- クライアントIDは秘密ではないため、単独でクライアント認証に使ってはならない。
- クライアントシークレットは、クライアント認証用の秘密の値である。
- 認可サーバは、シークレットを発行したクライアントについて、Basic認証に対応しなければならない。
- 例の値をデコードすると「abcd1234:P@ssword」となり、シークレットが読める。HTTPSで保護しないと、シークレットが漏れてクライアントになりすまされる。
- 仕様で決まっている部分：「Basic」、「ID:シークレット」をbase64化する形式
- 任意の部分：シークレットの値、エンコード値（R5春Ⅱ問2で設定された値）

### 8. state（CSRF対策）

```
GET /authorize?response_type=code&client_id=abcd1234&redirect_uri=https://△△△.com/callback&state=Xk29fQ7a HTTP/1.1

HTTP/1.1 302 Found
Location: https://△△△.com/callback?code=5810f68ad195469d85f59a6d06e51e90&state=Xk29fQ7a
```

（R5春Ⅱ問2の表8に、説明用として state を追加した例）

概要：クライアントが、推測できない値を利用者のセッションに紐付けて認可要求に含める。認可サーバはその値をそのまま返し、クライアントは返ってきた値がセッションの値と一致するかを確認する。

- stateがない場合の攻撃は、次の流れで行われる。攻撃者は、まず自分のアカウントで認可要求を行い、自分用の認可コードを取得する。このとき、コールバックへのアクセスは止めておく。
- 次に、そのコードを含むコールバックURL（「https://△△△.com/callback?code=攻撃者のコード」）を罠ページに仕込み、被害者のブラウザにアクセスさせる。
- クライアントは、被害者のセッションで攻撃者のコードをトークンと交換してしまう。その結果、被害者のセッションが攻撃者のアカウントに紐付く。
- 被害として、被害者が登録した情報が攻撃者のアカウント側に保存される。あるいは、攻撃者のアカウントが被害者のアカウントに連携され、攻撃者が被害者としてログインできるようになる。
- stateがあれば、攻撃者は被害者のセッションに紐付いた値を知ることができない。そのため、一致確認で偽のコールバックが拒否される。PKCE（RFC 7636）も、同様の防御になる。
- 仕様で決まっている部分：パラメータ名「state」、値をそのまま返すという動作、302とLocationによるリダイレクト
- 任意の部分：「Xk29fQ7a」、「/callback」

### 9. 認可応答（認可コードの受け渡し）

```
HTTP/1.1 302 Found
Location: https://△△△.com/callback?code=5810f68ad195469d85f59a6d06e51e90

GET /callback?code=5810f68ad195469d85f59a6d06e51e90 HTTP/1.1
Host: △△△.com
```

（R5春Ⅱ問2の図3の(5)(6)を、表8の値を使って示した例）

概要：認可サーバが、利用者の同意後に認可コードをリダイレクトで返す。前半が認可サーバからブラウザへの302（(5)）、後半がブラウザからクライアントへのGET（(6)）である。コードはブラウザ経由のURLのクエリ文字列で渡る。

- code：認可サーバが発行した認可コードで、必須である。
- state：認可要求に含まれていた場合は必須で、同じ値を返す（上の例は、認可要求にstateがない場合）。8を参照。
  ```
  <!--仮にstateの要求があった場合-->
  HTTP/1.1 302 Found
  Location: https://△△△.com/callback
  code=5810f68ad195469d85f59a6d06e51e90&state=Xk29fQ7a
  
  GET /callback
  code=5810f68ad195469d85f59a6d06e51e90&state=Xk29fQ7a HTTP/1.1
  Host: △△△.com
  ```
- 認可コードは短寿命でなければならない。有効期間は最大10分が推奨されている。
- 認可コードは一度だけ使える。再使用された場合、認可サーバは要求を拒否しなければならず、そのコードで発行済みのトークンを取り消すことが推奨されている（トークン要求側のエラーはerror=invalid_grant）。
- 利用者が同意しなかった場合は、codeの代わりにerror=access_deniedなどがクエリ文字列で返る。
- 仕様で決まっている部分：パラメータ名「code」「state」、クエリ文字列による返却、認可コードの短寿命と一度きりの規定
- 任意の部分：認可コードの値（R5春Ⅱ問2で設定された値）、「/callback」

### 10. アクセストークン応答

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA"
}
```

（R5春Ⅱ問2の図3の(8)を、RFC 6749の応答例を元に示した例。表8には含まれない）

概要：認可サーバが、トークン要求（6）に対して、アクセストークンをJSONで直接クライアントに返す。ブラウザは介在しない。

- access_token：発行されたアクセストークンで、必須である。
- token_type：トークンの種類で、必須である。大文字小文字は区別されない。
- expires_in：有効期間（秒）で、推奨されている。省略する場合は別の方法で有効期間を伝える。
- refresh_token：新しいアクセストークンを得るためのトークンで、任意である。
- scope：要求と同じなら任意で、異なる場合は必須である。
- Cache-ControlとPragmaは、トークンを含む応答に必ず付ける。キャッシュからの漏えいを防ぐためである。
- 認可コードの再使用などは、HTTP 400とerror=invalid_grantで返る。
- 仕様で決まっている部分：パラメータ名、JSON形式、Cache-Control: no-store、Pragma: no-cache
- 任意の部分：トークンの値と形式（トークンの中身の形式は仕様で決まっていない）、expires_inの値、refresh_tokenの有無
- 補足：token_typeの値「Bearer」の使い方は、RFC 6749ではなくRFC 6750で定義されている（11を参照）。

### B. OAuth 2.0（RFC 6749）に含まれない

### 11. 記事投稿（リソースサーバへのAPI呼出し）

```
POST /api/posts HTTP/1.1
Host: api.service-t.example
Authorization: Bearer 2YotnFZFEjr1zCsicMWpAA
Content-Type: application/json

{"text": "今日の食事の記事"}
```

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="The access token expired"
```

（R5春Ⅱ問2の図3の(9)(10)。表8には含まれない。パスと本文は説明用に作った例）

概要：クライアントが、アクセストークンを付けてリソースサーバのAPIを呼び出す。RFC 6749はトークンの使い方を定めておらず、Bearerトークンの使い方はRFC 6750が別に定めている。

- Authorization: Bearer：トークンを提示するヘッダーである。トークンを持っている者なら誰でも使えるため、「Bearer（持参人）」と呼ばれる。
- 7のBasicとの違い：Basicはクライアント自身の認証（トークン要求の宛先は認可サーバ）で、Bearerはトークンの提示（宛先はリソースサーバ）である。
- RFC 6750では、トークンをフォーム本文やURLのクエリで送る方法も定義されている。クエリは、ログなどに残るため推奨されていない。
- トークンは必ずTLS（HTTPS）で送る。
- トークンが無効・期限切れの場合、リソースサーバはHTTP 401とWWW-Authenticateヘッダーで返す。
- 仕様で決まっている部分：「Authorization: Bearer ＜トークン＞」の形式、401とWWW-Authenticate、error=invalid_token（いずれもRFC 6750）
- 任意の部分：APIのパス、ホスト名、リクエスト本文、成功時の応答の形式（リソースサーバが決める）

### C. PKCE（RFC 7636）

### 12. PKCE（認可コード横取り対策）

```
GET /authorize?response_type=code&client_id=abcd1234&redirect_uri=https://△△△.com/callback&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM&code_challenge_method=S256 HTTP/1.1

POST /oauth/token HTTP/1.1
Authorization: Basic YWJjZDEyMzQ6UEBzc3dvcmQ=

grant_type=authorization_code&code=5810f68ad195469d85f59a6d06e51e90&redirect_uri=https://△△△.com/callback&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

（R5春Ⅱ問2の表8のpとqに、PKCEのパラメータを追加した例。値はRFC 7636の付録Bの例）

概要：クライアントが乱数から検証コード（code_verifier）を作り、そのハッシュ値をチャレンジコード（code_challenge）として認可要求（5）で送る。認可サーバはチャレンジコードを認可コードに紐付けて保管し、トークン要求（6）で検証コードを受け取って関係を検証する。

- code_challenge：認可要求に追加する。必須である。
- code_challenge_method：認可要求に追加する。S256かplainで、省略するとplainとみなされる。S256を使えるクライアントは、S256を使わなければならない。
- code_verifier：トークン要求に追加する。必須である。43〜128文字で、使える文字は英数字と「-」「.」「_」「~」である。
- S256の場合：code_challenge = BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))。base64urlは、標準のbase64の「+」を「-」、「/」を「_」に置き換え、末尾の「=」を付けない。
- 検証：認可サーバは、受け取ったcode_verifierから同じ計算をして、保管したcode_challengeと一致するか確認する。不一致ならerror=invalid_grantで拒否する。
- 防御の仕組み：認可コードを横取りした攻撃者は、code_verifierを知らない。code_challengeからcode_verifierはハッシュの一方向性によって逆算できない。
- 秘密を持てない公開クライアント（スマホアプリなど）では、7のBasic認証を使えない。その場合はトークン要求にclient_idを含める。
- 仕様で決まっている部分：パラメータ名「code_challenge」「code_challenge_method」「code_verifier」、値「S256」、ハッシュの計算式
- 任意の部分：検証コードとチャレンジコードの値（RFC 7636の付録Bの例）、client_idなどのその他の値（R5春Ⅱ問2で設定された値）
````
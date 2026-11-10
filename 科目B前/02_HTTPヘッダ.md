# HTTPヘッダ あんちょこ（見直し版）

各項目は「例」「何が起きるか」「解答の型」の順に、具体的な値で書いている。

## 0. メッセージの形

```
POST /login HTTP/1.1                                    ← リクエストライン
Host: www.example.jp                                    ← ここから空行までがヘッダ
Content-Type: application/x-www-form-urlencoded
Cookie: SID=abc123
                                                        ← 空行（CRLFのみ）
userid=taro&password=xxxx                               ← ボディ
```

```
HTTP/1.1 302 Found                                      ← ステータスライン
Location: https://www.example.jp/mypage
Set-Cookie: SID=def456; Secure; HttpOnly; SameSite=Lax
```

- 仕様で決まっている部分：「メソッド␣パス␣バージョン」「バージョン␣コード␣理由」の形、「名前: 値」、空行でヘッダが終わること、各ヘッダ名と属性名（RFC 9110／9112、Cookieは RFC 6265）。
- 任意の部分：「/login」「www.example.jp」「SID」「abc123」「def456」、ボディの項目名。
- 元のPDFでは応答の1行目にも「リクエストライン」と書かれていたが、応答の1行目は「ステータスライン」である。

## 1. 混ざりやすい組を先に覚える

| 組 | 違い（一言） |
|---|---|
| Host と SNI | Host はHTTPの要素（TLSの内側）。SNI はTLSの要素（ClientHelloに平文で載る）。CDNで比較する、ドメインフロンティングで食い違わせる |
| Origin と Referer | Origin はスキーム・ホスト・ポートだけ。Referer はパスやクエリまで含みうる（Referrer-Policyで削れる）。CSRF対策の検証にはどちらも使える |
| no-cache と no-store | no-cache は「保存してよいが使う前に確認」。no-store は「保存するな」。個人情報のページは no-store |
| Secure と HSTS | Secure は「このCookieをHTTPSのときだけ送る」。HSTS は「このサイトには以後HTTPSでしか接続しない」 |
| HttpOnly と CSP | HttpOnly は「スクリプトからCookieを読めない」（XSSは起きる）。CSP は「スクリプトの実行や読み込み元を制限」（XSSの被害を緩和） |
| SameSite と CORS | SameSite は「他サイト発のリクエストにCookieを付けるか」。CORS は「他オリジンの応答をスクリプトに読ませるか」 |
| X-Frame-Options と frame-ancestors | どちらもクリックジャッキング対策。frame-ancestors はCSPのディレクティブで、許可するサイトを細かく指定できる |
| Content-Type と X-Content-Type-Options | Content-Type で種類を宣言し、nosniff でその宣言どおりにしか解釈させない |
| Authorization と WWW-Authenticate | WWW-Authenticate は401応答でサーバが方式を示す。Authorization はクライアントが資格情報を送る |
| 401 と 403 | 401 は「認証されていない（資格情報を出して）」。403 は「認証済みでも権限がない」 |

## 2. 主要ヘッダ（例・何が起きるか・解答の型）

### 2.1 リクエスト側

| ヘッダ | 例 | 何が起きるか | 解答の型 |
|---|---|---|---|
| Host | `Host: www.example.jp` | 仮想ホストの振り分けに使う。値をそのままURL生成に使うと、パスワード再設定メールのリンクを攻撃者ドメインに向けられる（Hostヘッダインジェクション） | 「URLの生成にHostヘッダの値を使わず、設定値のドメインを使う」 |
| Cookie | `Cookie: SID=abc123` | ブラウザが自動で付ける。CSRFが成立する理由 | 「ブラウザがCookieを自動で送信するため、サーバは利用者の意図した操作か区別できない」 |
| Authorization | `Authorization: Bearer eyJ...` | Basic はIDとパスワードをbase64にしただけ。Bearer はトークンを持つ者が誰でも使える | 「base64は復号できるので、TLSで保護する」 |
| Origin | `Origin: https://evil.example` | どこのページから送られたか（オリジン）。偽装はブラウザ経由ではできない | 「Originヘッダの値が自サイトのオリジンと一致するか確認する」 |
| Referer | `Referer: https://www.example.jp/reset?token=xyz` | URLに入れた秘密（トークン、セッションID）が外部サイトに漏れる | 「秘密情報をURLに含めない」「Referrer-Policyで送信範囲を制限する」 |
| X-Forwarded-For | `X-Forwarded-For: 198.51.100.7` | プロキシ経由の元のIP。クライアントが自由に書ける | 「偽装可能なので、アクセス制御やログの送信元判定に単独で使わない」 |

### 2.2 レスポンス側

| ヘッダ | 例 | 何が起きるか | 解答の型 |
|---|---|---|---|
| Location | `Location: https://evil.example/` | 遷移先をパラメータで受け取ると、正規サイトのURLから攻撃者サイトへ飛ばせる（オープンリダイレクト） | 「遷移先を自サイト内の許可リストに限定する」 |
| Set-Cookie | 3章 | Cookieの保存と送信条件の指示 | 3章の属性ごとの型 |
| Content-Type | `Content-Type: text/html; charset=UTF-8` | charset がないと別の文字コードとして解釈され、XSSにつながることがある | 「Content-Typeにcharsetを明示する」 |
| X-Content-Type-Options | `X-Content-Type-Options: nosniff` | ブラウザが中身から種類を推測しない。画像としてアップロードしたファイルをスクリプトやHTMLとして実行させる攻撃を防ぐ | 「Content-Typeで指定したMIMEタイプ以外として解釈させない」 |
| Content-Disposition | `Content-Disposition: attachment; filename="a.pdf"` | 表示せずダウンロードさせる。アップロードファイルをブラウザ内で開かせない | 「ファイルをインライン表示させずダウンロードさせる」 |
| Strict-Transport-Security | `Strict-Transport-Security: max-age=31536000; includeSubDomains` | 以後、max-ageの間はHTTPで接続しない。初回のHTTP接続を狙うSSLストリッピングを防ぐ | 「HTTPでのアクセスをブラウザがHTTPSに置き換える」 |
| Content-Security-Policy | `Content-Security-Policy: script-src 'self'` | 許可した読み込み元以外のスクリプトを実行しない。インラインスクリプトも止まる | 「外部サイトやインラインのスクリプトの実行を禁止し、XSSの被害を緩和する」 |
| X-Frame-Options | `X-Frame-Options: DENY` | 他サイトのiframeに表示させない | 「フレーム内表示を禁止し、クリックジャッキングを防ぐ」 |
| Cache-Control | `Cache-Control: no-store` | ブラウザや共有キャッシュに残さない | 「個人情報を含む応答がキャッシュから漏えいしない」 |
| Access-Control-Allow-Origin | `Access-Control-Allow-Origin: https://www.example.jp` | 指定オリジンのスクリプトに応答の読み取りを許す。要求のOriginをそのまま返す設定だと、どのサイトからでも読める | 「許可するオリジンを固定のリストと照合して返す」 |
| Access-Control-Allow-Credentials | `Access-Control-Allow-Credentials: true` | Cookie付きの要求の応答を読ませる。Allow-Origin の「*」とは併用できない | 同上 |
| WWW-Authenticate | `WWW-Authenticate: Bearer error="invalid_token"` | 401応答で方式やエラーを示す | ― |

- 仕様で決まっている部分：各ヘッダ名、「nosniff」「DENY」「SAMEORIGIN」「no-store」「max-age」「includeSubDomains」「'self'」「Bearer」「Basic」「attachment」「charset」などの値・ディレクティブ名（RFC 9110、6797、7034、Fetch Standard、CSP Level 3、RFC 6750）。
- 任意の部分：ドメイン名（evil.example、www.example.jp）、IPアドレス、秒数「31536000」、ファイル名、トークン値。

### 2.3 CSPの値（頻出のものだけ）

| 値 | 意味 |
|---|---|
| 'self' | 自オリジンのみ許可 |
| 'none' | すべて禁止 |
| 'unsafe-inline' | インラインのスクリプトを許可（XSS対策が大きく弱まる） |
| 'unsafe-eval' | eval() などで文字列をコードとして実行するのを許可 |
| 'nonce-ランダム値' | 同じnonce属性をもつscript要素だけ実行を許可。値はレスポンスごとに変える |
| 'sha256-ハッシュ値' | 中身のハッシュが一致するインラインスクリプトだけ許可 |

- 注意：script-src 'self' のとき、攻撃者が自サイト内にファイルをアップロードできると、そのファイルは「自オリジン」なので読み込まれる（R7秋問1の論点）。nosniff とアップロードファイルのContent-Typeの固定が併せて必要になる。

## 3. Set-Cookieの属性

```
Set-Cookie: SID=a1b2c3; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=3600
```

| 属性 | 付けないと何が起きるか | 解答の型 |
|---|---|---|
| Secure | HTTP通信のときにCookieが平文で送られ、盗聴される | 「HTTPS通信のときだけCookieを送信させる」 |
| HttpOnly | XSSで document.cookie からセッションIDを読まれる | 「スクリプトからCookieを参照させない」（XSS自体は防げない） |
| SameSite=Strict | 他サイトからのリクエストには常に付けない | 「他サイト経由のリクエストにCookieを付けず、CSRFを防ぐ」 |
| SameSite=Lax | 他サイトからのリンク遷移（トップレベルのGET）には付ける。POST・iframe・画像読み込みには付けない | 「他サイトからのPOSTにCookieを付けない」 |
| SameSite=None | 常に付ける（Secure必須） | ― |
| Domain | 指定するとサブドメインにも送られる。脆弱なサブドメインから盗まれる | 「Domain属性を指定しない（発行ホストだけに送る）」 |
| Path | 指定パス配下だけに送るが、セキュリティ境界にはならない | ― |
| Max-Age / Expires | どちらもないとブラウザ終了で消える。両方あればMax-Age優先 | ― |
| 名前の接頭辞 __Host- | Secure必須・Domain禁止・Path=/ のときだけ保存される | 「サブドメインからのCookie上書き（セッション固定）を防ぐ」 |

- 仕様で決まっている部分：属性名、Strict/Lax/None、__Host- と __Secure- の条件（RFC 6265、改訂中の RFC 6265bis）。
- 実装に依存する部分：SameSite未指定のCookieをLaxとして扱う挙動はChrome系ブラウザの実装（元のPDFの「主要ブラウザは未指定時Lax扱い」はこの範囲に限られる）。
- 任意の部分：「SID」「a1b2c3」「3600」。
- サイトの同一性（SameSiteの「サイト」）は登録可能ドメイン単位（例：a.example.jp と b.example.jp は同一サイト）。オリジン（スキーム・ホスト・ポート）より広い。

## 4. メソッド（狙われ方だけ）

| メソッド | 安全 | 冪等 | 狙われ方と対策 |
|---|---|---|---|
| GET | ○ | ○ | 更新処理をGETで受けると、リンクや画像を踏ませるだけでCSRFが成立。クエリはログや Referer に残る |
| POST | × | × | CSRFの主な対象。Content-Type が form-urlencoded、multipart/form-data、text/plain ならプリフライトなしで他サイトから送れる |
| PUT / DELETE | × | ○ | 不要なら無効化。DELETEはIDの書き換えで他人のデータを消せる（IDOR）→所有者を毎回確認 |
| PATCH | × | × | 送られた項目をそのまま反映すると role などまで変えられる（マスアサインメント）→更新可能な項目を許可リストで限定 |
| OPTIONS | ○ | ○ | CORSのプリフライト。Originの反射とCredentials: true の組合せが危険 |
| TRACE | ○ | ○ | 応答にCookieやAuthorizationが含まれる（XST）→サーバで無効化 |
| CONNECT | × | × | 接続先を制限しないと踏み台になる→接続先ポートを443などに限定 |

## 5. Referrer-Policy の値

| 値 | 外部サイトに送るもの |
|---|---|
| no-referrer | 何も送らない |
| origin | オリジンだけ |
| same-origin | 外部には送らない（同一オリジンには完全なURL） |
| strict-origin-when-cross-origin | 外部にはオリジンだけ。HTTPS→HTTPでは送らない（多くのブラウザの既定値） |
| unsafe-url | 常に完全なURL |

## 6. ヘッダの値の形（名前を問われたときの呼び方）

| 形式 | 例 | 名前の呼び方 |
|---|---|---|
| 単一の値 | `Host: www.example.jp` | ― |
| パラメータ | `Content-Type: text/html; charset=UTF-8` | charset は「パラメータ」 |
| ディレクティブ | `Cache-Control: private, max-age=0`（区切り「,」）、HSTS・CSP（区切り「;」、CSPは名前と値を空白で区切る） | max-age、script-src は「ディレクティブ」 |
| 属性 | `Set-Cookie: SID=…; Secure; HttpOnly` | Secure、HttpOnly は「属性」 |
| 認証パラメータ | `WWW-Authenticate: Basic realm="Admin"` | realm は「認証パラメータ」 |

- 試験で名前を問われたら仕様上の名前で答える。Cookie名のように仕様で決まっていない名前は、問題文の図表の名前で答える。

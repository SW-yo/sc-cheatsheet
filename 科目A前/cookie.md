### 1. cookieの設定
| 属性            | 概要                                         | 指定例                                   | 試験上のポイント                                                                                     |
| ------------- | ------------------------------------------ | ------------------------------------- | -------------------------------------------------------------------------------------------- |
| Secure        | HTTPS通信のときだけcookieを送信する                    | Secure                                | 平文HTTPでの盗聴によるcookie漏えいを防ぐ                                                                    |
| HttpOnly      | JavaScript（document.cookie）からcookieを読めなくする | HttpOnly                              | XSSによるセッションIDの窃取を緩和する。XSS自体は防げない                                                             |
| SameSite      | 別サイトからのリクエストにcookieを付けるかを制御する              | SameSite=Strict / Lax / None          | CSRF対策。Strictは別サイトからは常に送らない、Laxはトップレベルの遷移（GETのリンク等）のみ送る、Noneは常に送る（Secure必須）。主要ブラウザは未指定時Lax扱い |
| Domain        | cookieを送信するドメインを指定する                       | Domain=example.com                    | 省略すると発行したホストのみに送られる。指定するとサブドメインにも送られ、範囲が広がる                                                  |
| Path          | cookieを送信するURLのパスを指定する                     | Path=/app                             | 指定パス配下にのみ送信。セキュリティ上の境界にはならない                                                                 |
| Expires       | cookieの有効期限を日時で指定する                        | Expires=Wed, 21 Oct 2026 07:28:00 GMT | Expires・Max-Ageとも省略するとブラウザ終了で消えるセッションcookieになる                                               |
| Max-Age       | cookieの有効期間を秒数で指定する                        | Max-Age=3600                          | Expiresと両方あればMax-Ageが優先。0以下で即削除                                                              |
| __Secure- 接頭辞 | 名前の先頭に付けると、Secure属性付きかつHTTPSからの設定のみ受け付ける   | __Secure-sid=...                      | cookie名で安全な設定を強制する                                                                           |
| __Host- 接頭辞   | Secure必須、Domain指定禁止、Path=/ 必須の条件でのみ受け付ける   | __Host-sid=...                        | サブドメインからのcookie上書き（セッション固定攻撃など）を防ぐ                                                           |

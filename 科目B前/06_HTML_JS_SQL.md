# HTML・JavaScript・SQL（見分け方とお試しサンプル）

## 0. 問題選択のための見分け方

R4〜R7で解答例を確認した範囲では、<mark style="background: #FFB8EBA6;">サーバ側のコードはJava（R4春Ⅱ問1、R5春Ⅰ問1、R6春問4）</mark>、<mark style="background: #FFB8EBA6;">ブラウザ側はHTMLとECMAScript（R5秋問1、R6秋問3、R7秋問1）</mark>だった。C#はこの範囲では出ていないが、見分け方は載せておく。

| 手掛かり                                                | HTML                         | JavaScript（ECMAScript）                              | Java                                                               | C#                                                                   |
| --------------------------------------------------- | ---------------------------- | --------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------- |
| <mark style="background: #FFB8EBA6;">一目で分かる形</mark> | `<div class="x">…</div>` のタグ | `function f(a) {`、`const x = …`、`(a) => …`          | `public class X {`、`public static void main(String[] args)`        | `namespace X {`、`public class X {`、`static void Main(string[] args)` |
| 先頭の読込み                                              | `<!DOCTYPE html>`            | `import … from '…'`（ないことも多い）                        | `import java.util.*;`                                              | `using System;`                                                      |
| 変数の宣言                                               | ―                            | 型を書かない（`let`、`const`、`var`）                         | 型を先に書く（`String s = …;`、`int n = …;`）                               | 型を先に書く（`string s = …;`、`var` も使う）                                    |
| 画面・出力                                               | ―                            | `document.getElementById`、`innerHTML`、`console.log` | `System.out.println`、サーブレットでは `out.println`、`request.getParameter` | `Console.WriteLine`、ASP.NETでは `Response.Write`                       |
| 比較・文字列                                              | ―                            | `===`、テンプレート文字列 `` `…${x}` ``                       | `equals()` で文字列比較、`String`（大文字）                                    | `==` で文字列比較できる、`string`（小文字）                                         |
| 例外・型                                                | ―                            | `try { } catch (e) { }`（型なし）                        | `catch (IOException e)`、`throws`、`finally`                         | `catch (Exception e)`、`using (…) { }`                                |
| DB接続                                                | ―                            | `fetch('/api/…')` でサーバに頼む                           | `PreparedStatement`、`setString(1, …)`                              | `SqlCommand`、`Parameters.AddWithValue("@id", …)`                     |

- 判断の手順：①`<タグ>` が中心ならHTML。②`document.`、`addEventListener`、`fetch`、`XMLHttpRequest` があればブラウザ側JavaScript。③変数の前に型名があり、`public class` で囲まれていればJavaかC#。④`System.out` ならJava、`Console.` や `using System;` ならC#。
- 試験での使い方：<mark style="background: #FFB8EBA6;">HTML・JavaScriptの読解が中心の問題は選択候補に入れる。Javaのコード読解が設問の中心（例外処理、スレッド、ファイル操作）なら避ける。Javaでも「この行で何が出力されるか」程度なら、問題文の説明で追えることがある。
</mark>
## 1. お試しサンプルの使い方

- HTMLとJavaScriptは、各サンプルを「.html」で保存してブラウザで開く。alert が出たら「スクリプトが実行された」ことを意味する。
- SQLは、SQLite（オンラインのSQLite実行環境やDB Browser for SQLiteなど）にそのまま貼り付けて実行する。他のDB製品でもほぼ同じように動く。
- いずれも自分の手元だけで試すこと。他人のサイトに対して行うと不正アクセスになりうる。

## 2. HTMLサンプル

### HTML-1：属性値からの抜け出し（R4春Ⅱ問1の形）

```html
<!DOCTYPE html>
<html lang="ja">
<head><meta charset="UTF-8"><title>HTML-1</title></head>
<body>
<p>1. 普通の値（何も起きない）</p>
<input type="text" value="taro">

<p>2. 値が「"&gt;&lt;img src=1 onerror=alert('HTML-1')&gt;」のとき（エスケープなし）</p>
<input type="text" value=""><img src=1 onerror=alert('HTML-1')>">

<p>3. 同じ値を「&quot; &lt; &gt;」にエスケープしたとき</p>
<input type="text" value="&quot;&gt;&lt;img src=1 onerror=alert('HTML-1')&gt;">
</body>
</html>
```

解説
- 2では、<mark style="background: #FFB8EBA6;">最初の「"」で value の値が終わり、「>」で input タグが閉じる。その後ろの <img …> が独立したタグになり、画像が読めないので onerror が動いて alert が出る。末尾の「">」は画面に文字として出る。</mark>
- 3では、「&quot;」は値の中の文字「"」として扱われるので、属性値は閉じない。入力欄に攻撃文字列がそのまま文字として表示されるだけで、alert は出ない。
- 試験での読み方：<mark style="background: #FFB8EBA6;">出力先が属性値の中なら、まず「"」で抜け出せるかを見る。対策は「出力時に &quot; &#39; &lt; &gt; &amp; を文字参照に変換する」。</mark>
- 仕様で決まっている部分：属性値は次の「"」で終わる、文字参照 `&quot;` `&lt;` `&gt;`、onerror の動作（HTML Living Standard）。
- 任意の部分：「taro」、alert の文字列「HTML-1」、src の値「1」。

出力エスケープサンプル（Java）
```Java
static String escapeHtml(String s) {
    StringBuilder sb = new StringBuilder();
    for (char c : s.toCharArray()) {
        switch (c) {
            case '&':  sb.append("&amp;");  break;
            case '<':  sb.append("&lt;");   break;
            case '>':  sb.append("&gt;");   break;
            case '"':  sb.append("&quot;"); break;
            case '\'': sb.append("&#39;");  break;
            default:   sb.append(c);
        }
    }
    return sb.toString();
}
```

### HTML-2：script要素の中への出力と「</script>」

```html
<!DOCTYPE html>
<html lang="ja">
<head><meta charset="UTF-8"><title>HTML-2</title></head>
<body>
<p>1. JavaScriptの文字列の中に「&lt;/script&gt;」を含む値をそのまま出力した場合</p>
<script>
  var title = "</script><img src=1 onerror=alert('HTML-2')>";
  document.write("<p>titleの中身（文字として表示）：</p>");
  var p = document.createElement("p");
  p.textContent = title;
  document.body.appendChild(p);
</script>

<p>2. 「&lt;」を \u003c に置き換えて出力した場合</p>
<script>
  var title2 = "\u003c/script\u003e\u003cimg src=1 onerror=alert('HTML-2')\u003e";
  document.write("<p>title2の中身（文字として表示）：</p>");
  var p = document.createElement("p");
  p.textContent = title2;
  document.body.appendChild(p);
</script>
</body>
</html>
```

解説
- 1では、<mark style="background: #FFB8EBA6;">ブラウザはJavaScriptの文字列かどうかを考えず、最初に現れた &lt; /script &gt; でscript要素を終える。そのため script の中身は途中で切れた文（構文エラー）になり、続く &lt; img … &gt; がHTMLのタグとして解釈されて alert が出る。</mark>\<image\>タグ以降は文字としてそのまま画面に表示される。
- 2では、<mark style="background: #FFB8EBA6;">\u003c はJavaScriptの文字列の中では「<」と同じ意味だが、HTMLの解析では「<」に見えない。そのため script 要素は途中で終わらず、値は文字として表示されるだけになる。</mark>
- 試験での読み方：<mark style="background: #FFB8EBA6;">出力先がscript要素の中なら、JavaScriptの文字列としてのエスケープ（「"」「\」）だけでは足りず、「&lt; /script &gt;」でも抜けられる</mark>（R5秋問1のメモにある「script data」の扱い）。
- 仕様で決まっている部分：script要素の中身は `</script>` が現れるまで続く（HTML Living Standard）、`\u003c` の意味（ECMAScript）。

## 3. JavaScriptサンプル

### JS-1：innerHTML と textContent

```html
<!DOCTYPE html>
<html lang="ja">
<head><meta charset="UTF-8"><title>JS-1</title></head>
<body>
<input id="msg" size="50" value="<img src=1 onerror=alert('JS-1')>">
<button id="danger">innerHTMLで表示</button>
<button id="safe">textContentで表示</button>
<div id="out" style="border:1px solid #888; min-height:2em; margin-top:8px"></div>
<p>※入力欄を <code>&lt;script&gt;alert('JS-1')&lt;/script&gt;</code> に書き換えて押すと、「innerHTMLで表示」でもalert は出ない。</br>
元の入力内容：<code>&lt;img src=1 onerror=alert('JS-1')&gt;</code></p>
<script>
  const msg = document.getElementById("msg");
  const out = document.getElementById("out");
  document.getElementById("danger").addEventListener("click", () => {
    out.innerHTML = msg.value;      // 文字列をHTMLとして解釈する
  });
  document.getElementById("safe").addEventListener("click", () => {
    out.textContent = msg.value;    // 文字列を文字として扱う
  });
</script>
</body>
</html>
```

試し方と解説
- ＜input>で取り込んだ文字列の扱いに関して、innerHTMLとtextContentそれぞれでの動作の違い。
- 「innerHTMLで表示」を押すと alert が出る。入力欄を `<script>alert('JS-1')</script>` に書き換えて押すと、alert は出ない。<mark style="background: #FFB8EBA6;">innerHTMLで差し込んだscript要素は実行されない決まりがあるため。</mark>ただし onerror などのイベント属性は実行されるので、innerHTMLは危険なまま。
- 「textContentで表示」を押すと、どちらの値も文字として表示されるだけ。
- これがDOM-based XSSの形：サーバを通らず、ブラウザ側のJavaScriptがDOMに書き込むときに起きる。対策の<mark style="background: #FFB8EBA6;">解答の型は「innerHTMLではなくtextContentを使う」「HTMLとして解釈させない方法で出力する」。</mark>
- script タグが実行されるのは、主に次の2つの場合。
	- ページを読み込むとき、HTML の中に最初から書かれている。
	- JavaScript で `document.createElement("script")` を使って作り、ページに追加する。
- innerHTML で文字列から作られた script タグは、このどちらにも当たらない。ページ上に要素としては存在するが、中身は実行されないまま置かれるだけ。そのため alert は出ない。
- 仕様で決まっている部分：innerHTML で挿入した script は実行されない、textContent はHTMLとして解釈しない（HTML Living Standard、DOM Standard）。

### JS-2：URL検証の不備（R7春問3の形）

```html
<!DOCTYPE html>
<html lang="ja">
<head><meta charset="UTF-8"><title>JS-2</title></head>
<body>
<pre id="out"></pre>
<script>
  const ALLOWED_HOST = "www.example.jp";

  // 不十分な検証1：先頭が一致するか
  function startsWithCheck(u) { return u.startsWith("https://www.example.jp"); }
  // 不十分な検証2：文字列が含まれるか
  function includesCheck(u) { return u.includes("www.example.jp"); }
  // 正しい検証：URLとして解析し、スキームとホスト名を完全一致で比べる
  function urlCheck(u) {
    try {
      const x = new URL(u);
      return x.protocol === "https:" && x.hostname === ALLOWED_HOST;
    } catch (e) { return false; }
  }

  const tests = [
    "https://www.example.jp/mypage",
    "https://www.example.jp.evil.example/",
    "https://www.example.jp@evil.example/",
    "https://evil.example/?r=www.example.jp",
    "javascript:alert(1)//www.example.jp"
  ];
  let s = "URL | startsWith | includes | URL解析\n";
  for (const t of tests) {
    s += `${t} | ${startsWithCheck(t)} | ${includesCheck(t)} | ${urlCheck(t)}\n`;
  }
  document.getElementById("out").textContent = s;
</script>
</body>
</html>
```

結果

| URL | startsWith | includes | URL解析 |
|---|---|---|---|
| https://www.example.jp/mypage | true | true | true |
| https://www.example.jp.evil.example/ | true | true | false |
| https://www.example.jp@evil.example/ | true | true | false |
| https://evil.example/?r=www.example.jp | false | true | false |
| javascript:alert(1)//www.example.jp | false | true | false |
※true：通る、false：通らない

解説
- 2行目：ホスト名は「www.example.jp.evil.example」で、攻撃者が取得できるドメイン。<mark style="background: #FFB8EBA6;">先頭一致では通ってしまう。</mark>
- 3行目：<mark style="background: #FFB8EBA6;">「@」より前はユーザ情報として扱われ、実際の接続先は evil.example になる。</mark>
- 5行目：<mark style="background: #FFB8EBA6;">javascript: スキームはリンク先にするとスクリプトとして実行される。</mark>
- 解答の型：<mark style="background: #FFB8EBA6;">「URLを解析してスキームとホスト名を取り出し、許可したものと完全一致で比較する」。</mark>
- 仕様で決まっている部分：「@」の前がユーザ情報になる、URLの解析方法（URL Standard、RFC 3986）、javascript: スキーム。

## 4. SQLサンプル

アプリが「入力値を文字列連結してSQLを組み立てる」と、入力値がSQLの一部として解釈される。以下は、組み立てた結果のSQLを直接実行して、何が起きるかを確かめるもの。

### SQL-1：文字列の中に入る値（ログイン処理の形）

```sql
-- すでにテーブルがあれば消す
DROP TABLE IF EXISTS users;

-- サンプルSQL
CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, password TEXT, role TEXT);
INSERT INTO users VALUES (1,'taro','pass-taro','user'),(2,'hanako','pass-hanako','user'),(3,'admin','pass-admin','admin');

-- 1. 正しい入力（name=taro、password=pass-taro）→ 1行
SELECT id, name, role FROM users WHERE name = 'taro' AND password = 'pass-taro';

-- 2. パスワード欄に「' OR '1'='1」→ 全行
SELECT id, name, role FROM users WHERE name = 'taro' AND password = '' OR '1'='1';

-- 3. 名前欄に「admin'--」→ パスワードを見ずにadminの行
SELECT id, name, role FROM users WHERE name = 'admin'-- ' AND password = 'xxx'
;

-- 4. 「'」を「''」に重ねてエスケープした場合 → 0行（文字列として比較されるだけ）
SELECT id, name, role FROM users WHERE name = 'taro' AND password = ''' OR ''1''=''1';
```

結果
+----+------+------+
| id | name | role |
+----+------+------+
|  1 | taro | user |
+----+------+------+
+----+--------+-------+
| id | name   | role  |
+----+--------+-------+
|  1 | taro   | user  |
|  2 | hanako | user  |
|  3 | admin  | admin |
+----+--------+-------+
+----+-------+-------+
| id | name  | role  |
+----+-------+-------+
|  3 | admin | admin |
+----+-------+-------+

解説
- 2：<mark style="background: #FFB8EBA6;">入力の先頭の「'」で文字列が閉じ、`OR '1'='1'` という常に真の条件が加わる。ANDはORより先に評価されるので、`(name='taro' AND password='') OR ('1'='1')` となり、全行が返る。</mark>
- 3：<mark style="background: #FFB8EBA6;">「--」から行末まではコメント。パスワードの条件が消えるadminでSELECTを実行。行末の「;」もコメントに入るので、次の行に「;」を置いている。</mark>
- 4：<mark style="background: #FFB8EBA6;">SQLの文字列の中で「'」を表すには「''」と重ねる。値が文字列のまま比較されるだけになる。</mark>この場合はpasswordが「' OR '1'='1」の行の抽出。
- 実行結果：1は taro の1行、2は3行すべて、3は admin の1行、4は0行。
- 仕様で決まっている部分：<mark style="background: #FFB8EBA6;">「'」で囲む文字列リテラル、「''」による「'」の表し方、「--」のコメント、ANDがORより優先</mark>（SQL標準）。

### SQL-2：数値として入る値とUNION（商品検索の形）

```sql
-- すでにテーブルがあれば消す
DROP TABLE IF EXISTS users;

-- サンプルSQL
CREATE TABLE products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);
CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, password TEXT);
INSERT INTO products VALUES (1,'pen',100),(2,'note',200);
INSERT INTO users VALUES (1,'taro','pass-taro'),(2,'admin','pass-admin');

-- 1. 正しい入力（id=1）→ pen
SELECT name, price FROM products WHERE id = 1;

-- 2. id欄に「1 OR 1=1」→ 全商品（「'」を使っていない）
SELECT name, price FROM products WHERE id = 1 OR 1=1;

-- 3. id欄に「0 UNION SELECT name, password FROM users」→ 利用者の名前とパスワード
SELECT name, price FROM products WHERE id = 0 UNION SELECT name, password FROM users;
```

結果
+------+-------+
| name | price |
+------+-------+
| pen  |   100 |
+------+-------+
+------+-------+
| name | price |
+------+-------+
| pen  |   100 |
| note |   200 |
+------+-------+
+-------+------------+
| name  | price      |
+-------+------------+
| taro  | pass-taro  |
| admin | pass-admin |
+-------+------------+

解説
- 2：<mark style="background: #FFB8EBA6;">数値の場所には「'」がないので、「'」をエスケープするだけの対策は効かない。</mark>
- 3：<mark style="background: #FFB8EBA6;">UNIONは2つのSELECTの結果をつなげる。列の数をそろえれば、別テーブルの中身を商品一覧の画面に表示させられる。</mark>
- 実行結果（SQLiteで確認済み）：1は pen、2は pen と note、3は admin と taro の名前とパスワード。
- 対策の解答の型：<mark style="background: #FFB8EBA6;">「プレースホルダ（バインド機構）を使ってSQL文を組み立てる」。プレースホルダでは、SQL文の形（`WHERE id = ?`）を先に確定させ、値は後から渡すため、値がSQLの構文として解釈されない。Javaでは `PreparedStatement` の `setInt(1, id)`、`setString(1, name)` がこれに当たる。</mark>
- 仕様で決まっている部分：UNIONの規則（列数が同じ）、`?` のプレースホルダ（JDBCなどのDB接続の仕様）。

プレースホルダーサンプル（Java）
```Java
PreparedStatement ps = conn.prepareStatement(
    "SELECT name, price FROM products WHERE id = ?");
ps.setInt(1, Integer.parseInt(input));   // 値は後から渡す
ResultSet rs = ps.executeQuery();
```

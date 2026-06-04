# GAS ウェブアプリ開発の注意点

## 相対URLが動かない（iframe問題）

GAS ウェブアプリの HTML は `*.googleusercontent.com` の iframe 内で配信される。そのため、相対URL（例: `?page=privacy`）はスクリプト URL ではなく iframe の URL に解決されてしまい、**リンクが壊れる**。

### 解決方法

`ScriptApp.getService().getUrl()` でスクリプト URL を取得し、テンプレート変数として渡す。リンクには `target="_top"` を付けて iframe を抜ける。

```javascript
// main.gs の doGet
var template = HtmlService.createTemplateFromFile('index');
template.scriptUrl = ScriptApp.getService().getUrl();
return template.evaluate().setTitle('ページタイトル');
```

```html
<!-- HTML内でのリンク -->
<a href="<?= scriptUrl ?>?page=privacy" target="_top">プライバシーポリシー</a>
```

**注意:**
- `createHtmlOutputFromFile` ではテンプレートタグが使えない。`createTemplateFromFile` + `.evaluate()` を使うこと
- 外部 URL（Stripe 等）はそのまま使えるが、`target="_blank"` を推奨

## GAS HTML での画像表示

GAS ウェブアプリではローカルファイルを配信できない。画像を表示するには以下の方法がある:

| 方法 | メリット | デメリット |
|------|---------|-----------|
| **base64 データURI埋め込み** | 外部依存なし、確実に表示 | HTMLファイルが大きくなる（〜200KB程度まで推奨） |
| **Google Drive 公開リンク** | ファイルサイズ制限なし | 共有設定が必要、URLが変わる可能性 |
| **外部URL** | シンプル | 外部サービスに依存 |

### base64 埋め込みの例

```python
# Python でPNG → JPEG圧縮 → base64変換
from PIL import Image
import io, base64

img = Image.open('image.png').convert('RGB')
img.thumbnail((1200, 1200), Image.LANCZOS)
buf = io.BytesIO()
img.save(buf, format='JPEG', quality=80)
b64 = base64.b64encode(buf.getvalue()).decode()
# → HTML: <img src="data:image/jpeg;base64,{b64}">
```

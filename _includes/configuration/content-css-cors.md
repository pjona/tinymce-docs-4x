
## content_css_cors
When setting the `content_css_cors` setting to `true` the editor will add a `crossorigin="anonymous"` attribute to the link tags that the StyleSheetLoader uses when loading the `content_css`, allowing you to host the `content_css` on a different server than the editor itself.

**Type:** `Boolean`

**Default Value:** `false`

##### Example

```js
// File: http://domain.mine/mysite/index.html

tinyMCE.init({
  selector: 'textarea',  // change this value according to your HTML
  content_css : 'http://www.somewhere.example/mycontent.css',
  content_css_cors: true
});
```

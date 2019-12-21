## WebView和Native交互方式

### Webview to Java

1. 拦截prompt提示框方式
2. 拦截url跳转方式
3. Android原生交互方式
4. 利用console.log

### Java to Webview

1. WebView.loadUrl("javascript:" + data)
2. WebView.evaluateJavascript(data, valueCallback） //(SDK > KITKAT)
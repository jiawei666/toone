## 9.2

`"browser_action"` //用户点击触发

`"content_scripts"` //指定URL触发

`"background"` //后台

`"permissions"` //可进行跨域的URL

`"options_page"` //用户点击选项触发

`"chrome.runtime.sendMessage('Hello',callback())"` //跨页面，跨域请求

`"chrome.runtime.onMessage.addListener(callback(){
	sendRespones('Hi~~');
})"` //跨页面，跨域接收并返回

## 9.3
`"web_accessible_resources" : ["image/*"]` //统一获取资源，可以在注入中使用

`chrome.extension.egtURL('images/top.png')` //获取资源相对路径 chrome-extension://gladeibgegajeemmnhanbdochpeheimn/images/top.png


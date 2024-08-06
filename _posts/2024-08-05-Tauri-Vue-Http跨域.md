
最近遇到tauri的http跨域的问题。

服务器不能改，因为早就发过去了

将原来的tauri 1.0+axios网上那些办法都是没用的。

需要升级到tauri2.0+tauri http plugin （现在还是测试的版本）

这个只需要本地配置就可以，不需要服务器去改

(https://v2.tauri.app/plugin/http-client/)[https://v2.tauri.app/plugin/http-client/]

是可以跨域的。 但轮到vue的样式全部崩掉（要css3）

所以……就是要配套的用 1.0 + axios就别想了

还有tauri怎么折腾已经忘了……

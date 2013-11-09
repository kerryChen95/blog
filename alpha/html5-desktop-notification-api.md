# HTML5桌面提醒API

[支持程度](http://caniuse.com/#feat=notifications)


只有当浏览器中设置为默认（询问）时，调用请求允许的接口，浏览器才会询问用户；
否则当浏览器中设置为允许/决绝时，调用请求允许的接口，浏览器都不会询问用户。（目前测了win8 chrome30/FF）

请求允许的接口，只有在用户触发的事件的回调中才有效，并且可以传入一个回调函数。
chrome下必须传入这个回调函数

## win8 chrome30

每当浏览器的桌面提醒设置更改时，都会提示用户刷新页面以使更改生效，所以，可以认为在一个页面的生命周期中，浏览器的设置不会改变

最多同时出现3个桌面提醒，有更多的提醒时会放入一个队列中。只有之前的提醒关闭时，队列中后面的提醒才会显示出来，否则一直放在队列中。

可以设置是否带有声音提醒（貌似都没有出声音）。

## win8 FF25

最多显示一个桌面提醒弹窗。即如果创建一个桌面提醒时已经有一个桌面提醒正在显示，会立刻换成新创建的。

## IE9+

http://caniuse.com/notifications

`window.external.msIsSiteMode` method introduction:
http://msdn.microsoft.com/en-us/library/ff976310(v=vs.85).aspx

Also, we canNOT detect if msIsSiteMode method exists, as it is
a method of host object. **In IE check for existing method of host
object returns undefined.** So, we try to run it - if it runs 
successfully - then it is IE9+, if not - an exceptions is thrown.

IE10中只是状态栏中的icon闪烁，没有弹窗

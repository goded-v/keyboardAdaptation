<center><font  size = "6px">H5混合开发软键盘适配方案</font></center>

### 现象：

​	当前端界面的输入框位于页面底部，键盘唤醒时，就会遮挡输入框。此时用户在输入时就不能看到已经输入的内容，造成很不好的用户体验。

### 思路分析：

​	原生键盘的唤醒方式大概分为两种，一种是平铺在页面上，和页面不属于同一层级；另一种是键盘唤醒时将页面向上挤压，使其位于同一层级。这里我们采用第二种方案。当键盘唤醒时，将整个webview向上挤压，页面向上挤压的高度为键盘的高度。此时预想的结果是实现qq微信发送消息的效果。

### 实现方式：

android代码：

```java
cordova.getActivity().runOnUiThread(new Runnable() {
    @Override
    public void run() {
        cordova.getActivity().getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);
        callbackContext.success();
    }
});
```

在这里前端不需要做处理。本以为这个方案是最完美的。可是发现ios11.1和ios11.2不生效。这就很头疼了。为了适配ios，不得不想新的解决思路。

### 改变思路：

​	综合考虑了android和ios的版本问题，这里我们采用不同系统采用不同处理方式的方案。通过前端判断设备类型，在输入框获取焦点的时候，进行下一步处理。android采取以上的实现方式。ios就采取下面要说的这种方式。

### ios实现方式：

​	根据ios的特性，我们采取前端处理的方式来实现。当前端获取到焦点时，将整个body向上推。类型于第一种方式，只不过是前端来处理。

前端代码：

```javascript
var scrollTime;
var timerId;
if(typeof (device)!="undefined"&&device.platform=='iOS'){
    let num = 0;
    scrollTime = setInterval(function(){
       timerId = true;
       if (num < 9) {
       num++;
     } else {
       clearInterval(scrollTime);
       timerId = null;
       document.body.scrollTop = document.body.scrollHeight;
       return;
     }
      document.body.scrollTop = document.body.scrollHeight;
    },100)
}
```

下面来讲一下原理：

在获取焦点时，执行以上代码。因为键盘弹起有一个延迟，我们在这里写了一个定时器，来实现这个过度。就能实现类似于qq微信的效果了。

这里需要注意的是，在失去焦点的时候，我们要清除这个定时器。再执行

```javascript
document.body.scrollTop = document.body.scrollHeight;
```

达到完美过度的效果。当然，想实现顺滑的效果，还是用原生写吧。

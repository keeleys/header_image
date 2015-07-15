title: 网站头像上传剪辑
tags: flash
categories: html
---



##界面预览
![界面预览](https://dn-tianjun.qbox.me/ttianjun_header_upload.png)

<!-- more -->
## 页面使用

1. 引用js

```html
<script src="${ctx}/js/swfobject.js"></script>
```

2. 构建展示的div

```html
<div style="width:750px;height:550px;margin:40px auto 0; padding:10px;border:solid 1px #EFEFEF;">
		<div id="altContent">
			<h1>AvatarUpload</h1>
			<p><a href="http://www.adobe.com/go/getflashplayer">Get Adobe Flash player</a></p>
		</div>
	</div>
```

3. 配置js参数

```html
<script type="text/javascript">
		var headimg ="${session.session_customer.customerImg}";
		if(headimg=="")//判断用户是否有头像 有的话就用 没有用默认头像
			headimg = "${ctx}/images/avatar.png";
		else
			headimg='${upload}/'+headimg;
		var flashvars = {
			js_handler:"jsfun", // 全局的回调函数，提供给flash在后续情况下调用
			swfID:"avatarEdit", // 当前flash文件的id，方便区分多个相同组件
			picSize:"5242880", //选择本机图片允许的最大值单位为字节；比如5M 该值为：5242880
			sourceAvatar:headimg, //默认头像的url地址
			avatarLabel:"头像预览，请注意清晰度", // 右侧头像预览区的label；
			avatarAPI:"${ctx}/customer/customer!saveHead", // 保存头像的接口地址
			//sourcePicAPI:"${ctx}/customer/customer!saveHead", 保存头像原图的接口地址  如果不想使用“保存原图”这个功能，可以不设置该值
			//sourceLabel: 复选框 "是否保存原图"的文案；
			avatarSize:"126,126", //生成的头像尺寸，默认是："180,180|50,50|30,30"; "|" 区分头像尺寸的个数；最多3个尺寸
			avatarSizeLabel:"用户头像" //各个尺寸头像预览的文本简介；默认值："大头像|中头像|小头像";
		};
		var params = {
			menu: "false",
			scale: "noScale",
			allowFullscreen: "true",
			allowScriptAccess: "always",
			bgcolor: "",
			wmode: "transparent" // can cause issues with FP settings & webcam
		};
		var attributes = {
			id:"AvatarUpload"
		};
		swfobject.embedSWF(
			"${ctx}/js/headimg/avatarUpload.swf", //下载下来的swf文件地址
			"altContent", "100%", "100%", "10.0.0", 
			"${ctx}/js/headimg/expressInstall.swf", 
			flashvars, params, attributes);
			
		function jsfun(obj)
		{
			if(obj.type == "avatarSuccess") { // 上传头像成功
				bootbox.alert("头像上传成功",function(){ //bootbox提示插件
					location.reload();
				});
			}
			if(obj.type == "avatarError") { //上传头像失败
				bootbox.alert("头像上传失败");
			}
			console.log(obj);
		}
	</script>
```

## java服务器后端处理

插件文档不是很全 没有提供后端文档,   
通过chrome浏览器抓取网页请求，发现图片保存的时候 请求的数据是用post的`body流发`送的图片的流数据。

![请求数据](https://dn-tianjun.qbox.me/ttianjunQQ截图20150715143501.png)

那么就简单了 通过java把request流转写入成图片,相关伪代码

```java

File file = new File(filePath+NoKit.randomTimeNo()+".jpg");//创建一个File保证将要写入的图片文件 ,filePath 配置的目录
if(!file.getParentFile().exists()) //如果目录不存在 新建目录
	file.mkdirs();
ImageIO.write(ImageIO.read(request.getInputStream()), "jpg", file); //将流文件通过ImageIO写入到文件

Map<String,Object> param = new HashMap<String,Object>();
param.put("code", "200");
param.put("pic", file.getName());
outputJson(param); //返回json到前端 通知前端上传成功

```

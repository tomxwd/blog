---
title: 01_BootStrap_初试
date: 2019-09-14 10:06:25
tags: 
 - HTML
 - BootStrap
categories:
 - BootStrap
---

# 01_BootStrap_初试

## 1. 环境

- BootStrap官网下载css以及js文件
- 依赖于JQuery，所以也去JQuery下载jquery的js文件

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
	</head>
	<body>
	</body>
	<script src="../js/jquery3.4.1.js"></script>
	<script src="../js/bootstrap.min.js"></script>
</html>
```

## 2. 容器

1. 流体容器

   ```html
   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title></title>
   		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
   		<style type="text/css">
   			.container-fluid{
   				background: #3E8F3E;
   			}
   		</style>
   	</head>
   	<body>
   		
   		<div class="container-fluid">
   			流体容器
   		</div>
   		
   	</body>
   	<script src="../js/jquery3.4.1.js"></script>
   	<script src="../js/bootstrap.min.js"></script>
   </html>
   ```

2. 固体容器

   - 阈值
     - 【大屏PC】lg大于等于1200px时，1170px
     - 【中屏PC】md大于等于992px时，970px
     - 【平板】sm大于等于768px时，750px
     - 【移动手机】xs小于768px时，auto

   ```html
   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title></title>
   		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
   		<style type="text/css">
   			.container{
   				background: #3E8F3E;
   			}
   		</style>
   	</head>
   	<body>
   		
   		<div class="container">
   			固体容器
   		</div>
   		
   	</body>
   	<script src="../js/jquery3.4.1.js"></script>
   	<script src="../js/bootstrap.min.js"></script>
   </html>
   ```



## 3. 栅格系统

container容器

row行

col列

实例1：

普通例子

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
		<style type="text/css">
			.container{
				background: #3E8F3E;
			}
			div[class|=col]{
				border:1px solid
			}
		</style>
	</head>
	<body>
		
		<div class="container">
			<div class="row">
				<div class="col-lg-10">10</div>
				<div class="col-lg-2">2</div>
				<div class="col-lg-6">6</div>
				<div class="col-lg-6">6</div>
				<div class="col-lg-4">4</div>
				<div class="col-lg-8">8</div>
			</div>
		</div>
		
	</body>
	<script src="../js/jquery3.4.1.js"></script>
	<script src="../js/bootstrap.min.js"></script>
</html>
```

实例2：

自适应栅格

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
		<style type="text/css">
			.container{
				background: #3E8F3E;
			}
			div[class|=col]{
				border:1px solid
			}
		</style>
	</head>
	<body>
		
		<div class="container">
			<div class="row">
				<div class="col-lg-10 col-md-6 col-sm-2 col-xs-6">10-6</div>
				<div class="col-lg-2 col-md-6 col-sm-10 col-xs-6">2-6</div>
			</div>
		</div>
		
	</body>
	<script src="../js/jquery3.4.1.js"></script>
	<script src="../js/bootstrap.min.js"></script>
</html>
```

实例3：

模仿BootStrap官网写一个

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
		<meta name="viewport" content="width=device-width, initial-scale=1">
		<!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
		<title>栅格实例</title>
		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
	</head>
	<body>
		
		<div class="container">
			<div class="jumbotron">
			  <h1>Hello, world!</h1>
			  <p>...</p>
			  <p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
			</div>
			<div class="row">
			  <div class="col-sm-6 col-md-4 col-lg-3 col-lg-push-9 col-md-push-8 col-sm-push-6">
			    <div class="thumbnail">
			      <img src="https://cdn.jsdelivr.net/npm/@bootcss/www.bootcss.com@0.0.3/dist/img/expo.png" alt="官网偷的图1">
			      <div class="caption">
			        <h3 class="text-center">优站精选</h3>
					<p class="text-center">
						<a href="javascript:;">by @mdo</a>
					</p>
					<p>Bootstrap 优站精选频道收集了众多基于 Bootstrap 构建、设计精美的、有创意的网站。</p>
			      </div>
			    </div>
			  </div>
			  <div class="col-sm-6 col-md-4 col-lg-3 col-lg-push-3 col-md-pull-0 col-sm-pull-6">
			    <div class="thumbnail">
			      <img src="https://cdn.jsdelivr.net/npm/@bootcss/www.bootcss.com@0.0.3/dist/img/webpack.png" alt="官网偷的图2">
			      <div class="caption">
			        <h3 class="text-center">WebPack</h3>
			        <p class="text-center">
			        	<a href="javascript:;">by @mdo</a>
			        </p>
			        <p>Bootstrap 优站精选频道收集了众多基于 Bootstrap 构建、设计精美的、有创意的网站。</p>
			      </div>
			    </div>
			  </div>
			  <div class="col-sm-6 col-md-4 col-lg-3 col-lg-pull-3 col-md-pull-8">
			    <div class="thumbnail">
			      <img src="https://cdn.jsdelivr.net/npm/@bootcss/www.bootcss.com@0.0.3/dist/img/react.png" alt="官网偷的图3">
			      <div class="caption">
			        <h3 class="text-center">React</h3>
			        <p class="text-center">
			        	<a href="javascript:;">by @mdo</a>
			        </p>
			        <p>Bootstrap 优站精选频道收集了众多基于 Bootstrap 构建、设计精美的、有创意的网站。</p>
			      </div>
			    </div>
			  </div>
			  <div class="col-sm-6 col-md-4 col-lg-3 col-lg-pull-9">
			    <div class="thumbnail">
			      <img src="https://cdn.jsdelivr.net/npm/@bootcss/www.bootcss.com@0.0.3/dist/img/typescript.png" alt="官网偷的图4">
			      <div class="caption">
			        <h3 class="text-center">typescript</h3>
			        <p class="text-center">
			        	<a href="javascript:;">by @mdo</a>
			        </p>
			        <p>Bootstrap 优站精选频道收集了众多基于 Bootstrap 构建、设计精美的、有创意的网站。</p>
			      </div>
			    </div>
			  </div>
			</div>
		</div>
		
	</body>
	<script src="../js/jquery3.4.1.js"></script>
	<script src="../js/bootstrap.min.js"></script>
</html>
```



## 4. 响应式工具

在什么时候可看见

- visible-xs
- visible-sm
- visible-md
- visible-lg

在什么时候隐藏

- hidden-xs
- hidden-sm

- hidden-md
- hidden-lg

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
		<meta name="viewport" content="width=device-width, initial-scale=1">
		<!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
		<title>栅格实例</title>
		<link rel="stylesheet" type="text/css" href="../css/bootstrap.min.css"/>
		
		<style type="text/css">
			.jumbotron{
				background: pink;
			}
		</style>
	</head>
	<body>
		
		<div class="container">
			<div class="jumbotron hidden-xs visible-lg">
			  <h1>Hello, world!</h1>
			  <p>...</p>
			  <p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
			</div>
			
		</div>
		
	</body>
	<script src="../js/jquery3.4.1.js"></script>
	<script src="../js/bootstrap.min.js"></script>
</html>
```


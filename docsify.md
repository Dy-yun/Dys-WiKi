# Docsify

> 本站基本参照[官方文档](https://docsify.js.org/#/zh-cn/quickstart)部署

2021/09/12

---

## 部署Docsify

> 基本上就是按照官方文档操作了

- 全局安装docsify-cli工具

	```bash
	npm i docsify-cli -g
	
	ln -s /usr/local/temp/Node.js/bin/docsify /usr/bin/docsify
	```
- 初始化项目，选择自己喜欢的位置，最好就按照官方命令，方便以后操作
    ```bash
    docsify init ./docs
    ```
- 启动服务，就可以开始预览了 [http://localhost:3000](http://localhost:3000/)或者是`ip`:`3000`
     ```bash
    docsify serve docs
    ```
## 配置说明
> 配置集中在index.html文件当中，这里摘取本文档所用到的插件进行分解解释
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="description" content="Description">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
    <link rel="stylesheet" href="//unpkg.com/docsify/lib/themes/vue.css">
    <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/gitalk/dist/gitalk.css">
</head>
<body>
    <div id="app"></div>
    <script>
    //插件的配置都写在这里
        window.$docsify = {
        	//图标
            logo:'',
            
            //整个网站的名字
            name:'Dy\'s Doc',
            
            //右上角github链接地址
            
            repo:'https://github.com/JuemingDy',
            // 导航栏，需要有_navbar.md文件
            loadNavbar: true,
            
            // 封面，需要有_coverpage.md文件
            coverpage: true,
            
            //侧边栏，需要有_sidebar.md文件
            loadSidebar: true,
            
            //开启侧边栏后不再自动生成目录，需要开启此项决定生成多少级
            subMaxLevel: 2,
            
            // 404页   
            notFoundPage:'_404.md',
            
            //代码复制插件
		    copyCode: {
	        buttonText : '点击复制',
	        errorText  : '发生错误',
	        successText: '已复制'
	        },
	        
	        //谷歌统计，需要服务器能链接外网
	        //ga: 'UA-207767065-1',
            
	        //字数统计
	        count:{
	            countable:true,
	            fontsize:'0.9em',
	            color:'rgb(90,90,90)',
	            language:'chinese',
	        },
            
	        //搜索
	        search: 'auto',
	        search: {
	            paths: '/',
	            placeholder: '搜索',
	            noData: '找不到结果',
	            depth: 3,
	        },
            
	        //tab插件
	        tabs: {
	            persist    : true,      // default
	            sync       : true,      // default
	            theme      : 'classic', // default
	            tabComments: true,      // default
	            tabHeadings: true       // default
	        },
            
	        //分页
	        /*
	        pagination: {
	            previousText: '上一篇',
	            nextText: '下一篇',
	            crossChapter: true,
	            crossChapterText: true,
	        },
	        */
            
	        //脚
	        footer: {
	            copy: 'Copyright©2021 ,',
	            auth: 'DyCloud',
	            pre: '<hr>',
	            style:'text-align: center;'
	        },
	
	    }
	</script>
	<!--cdn地址，按自己喜欢的选-->
	<!--<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>-->
	<!--<script src="//cdnjs.cloudflare.com/ajax/libs/docsify/4.11.2/docsify.min.js"></script>-->
	<!--<script src="//unpkg.com/docsify/lib/docsify.min.js"></script>-->
	
	<!-- Docsify，不能少的 -->
	<script src="//unpkg.com/docsify/lib/docsify.min.js"></script>
	
	<!-- emoji -->
	<script src="//unpkg.com/docsify/lib/plugins/emoji.min.js"></script>
	
	<!-- 复制到剪贴板 -->
	<script src="//unpkg.com/docsify-copy-code@"></script>
    
	<!-- 图片缩放 -->
	<script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/zoom-image.min.js"></script>
	
	<!-- 代码高亮 -->
	<script src="//unpkg.com/prismjs/components/prism-bash.min.js"></script>
	<script src="//unpkg.com/prismjs/components/prism-php.min.js"></script>
	
	<!-- 谷歌统计 -->
	<!--<script src="//unpkg.com/docsify/lib/plugins/ga.min.js"></script>-->
    
	<!-- 字数统计 -->
	<script src="//unpkg.com/docsify-count/dist/countable.js"></script>
    
	<!-- 搜索 -->
	<script src="//unpkg.com/docsify/lib/plugins/search.min.js"></script>
	
	<!-- 标签 -->
	<script src="https://cdn.jsdelivr.net/npm/docsify-tabs@1"></script>
    
	<!-- 分页 -->
	<!--<script src="//unpkg.com/docsify-pagination/dist/docsify-pagination.min.js"></script>-->
    
	<!-- 脚 -->
	<script src="//unpkg.com/docsify-footer-enh/dist/docsify-footer-enh.min.js"></script>
    
	<!-- git评论 -->
	<script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/gitalk.min.js"></script>
	<script src="//cdn.jsdelivr.net/npm/gitalk/dist/gitalk.min.js"></script>
	<script>
	    var gitalk = new Gitalk({
	        clientID: '406162e419988e219faf',
	        clientSecret: '77ae615c44b9ab0c08b7e0ae097d0d11ae7bd9f0',
	        repo: 'Dy-s-Docsify',
	        owner: 'JuemingDy',
	        admin: ['JuemingDy'],
	        id: location.pathname,    
	
	        distractionFreeMode: true
	    })
	</script>
    
	<!-- 离线模式 -->
	<script>
	    if (typeof navigator.serviceWorker !== 'undefined') {
	        navigator.serviceWorker.register('sw.js')
	    }
</script>
</body>

</html>

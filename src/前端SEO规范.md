## 1.0前端

### 1.1页面切片

页面切片，需注意文字切成文字，图片LOGO单独切，spider无法识别图片内容，并且更喜欢文字性的页面；

### 1.2页面代码

#### 1.2.1链接a标签

页面的链接URL要用a标签（下载链接例外），页面文字有超链要用a标签，将文字以及链接URL包住一起，

举例如下：

<a href="http://a.9game.cn/hymnsjhz/" type="btn" data-statis="text:btn_xzh_news_footer_game_name">

<div class="game-icon"> <img src="http://image.9game.cn/2018/1/4/19063369.jpg" alt="虎牙迷你世界盒子">

</div> <div class="game-detail"> <div class="game-name"> 虎牙迷你世界盒子 </div>

<div class="game-info"> 休闲<i></i>12.7M </div> <div class="game-one-word"> </div> </div>

</a>

#### 1.2.2图片alt

spider无法识别图片内容，每个图片都需添加ALT属性，ALT文字根据图片内容进行添加；

#### 1.2.3h标签/强调标签

H标签，一个页面只有唯一1对h1标签；h2一般用于版块标题；h3一般用于子版块标题；strong一般用于加粗的文字链或正文关键词，首页H1一般用于LOGO或核心主词上，列表页和详情页用于标题。

**m站标签与PC站同步**

#### 1.2.4nofollow属性

页面下载链接、系统后台、登录/注册、个人中心等链接都要加nofollow属性，集中权重避免分权；

#### 1.2.5链接title属性/title标签

对应完整的链接描述内容，提高用户体验，增加关键词密度；

### 1.3页面加载

1、用户打开页面时所能承受的时间，建议控制在3秒以内，且页面加载直接影响爬虫抓取效率；

2、页面数据加载页面内容需直接加载到源文件上，下拉加载才采用异步，spider无法爬取异步加载的内容；

3、除非实现网页特效等，建议少用js展现重要内容，对于代码较长的js以调用的形式分离出页面，且置于页面末尾

参考：<script language="javascript" type="text/javascript" src="/public/javascripts/pc2/common/ie6_png.js"></script>

4、flash、iframe、ajax和jquery等，如有使用到这些语言，尽量以其他方式替代，搜索引擎暂时无法做到高效地抓取内容；

### 1.4页面布局

纯div+css布局，简化div层的嵌套，减少对css样式的定义。尽可能以文字表现内容，而不是图片和js，同时注意删除不必要的空格或空行，提高爬虫抓取内容的效率；

### 2.0代码

### 2.1代码添加

#### 2.1.1禁止转码代码

<meta http-equiv="Cache-Control" content="no-siteapp" />

<meta http-equiv="Cache-Control" content="no-transform" />

**PC和移动页面都需要加上；**

#### 2.1.2适配代码

在pc版网页上，添加指向对应移动版网址的特殊链接 rel="alternate" 标记。这有助于发现网站的移动版网页所在的位置。

在移动版网页上，添加指向对应pc版网址的链接 rel="canonical" 标记。

例如，假设pc版网址为http://www.example.com/page-1，且对应的移动版网址为 http://m.example.com/page-1，那么此示例中的注释如下所示：

在pc版网页(http://www.example.com/page-1) 上，添加：

<link rel="alternate" media="only screen and (max-width: 640px)" href="http://m.example.com/page-1" >

而在移动版网页(http://m.example.com/page-1) 上，所需的注释应为:

<link rel="canonical"href="http://www.example.com/page-1" >

#### 2.1.3页面标识

在<head></head>中间加入代码，规则和对应关系见下：

如果是PC站，代码：<meta name="applicable-device"content="pc">

如果是移动站，代码：<meta name="applicable-device"content="mobile">

#### 2.1.4页面URL

#### URL唯一性

若同一个页面内容多路径URL打开会影响收录及排名，需确保每个页面只有唯一一个可打开访问的绝对URL，若多路径URL打开，需采用301跳转指向唯一URL；并且页面所有出现的链接均采用完整链接

（首页index特殊，无须301指向首页，可看序号1）

#### URL静态化

在静态页面上使用动态参数，会造成spider多次和重复抓取；url建议不要超过255byte，对于栏目列表页，使用目录名（url结构不宜太深），以 "/" 结尾；对于详情页，使用后缀名，如.html

#### URL规则

栏目列表页（以目录名），详细页（后缀名html） ，**具体路径由SEO确认输出**；

举例：

首页：http://www.域名.com/

列表页：http://www.域名.com/huodong/

详情页：http://www.域名.com/huodong/资讯ID.html

#### URL链接显示及打开方式

使用绝对完整链接，点击链接时新开页面（可在body前添加一行代码<base target="_blank" />实现，首页链接则可在当前页面跳转target="_self"）

#### 2.1.5meta language信息

采用：<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

#### 2.1.6 301重定向

任何可访问的站点首页的域名都指向首选域， **避免首页权重分流；**

举例：http://www.域名.com/index 要采用301指向http://www.域名.com/；

注意：列表页URL结尾为"/"，没带"/"的链接采用301指向带“/”；

#  

# 3.0常见情况

### 3.1页面改版

1、如果页面改版，原页面URL路径不变，原先已经有的路径继续沿用旧的URL，继承权重；

（新增的页面，URL和TKD由SEO定制提供）；

2、meta的信息和规则不变，包括TKD、适配代码、禁止转码代码；

3、页面下载抛包渠道，需与SEO确认抛包逻辑及抛包渠道标识；

4、页面数据加载页面内容需直接加载到源文件上，下拉加载才采用异步；

5、页面加载速度不超过3秒，页面加载直接影响爬虫抓取效率；
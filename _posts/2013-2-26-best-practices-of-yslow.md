---
layout: post
title: YSlow 最佳实践准则（初稿）
summary: YSlow 中包含 7 个分类共 35 条提高网站性能的实践准则。
category: javascript
published: false
list: 1
tags: [yslow, best practices]
---

{{page.title}}
==============

{{page.summary}}

### 减少 HTTP 请求 (#content)

终端用户等待页面响应时间的 80% 花费在前端，花费等待下载页面的各个组件上：images、stylesheet、scripts、flash 等。要让页面渲染更快，关键是减少页面组件个数，亦即减少 HTTP 请求，减少等待下载的时间。

一种减少页面中组件的方式是简化页面的设计。那么有没有既能保留页面中的丰富设计，又能快速加载页面的方式呢？这里有一些技术用来减少 HTTP 请求数，同时还支持富页面设计。

**合并文件** 是一种通过分别合并多个 js、css 文件为一个文件来达到减少 HTTP 请求的方式。尽管对页面之间变化比较多的站点来说，合并脚本、样式文件很有挑战，但是这部分的改进能够改进响应时间。

**[Css Sprites][CssSprites]** 是减少图片请求数目的首选。这种技术是合并背景图片到一张图片上，再通过 CSS 属性 `background-image` 和 `background-position` 控制显示期望的一部分。

**[Image maps][ImageMaps]** 合并多张图片为一张。总的文件大小一致，但是减少了请求数，提高了加载速度。Image maps 只在多个图片间是连续的情况下有效，比如导航条。定义图片地图的坐标非常繁冗且容易出错。其实导航条用图片地图也不好，不推荐这么使用。

**inline images** 使用 [data: URL scheme][UrlScheme] 嵌入图片数据到实际的页面。这会导致 HTML 文档尺寸变大。合并行内图片到样式文件（可以被缓存）可以减少 HTTP 请求数，并避免页面尺寸增大。行内图片还不被所有主流浏览器支持。

减少页面中的 HTTP 请求数是优化的一个开始，也是提高初次访问性能最重要的原则。Tenni Theurer 的文章 [Browser Cache Usage - Exposed](http://yuiblog.com/blog/2007/01/04/performance-research-part-2/) 中提到每天的访问者中 40-60% 没有站点的缓存，因此为这些第一次访问者提供快速响应是提高用户体验的关键。



### 使用 CDN(Content Delivery Network) (#server)

用户越接近站点服务器，响应时间也就越短。部署内容在多个地理分散的服务器上，对用户来说可以加快页面的下载速度。应该从哪儿着手呢？

第一步，为了实现地理上分散的内容分发，不要尝试重新把站点设计成分布式的。这种修改需要艰难的工作，如跨服务器的同步会话状态、数据库操作复制。尝试降低用户和服务器内容的距离会延迟甚至阻止应用架构的开发。

回想一下，终端用户 80-90% 的响应时间花费在下载页面组件上：图片、样式、脚本、Flash等。这是优化性能的金科玉律。与其开始重新设计应用的架构，不如首先分散网站的静态内容。幸亏有内容分发网络，这不仅极大地减少了响应时间，还很容易实现。

CDN 是一个跨地区的分布式网站服务托管，能够高效地向用户分发内容。它根据检测到的网络接近程度，有选择地向特定用户发送内容。例如，根据最少的网络节点或最快的响应速度选择提供服务的服务器。

一些大的互联网公司有自己的CDN，但是选择CDN服务提供商会更经济有效，如 Akamai Technologies、EdgeCast 或 level3。对于刚起步的公司和个人站点，CDN 的花费还是很高昂的，但是随着目标受众越来越大，甚至是成长为全球性的，为了提高响应速度，CDN 就是必须的了。雅虎自从把它的静态内容从原来的应用服务器挪到CDN，客户端的响应时间改善了20%甚至更多。切换到CDN需要一些简单的代码修改，但是带来的显著地站点速度提升是巨大的。



### 添加过期时间或缓存控制 Header (#server)

这条规则有两个方面：

- 对于静态组件，通过设置尽可能久的头部 `Expires`，来达到永不过期。
- 对于动态组件，用适当的头部 `Cache-Control` 帮助浏览器完成一定条件的请求。

网页设计越来越复杂，同时意味着页面中存在更多的脚本、样式、图片、Flash 等。第一次访问必须完成一些 HTTP 请求，但是通过 Expires 设置可以让一些组件可以被缓存起来。这可以避免后续请求时不必要的请求。过期设置更多的对图片使用，但是他们应该被用在所有组件上，包括样式、脚本、Flash。

浏览器（以及代理）会使用缓存减少 HTTP 请求，让网页更快的加载。服务器通过设置返回内容的头部 Expires 告诉客户端一个组件应该被缓存多长时间。这是一个过期设置，告诉浏览器内容在 April 15, 2010 之前不会过期：

> Expires: Thu, 15 Apr 2010 20:00:00 GMT

如果服务器是 Apache，可以使用 ExpiresDefault 设置相对与现在的过期时间。这个例子表示设置过期时间为从访问时间起10年：

> ExpiresDefault "access plus 10 years"

注意，如果想改变一个很久以后过期的组件，必须改变该组件的名字。在雅虎，经常需要做这部分工作：版本号数字要被嵌入到组件名称中，如 yahoo_2.0.6.js。

头部过期设置，只对用户访问过一次后的页面起作用。对于第一次访问站点或之前没有该站点缓存的用户，过期设置不会对减少 HTTP 请求有效果。因此，这个改进效果依赖于用户用准备好的缓存(包含页面所有组件)发起访问的频繁程度。我们[在yahoo上测试](http://yuiblog.com/blog/2007/01/04/performance-research-part-2/) 过这个问题，使用有缓存的访问大概在75-85%。通过使用未来过期头部设置，可以增加浏览器缓存的组件数，这些组件将在后续访问中被重用，而用户不必再发出哪怕一字节的请求。



### Gzip 组件 (#server)

一个请求从发起到经过网络返回的时间，应该被显著地减少。用户的带宽、网络提供商、交换节点的远近等明显的超出了开发团队的控制，但是有其他变量影响响应时间。可以通过压缩减少回应内容的大小，来减少响应时间。

从 HTTP/1.1 开始，web 客户端通过在请求中设置头部 Accept-Encoding 来指示支持的压缩形式。

> Accept-Encoding: gzip, deflate

如果服务器在请求头部看到这条设置，它可能会用客户端列出的某种压缩形式压缩响应内容。服务器通过在返回中的头部设置 Content-Encoding 通知客户端压缩方式：

> Content-Encoding: gzip

当前，Gzip 是最流行和有效的压缩方法。它被 GNU 开发，并被 [RFC 1952](http://www.ietf.org/rfc/rfc1952.txt) 标准化。其他的压缩形式还有 deflate，它既不够有效，也不怎么流行。

Gzip 压缩可以减小响应内容大约 70%。今天大约 90% 通过浏览器的网络传输声称支持 gzip。如果使用 Apache，配置 gzip 模块依赖于服务器版本： Apache 1.3 使用 mod_gzip，Apache 2.x 使用 mod_deflate。

目前，已知的问题是，浏览器和代理期望的结果和接收到的压缩内容不一致。幸运的是，这些边缘问题的发生随着旧版本浏览器的减少而降低。Apache 的模块通过自动添加 Vary 的头部设置帮助解决该问题。

服务器基于文件类型选择哪些被压缩，但是由它们决定哪些被压缩明显地有太多限制。绝大多数网站压缩 HTML 文件，同时，压缩脚本和样式文件也是值得的，但是很多站点遗漏了这点。事实上，任何文本响应包括 XML 和 JSON 都值得做压缩。图片和 PDF 不能被压缩，因为它们已经被压缩了，尝试压缩它们，不止是浪费 CPU 资源，还可能增加文件尺寸。

尽可能多地压缩多种类型文件，是一种精简页面和促进用户体验的简单方式。


### 样式文件放到顶部 (#css)

在 Yahoo! 研究性能时，发现把样式文件放到文档的 HEAD 中，页面显得下载更快。这是因为样式放到头部，允许页面渐进地渲染。

前端工程师关心性能，希望页面能渐进加载，也就是说，我们希望浏览器能尽可能快的显示任何内容。当用户网速慢，而页面内容又很多时，这种方式就显得尤为重要。视觉反馈（如进度条）被研究和证明对用户是很重要的。在我们看来，HTML 页面就是进度条指示器。队等待着页面加载的用户来说，页面逐步头部、导航、顶部的图标 Logo 等内容就是一个进度指示器。这些增强了整个用户体验。

把样式文件放到底部，在很多浏览器中，包括 IE，都会导致阻止页面的逐步渲染。浏览器阻塞渲染，以避免后面样式改变时还要重新绘制元素。用户会看到一个空白的卡住的页面。

[HTML 说明书](http://www.w3.org/TR/html4/struct/links.html#h-12.3) 明确声明样式文件要被包含到页面的头部：“不像 A 标记，LINK 标记可能只出现在文档的 HEAD 部分，尽管它可以出现任意多次”。不管是替代方案，还是无样式的空白页或flash，这样做都是值得的。最明智的解决方案，就是按照 HTML 说明书的方式，将样式文件放到页面的头部加载。


### 脚本文件放到底部 (#javascript)

脚本会阻塞页面资源的并行加载。[HTTP/1.1 说明书](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.1.4) 建议浏览器对同一个域名并行加载不超过两个。如果把图片资源放到多个域名上，那一个站点就可以同时并行加载两个以上的资源。但是在加载一个脚本文件时，浏览器不会加载任何其他资源，即使他们来自不同的域名。

有些情况下，要把脚本文件放到底部并不是那么容易的。比如脚本使用 `document.write` 来插入部分页面内容，那这个脚本就不能改变一点位置。同时可能还有作用域的问题。通常有很多种方式解决这种情况。

一个常用的变通建议是使用延迟执行脚本。属性 `DEFFER` 表明脚本中不含有 `document.write`，并告诉浏览器可以继续渲染。但是，Firefox 不支持这个属性。在 IE 中，脚本会被延迟执行，但表现并不一定是预期中那样。如果一个脚本可以被延迟执行，那它就可以被放到页面底部。这样就能让页面加载的更快些。


### 避免使用样式表达式 (#css)

CSS 表达式是强大而危险的动态设置 CSS 属性的方式。从 IE5 开始支持，但[从 IE8 开始弃用](http://msdn.microsoft.com/en-us/library/ms537634%28VS.85%29.aspx)。如下示例，通过 CSS 表达式设置背景色每小时轮换一次：

> background-color: expression( new Date().getHours() % 2 ? '#BBD4FE' : '#F08A00');

`expression` 方法接收一个 js 表达式参数，这个 js 表达式的执行结果被设置为 CSS 属性值。`expression` 方法会被其他浏览器忽略，因此如果要在 IE 浏览器中设置跨浏览器的独特体验，通过CSS表达式设置属性非常有用。

CSS 表达式的一个问题是它的执行频率比人们希望的要高很多。它们不止在页面渲染和重置大小时执行，在页面滚动时也会执行，甚至当用户控制鼠标在页面上移动时也会执行。在 CSS 表达式中加一个计数器，可以跟踪表达式什么时候以及有多频繁的执行。在页面上移动鼠标可以很容易就让 CSS 表达式执行一万次以上。

一个减少 CSS 表达式执行次数的方式，是设置一次性的 CSS 表达式，即在第一次执行了表达式之后，用一个明确的值替换掉表达式定义。如果在整个页面周期内，确实需要动态设置样式属性，一个可接受的替代方式是用事件处理替换 CSS 表达式。如果必须使用 CSS 表达式，那么要注意它可能会被执行成千上万次，并会影响页面性能。


### 将 js 和 css 放到外部文件 (#javascript, css)

很多优化性能的规则都是处理如何管理外部组件。然而，在这之前需要问一个更基础的问题：js 和 css 应该放在外部文件，还是在页面内部？

实际引用中，使用外部文件页面会加载的更快，因为 js 和 css 文件可以被浏览器缓存。嵌入到页面中的 js 和 css，在每次请求 HTML 页面时都会被重新加载。这样会减少发出的 HTTP 请求数，但是会增加一个 HTML 页面的大小。另一方面，如果 js 和 css 外部文件被浏览器缓存下来，那么 HTML 页面的大小会变小，还不会增加 HTTP 请求。

那么，问题的关键因素就成了根据 HTML 页面请求数，缓存 js 和 css 的频率。尽管这个因素很难准确量化，但可以通过多个方面来衡量。如果使用你网站的用户，在每个会话周期内会查看多个页面，并且这些页面共用一些 js 和 css，那么用户就能从缓存这些 js 和 css 外部文件中获得更大的好处。

很多站点从这些衡量中取中。对于这些站点，通常更好的解决方案是部署 js 和 css 为外部文件，唯一的例外是首页，比如 [Yahoo! 首页](http://www.yahoo.com) 和 [My Yahoo!](http://my.yahoo.com)。这些首页中把资源嵌入到页面内部更合适，因为在每个会话中，首页被访问的次数比较少（可能只有一次），内嵌的 js 和 css 反而可以大大加快终端用户的响应。

特别的，首页不同其他页面，单独处理，有很多种技术减少 HTTP 请求数，以达到类似缓存外部文件能得到的好处。其中一种是内嵌 js 和 css 在首页中，然后在页面加载完后动态加载外部文件，随后访问的页面中再对这些外部文件引用，就可以直接使用缓存中的文件。


### 减少 DNS 查找 (#content)

域名系统(DNS)指示的是域名和 IP 地址的对应关系，就像电话本指示的是一个人的名字与他的号码的对应关系。当你在浏览器中输入 `www.yahoo.com` 时，被浏览器联系的 DNS 解析器就会返回这个网站服务器的 IP 地址。DNS 是要耗时的，对于一个给定的域名，DNS 要找到对应的 IP 地址，大概要花费 20-120 毫秒。在 DNS 查找完成之前浏览器不能从这个域名下载任何资源。

为了更好的性能，DNS 查找结果会被缓存起来。这个缓存发生在 ISP（因特网服务提供商）或本地网络的特殊缓存服务器上，个人用户的电脑也会缓存。DNS 查询信息会留存在操作系统的 DNS 缓存上（windows 系统是“DNS client service”）。多数的浏览器也有自己的缓存，独立于操作系统的缓存。随着浏览器缓存的 DNS 记录的增多，它就不用再去操作系统请求已经缓存的 DNS 记录了。

IE 浏览器根据注册表 `DnsCacheTimeout` 的设置，默认会缓存 30 分钟的 DNS 查询记录。而 Firefox 是一分钟，可以用 `network.dnsCacheExpiration` 配置。

当客户端没有 DNS 缓存（浏览器和操作系统都没有）时，DNS 查找的次数等于页面中唯一域名的个数。这包含页面中所有使用到的 URL，图片、脚本、样式、Flash 等使用的域名。减少唯一域名的个数就能减少 DNS 的查询次数。

减少页面中的唯一域名，同时可能会减小页面中并行下载的总量。避免 DNS 查询可以减少响应时间，但是减少了并行加载又可能增加响应时间。我的原则是拆分页面资源到 2-4 域名。这是一个比较好的权衡，既保证减少 DNS 查询，又允许高度的并行加载。


### 压缩 js 和 css (#javascript, css)

压缩是通过移除代码中不必要的字符降低文件大小，达到改进加载时间的。当代码被压缩后，注释和不需要的空白字符（空格、空行、制表位）都被移除。对于 js 文件，改进加载时间性能是因为被加载的文件尺寸减小了。两个流行的压缩工具是 [JsMin](http://crockford.com/javascript/jsmin) 和 [YUI Compressor](http://developer.yahoo.com/yui/compressor/)，后者还可以压缩 CSS。

模糊混淆处理是可以应用到源码的替代优化方案。它比压缩更复杂，如果自己进行模糊混淆化处理，也更容易产生问题。在对美国排名前十的网站调查中发现，压缩可以减小 21% 的文件尺寸，而模糊混淆可以减小 25%。尽管模糊混淆能够更多的减小文件大小，但是压缩 js 的风险要小的多。

除了压缩外部文件的 js 和 css，内嵌的 `<script>` 和 `<style>` 代码块也可以并应该被压缩。尽管已经用 gzip 压缩了脚本和样式，但压缩代码，仍然能够减少 5% 甚至更多。随着使用的 js 和 css 文件尺寸的增加，压缩代码能得到的优势也会增加。


### 避免重定向 (#content)

重定向通过状态码 301、302 完成。下面是一个 HTTP 头部 301 时的示例：

{%highlight html%}
HTTP/1.1 301 Moved Permanently
Location: http://example.com/newurl
Content-Type: text/html
{%endhighlight%}

浏览器将用户自动导航到 `Location` 字段指定的网址。所有重定向必需的信息都是头部，而响应的主体内容则是空的。不管名称是 301 还是 302，实际上响应内容都会被缓存，除非有附加的头部定义指明如何处理，如 `Expires` 或 `Cache-Control`。其他跳转的方式可以通过元数据的 refresh 标签和 js 控制，但是如果必须要跳转页面，首选的技术就是标准的 3xx HTTP 状态码，主要是这样可以保证回退按钮可以正确工作。

主要要注意的是，跳转会降低用户体验。在用户和页面中插入跳转，会造成页面上所有东西都延迟，因为在新页面返回前，页面上没有任何东西可渲染，也没有任何资源被加载。

有一个最浪费的跳转频繁发生，但是开发者通常却没有意识到，这种情况一般发生在，地址结尾的斜线应该存在，却缺失了。比如要去 <http://astrology.yahoo.com/astrology> 结果返回一个包含 301 的重定向指向 <http://astrology.yahoo.com/astrology/> (注意结尾的斜线)。如果使用的是 Apache，可以通过指定 `Alias` 或 `mod_rewrite` 或 `DirectorySlash`指令 修正这个问题。

连接旧站点到新站点是跳转的另一个常用的地方，其他类似的还包括连接站点的不同部分，根据情况（根据浏览器、账户等）导航用户到特定页面。用跳转链接两个站点只要增加一点代码就好，很简单。尽管在这种情况下可以减少开发的复杂度，但是还是会降低用户体验。对于代码在同一服务器的同一域名下，跳转可以用 `Alias` 和 `mod_rewrite` 方案替代；如果域名改变，跳转的替代方案是设置 CNAME (一个 DNS 记录，可以创建从一个域名指向另一个域名的别名)，并联合使用 `Alias` 和 `mod_rewrite`。


### 移除重复的脚本 (#javascript)

在一个页面中多次包含同一脚本，是对性能的伤害。可能这不是你通常可能想到的。通过对美国排名前十的网站审查，发现其中两家有重复的脚本。两个主要因素增加了单个页面中脚本重复的发生几率：团队规模和脚本数量。如果发生重复脚本，那它们通过增加不必要的 HTTP 请求和浪费 js 的执行资源来伤害用户体验。

不必要的 HTTP 请求会在 IE 中发生，而 Firefox 则不会。在 IE 中，如果一个外部的脚本文件被引入了两次，并且还没被缓存，在页面加载时会产生两次 HTTP 请求；如果被缓存了，在用户刷新页面时，额外的 HTTP 请求居然还是会产生。

除了浪费 HTTP 请求，还会浪费 js 被多次重复执行的时间。多余的 js 执行在 IE 和 Firefox 中都会发生，不管 js 是否被缓存。

避免意外重复加载多次同一脚本的方式之一是在模板系统中实现一个脚本管理机制。典型的加载 js 方式是在页面中使用 `<script>` 标签：

{%highlight html%}
<script type="text/javascript" src="menu_1.0.17.js"></script>
{%endhighlight%}

在 PHP 中一个替代方式是创建一个名为 `insertScript` 的函数：

{%highlight php%}
<?php insertScript("menu.js") ?>
{%endhighlight%}

除了避免同一个脚本文件被加载多次，这个函数还可以处理脚本的其他问题，如依赖关系检查，为脚本文件名添加版本号以支持头部过期设置。


### 配置 ETags (#server)

实体标签(ETags)是判断浏览器端缓存的资源和服务器上保存的原始资源是否匹配的机制，这里的实体表示网页上的资源：图片、脚本、样式等。ETags 相对最后修改时间提供了一种更灵活的验证资源的机制。一个 ETag 是一个唯一的标识组件版本的字符串，唯一的格式要求是这个字符串要被引号括起。服务器是在返回内容的头部 `ETag` 标签指定 ETag：

{%highlight html%}
HTTP/1.1 200 OK
Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
ETag: "10c24bc-4ab-457e1c1f"
Content-Length: 12195
{%endhighlight%}

之后，如果浏览器要验证一个组件，会通过在头部设置 `If-None-Match` 把 ETag 返回给服务器。如果 ETag 匹配，状态码 304 的响应会被返回，本例中，会减少 12195 字节的大小。

{%highlight html%}
GET /i/yahoo.gif HTTP/1.1
Host: us.yimg.com
If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
If-None-Match: "10c24bc-4ab-457e1c1f"
HTTP/1.1 304 Not Modified
{%endhighlight%}

使用 ETag 的问题是，它们被服务器构造出，在这个服务器中是唯一的存在，但是浏览器之后验证时，如果请求的是其他服务器，那么就不会被匹配，尤其是现在站点通常都用服务器集群来处理请求的情况下，这种不匹配更容易发生。默认情况下，Apache 和 IIS 都会在 ETag 中嵌入数据，以显著减少在不同浏览器之间验证的机率。

在 Apache 1.3 和 2.x 版本中，ETag 格式是 `inode-size-timestamp`。尽管一个文件可能在多个服务器中是同一目录下的，有相同的大小、权限、时间戳等，但是不同服务器上的 inode 是不同的。

IIS 5.0 和 6.0 上的 ETag 也有类似的问题。IIS 上的格式为 `Filetimestamp:ChangeNumber`，`ChangeNumber` 是 IIS 的配置变化总数，显然一个站点所有服务器的这个值不太可能保持一致的。

Apache 和 IIS 产生的 ETag 最终结果是，都不能在不同服务器间精确匹配同一个组件。如果 ETag 不能匹配，那么用户就不能接收到小而快速的 304 状态响应，失去了 ETag 的设计初衷；替代的是得到标记为 200 状态码的组件全部数据。如果把站点放到一个服务器上，不会出现这个问题。但是如果用多个服务器部署站点，并且使用 Apache 或 IIS 的默认 ETag 配置，那么用户会得到一个慢速的网页，服务器产生更多的下载，消耗更多的带宽，代理器不会高效的缓存内容。即使组件有未来的过期设置，当用户点击重新加载或刷新，GET 请求还是会发生。

如果不能得到 ETag 灵活验证模式提供的优势，那最好移除所有的 ETag 设置，通过在头部设置 `Last-Modified` 仍然能够得到基于组件时间戳的验证。而且移除 ETag 还可以减少请求和回应的头部尺寸。[微软维护的文章](http://support.microsoft.com/?id=922733)描述了如何移除 ETag。Apache 中只要在配置文件中添加下面这行就能移除：

> FileETag none


### 缓存 Ajax (#content)

Ajax 一个经常被应用的优势，是它能给用户一个即时的反馈，因为它异步地从后端服务器请求信息。但是 Ajax 并不能保证用户在等待异步 js 和 XML 响应返回时不再有其他操作。在很多应用中，用户是否等待 Ajax 返回，取决于 Ajax 的使用方式。在一个基于 web 的 email 客户端，用户会一直等待符合搜索条件的 email 结果。“异步”并不意味着“即时”，区分这点很重要。

为了增进性能，优化 Ajax 响应是很重要的。改进 Ajax 性能最重要的方式是缓存 Ajax 响应，就像前面讨论过的 添加过期时间或缓存控制。其他规则也适用于 Ajax：

- gzip 组件资源
- 减少 DNS 查找
- 压缩 JavaScript
- 避免跳转
- 配置 ETags

让我们看一个例子。一个 web 2.0 的邮箱客户端很可能用 Ajax 自动下载用户的地址簿，如果这个 Ajax 的响应结果通过设置过期时间或缓存控制被缓存了下来，并且用户地址簿在连续的两次访问间没有变化，那么后一次访问对地址簿的请求应该从上一次的缓存中读取。如果不是请求新的，而是适用缓存的内容，那么浏览器必须被通知这一行为。要完成这一行为，可以在 Ajax 请求 URL 中添加一个表明地址簿上次修改的时间戳，如 `&t=110241516` ，如果从上次加载后没有再修改，那么时间戳和上次是相同的，将会从缓存中读取，避免了一次 HTTP 往返。如果修改过，那么时间戳的存在就确保这次请求是新的 URL，就不匹配缓存中的内容，浏览器将会请求新的地址簿。

即使 Ajax 是动态创建，甚至可能只为一个用户所用，它们仍然可以被缓存起来。这样做将让你的 web 2.0 应用更快。


### 尽早输出缓冲区 (#server)

当用户请求一个页面，后端服务器组合页面可能会花费 200-500ms，在这段时间，浏览器是空闲的，它在等待数据返回。PHP 中有个函数 `[flush()](http://php.net/flush)`，这个函数允许先输出准备好的部分 HTML，这样浏览器就能先处理这部分，并开始读取相关资源，同时服务器继续在后端处理剩余的 HTML 部分。最大的好处是充分利用了前后端。

一个好的输出时机是 `<head>` 标记后面，因为网页的头部通常很容易产生，并且包含的 CSS 和 JavaScript 文件可以让浏览器并行加载，同时后端仍在继续处理。

{%highlight html%}
  ... <!-- css, js -->
</head>
<?php flush(); ?>
<body>
  ... <!-- content -->
{%endhighlight%}

[Yahoo! 搜索](http://search.yahoo.com/) 率先研究并通过真实用户测试检验这一技术的效益。


### Ajax 请求使用 GET 方式 (#server)

[Yahoo! 邮箱](http://mail.yahoo.com/) 团队发现，当使用 `XMLHttpRequest` 时，浏览器实现 POST 传送是分两步的：先发送头部，再发送数据。所以最好使用 GET 方法，它只会发送一个 TCP 数据包（除非你的 cookie 很大）。IE 中最大的 URL 长度是 2K，如果 URL 大于 2K 就不能使用 GET 方法了。

有趣的是 POST 实际上发送数据表现的与 GET 截然不同。根据 [HTTP 规则说明](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)，GET 的意思是获取信息，所以实际表示（语义上）的是：只请求数据，而不是向服务器发送并存储数据。


### 延迟加载组件 (#content)

仔细检查自己的网页并且问自己：“初始渲染页面时，哪些组件是绝对必需的？”，其他的内容和组件可以等待之后再加载。

JavaScript 是理想的候选者，它可以按页面加载完成事件分成之前和之后两部分。比如你的 js 代码或 js 库实现拖拽和动画，这些代码就可以被延后加载，因为在页面上拖拽元素发生在页面渲染完成后。其他的可以延迟加载的组件包括隐藏内容（用户操作后才会显示）和未在可视区的图片。

一些帮助性工具：[YUI Image Loader](http://developer.yahoo.com/yui/imageloader/) 可以延迟加载视区外的图片，[YUI Get 公共库](http://developer.yahoo.com/yui/get/) 是动态加载 js 和 css 文件的方式。样例可以用 Firefox 打开的网络面板查看 [Yahoo! 首页](http://www.yahoo.com/)。

能看到，其他的开发最佳实践与性能优化的目标是一致的。这种情况下，增强的改革观点告诉我们，当支持 JavaScript 时，可以增进用户体验，但是必须保证没有 JavaScript 时，页面也能正常运行。在保证页面正常工作的前提下，就可以用延迟加载增加更丰富的效果，如拖拽动画。


### 预加载组件 (#content)

预加载看起来和延迟加载是相反的，其实它是为了不同的目的。预加载是利用浏览器的空闲时间，请求那些可能将来用到的组件（如图片、脚本、样式）。这样，当用户访问其下一个页面时，缓存中已经有了大多数的组件，打开速度会更快。

实际上有这么几种预加载类型：

- 无条件预加载 —— 一旦页面加载完成，就开始加载一些额外的组件。作为一个例子，查看 `google.com`，看它是如何在加载完后请求合并的图片的，这张合并的图片并不是它首页需要的，但却是所有的搜索结果页需要的。
- 有条件预加载 —— 基于用户的操作，预测用户下一步可能的去向，预加载对应的资源。在 <search.yahoo.com> 可以看到在文本框输入后，它是如何请求额外资源的。
- 有预期的预加载 —— 在运行一个重新的设计时提前预加载。它通常发生在重新设计后，你会听到：“新网站很酷，但是比原来的慢”。这个问题的一部分原因应该是，用户有老版站点的所有缓存，但是新版却是一点缓存也没有。可以在新版运行前，预加载一部分新版的资源，这可以减轻这缓存造成的速度影响。旧站点可以在浏览器的空闲时间下载新站点需要的图片和脚本。


### 减少 DOM 元素个数 (#content)

复杂的页面意味着更多的字节需要下载，也意味着 JavaScript 更久才能访问 DOM。比如你想要给一个节点添加事件处理函数，遍历 500 个 DOM 元素和遍历 5000 个是不同的。

过高的 DOM 元素数，肯定可以在不移除必要内容的情况下，通过修改标记来得到改进。你有没有使用嵌套表格作为布局的打算？是否只是为了修复布局问题放置了大量的 `<div>` ？或许有更好的也更语义化的正确方式组织标记。

[YUI CSS 公共库](http://developer.yahoo.com/yui/)对布局有很大的帮助：`grids.css` 可以帮助部署整个布局，`fonts.css` 和 `reset.css` 可以移除浏览器默认的格式。这是一个尝鲜的机会并思考自己的标记处理方式，比如只在语义需要的地方使用 `<div>`，而不是只是为了换行。

DOM 总的元素数很容易统计，只要在 Firebug 的 console 中输入：

> document.getElementsByTagName('*').length

到底多少元素是太多？查看其他类似的有良好标记的网页，比如 Yahoo! 首页作为一个内容比较多的网页，它的 DOM 元素仍然低于 700。


### 分离组件到多个域 (#content)

拆分组件可以获得最大化的并行下载。为了避免 DNS 查找带来的性能惩罚，确保使用 2-4 个域名。比如可以把 HTML 和动态内容放到 `www.example.org` 域名下，拆分静态组件到 `static1.example.org` 和 `static2.example.org` 两个域名下。

关于该项的更多信息可以参考 Tenni Theurer 和 Patty Chi 的文章 [Maximizing Parallel Downloads in the Carpool Lane](http://yuiblog.com/blog/2007/04/11/performance-research-part-4/)。


### 减少 iframe (#content)

iframe 允许一个 HTML 被插入到父级文档中。了解 iframe 是如何工作的，对高效利用 iframe 非常重要。

`<iframe>` 的优点：

- 帮助显示较慢的第三部分内容，如广告
- 安全沙箱
- 并行加载脚本

`<iframe>` 的缺点：

- 占用大量资源，即使是空白页
- 页面载入时阻塞页面
- 不够语义化


### 不要返回 404 (#content)

HTTP 请求的消耗是昂贵的，一次请求得到一个无用的响应（比如404未找到页面）是完全没必要的，还会降低用户体验，总之，没有任何好处。

有些站点将 404 页面处理为 “你的是不是要找xx？”，这种方式极大地提高了用户体验，但还是浪费了服务器资源（如数据库等）。如果指向外部的 js 文件错误并返回 404，那么这种情况尤其严重，首先，它将阻塞并行下载；其次，浏览器可能会尝试解析 404 内容，试图找出可用的 js 代码。


### 减小 cookie 尺寸 (#cookie)

HTTP 有很多种用途，比如身份验证和个性化配置。cookie 信息通过包含在 HTTP 头部完成服务器和浏览器的数据交换。重要的一点是，要尽可能减小 cookie，以减小 cookie 对用户响应时间的影响。

更详细的信息参见：[When the Cookie Crumbles](http://yuiblog.com/blog/2007/03/01/performance-research-part-3/)。这里摘要这个研究的关键信息：

- 消除不必要的 cookie
- 尽可能减小 cookie 尺寸，以减小对用户响应时间的影响
- 谨慎地在合适的域名级别设置 cookie，不要影响到其他子域名
- 设置适当的过期时间，更早的过期时间或尽可能早地移除（不设置过期时间），会改善用户的响应时间。


### 对组件使用没 cookie 的域 (#cookie)

当浏览器请求一张静态图片时，cookie 也会随着请求发送过去，这些 cookie 对服务器来说没有任何用处。这种情况下，cookie 占用网络传输资源，没有任何好处。所以应该确保用不带 cookie 的请求获取静态组件。可以创建一个子域名，然后把所有静态组件都放到这个域名下。

如果域名是 `www.example.org`，可以把所有的静态组件都放到 `static.example.org`。然而，如果已经在顶级域名 `example.org` 而不是 `www.example.org` 设置了 cookie，那么所有指向 `static.example.org` 的请求中还是会包含这些 cookie。这种情况下，只能买一个全新的域名，把静态组件放在该域名下，并保持该域名没有 cookie。Yahoo! 用 yimg.com, YouTube 用 ytimg.com，Amazon 用 images-amazon.com，等等。

把静态资源放在无 cookie 的域名下，还有一个好处，就是有些代理可能拒绝缓存带有 cookie 请求的组件。相关的一点是，如果想用 example.org 或 www.example.org 用作首页，要把 cookie 带来的影响考虑进去。如果省略 www ，那么只能选择把 cookie 写到 *.example.org 下，所以为了性能考虑，最好是使用 www 的子域名，并把 cookie 写到这个子域名下。


### 减少 DOM 访问 (#javascript)

用 JavaScript 访问 DOM 元素很慢，为了提高页面的响应速度，应该：

- 缓存对元素的引用
- 在 DOM 树之外更新节点，然后把它加到 DOM 树上
- 避免用 JavaScript 固定布局

更多内容参考 YUI 的 [High Performance Ajax Applications](http://yuiblog.com/blog/2007/12/20/video-lecomte/)


### 开发巧妙的事件处理函数 (#javascript)

有时页面响应迟钝，是因为在 DOM 树上很多不同元素节点绑定了太多的处理函数，并且这些处理函数被执行的太频繁。这也是为什么使用事件代理更可取的原因。如果一个 div 内有 10 个 button，那么最好在 div 容器上绑定一个事件处理程序，而不是为每个 button 分别绑定一个处理程序。事件冒泡可以让你捕获到事件，并知道是哪个 button 最初触发的事件。

不必为了用 DOM 树做些什么而等待 onload 事件发生。通常需要的是要访问的 DOM 树中可用的元素节点，没必要等待所有图片都下载完成。考虑用 `DomContentLoaded` 事件，而不是 onload，但是在所有浏览器中都要等到它可用，可以使用 [YUI Event](http://developer.yahoo.com/yui/event/) 库中的 `[onAvailable](http://developer.yahoo.com/yui/event/#onavailable)` 方法。

更多信息查看 YUI 的文章 [High Performance Ajax Applications](http://yuiblog.com/blog/2007/12/20/video-lecomte/)。


### 选择 `<link>` 而不是 `@import` (#css)

前面的一个关于 CSS 的最佳实践是把 CSS 放到顶部，以便页面可以逐步渲染。

在 IE 中，`@import` 的表现和把 `<link>` 放到页面的底部是一样的，所以最好不要使用它。


### 避免用滤镜 (#css)

IE 专有属性 AlphaImageLoader 滤镜是为了修复 IE7 以下版本中 PNG 半透明颜色的问题。这个滤镜带来的问题是，它会阻塞渲染，并且冻结浏览器直到图片加载完成。它还会增加内存消耗，作用在所有元素上，而不只是图片。所以滤镜的问题很多。

最好的办法就是完全避免使用 AlphaImageLoader，有 PNG8 实现优雅退化，PNG8 在 IE 中表现正常。如果绝对需要使用滤镜，那么使用下滑线 hack `_filter`，避免对 IE7+ 的用户造成影响。


### 优化图片 (#images)

在设计师完成图片之后，上传到服务器之前，仍有一些事情可以尝试优化：

- 检查 GIF 图片，查看使用的调色板尺寸是否与图片包含的颜色数目匹配。使用 [imagemagick](http://www.imagemagick.org/) 可以方便地用 `identify -verbose image.gif` 检查。如果看到一个图片只有 4 种颜色，却使用了 256 的调色板，那这就有很大的优化空间。
- 转换 GIF 为 PNG，并查看是否节约了尺寸。通常都会有所节约。开发者经常犹豫是否在支持有限的浏览器中使用 PNG，现在来看，那是过去的事了。实际的问题是真色彩 PNG 的透明度问题，但是再次说明，GIF 既不是真色彩，也不支持多种透明度变化。所以 GIF 可以做的，PNG8 都可以做（除了动画）。一条简单的 imagemagick 命令，可以转换 GIF 为安全可用的 PNG：`convert image.gif image.png`。"All we are saying is: Give PiNG a Chance!"
- 运行 [pngcrush](http://pmt.sourceforge.net/pngcrush/)（任何优化 PNG 的工具）优化所有的 PNG 图片。如： `pngcrush image.png -rem alla -reduce -brute result.png`。
- 在所有 JPEG 图片上运行 jpegtran。这个工具可以对 JPEG 图片进行无损操作（如旋转），也可以用来优化图片，移除注释及其他无用的信息（如 EXIF 信息）： `jpegtran -copy none -optimize -perfect src.jpg dest.jpg` 。


### 优化 CSS Sprites 图片 (#images)

- 在 CSS Sprites 中，将图片横向排列通常比竖向排列，获得的文件更小
- 合并相似颜色的图片，可以减少整张图片的颜色数，理想情况下小于 256 更适合用 PNG8
- 保持手机友好性，CSS Sprites 上的图片不要间隔太大。这对文件大小并不会有太大的优化，却可以在用户浏览器在解压图片到像素展示中节约更多内存。100 * 100 的图片是一万像素，而 1000 * 1000 的图片是一百万像素。


### 不要在 HTML 中缩放图片 (#images)

不要使用超过你需要的过大的图，因为 HTML 中你可以设置图片的宽高。如果确实需要：

{%highlight html%}
<img width="100" height="100" src="mycat.jpg" alt="My Cat" /> 
{%endhighlight%}

那么图片（mycat.jpg）应该用 100*100px 的，而不是把 500*500px 的压缩。


### `favicon.ico` 尽量小且可缓存 (#images)

favicon.ico 是放在服务器根目录下的图片。不幸的是它是必需的，因为即使你不关心它，浏览器仍然会去请求它，所以最好不要响应成 404 未找到。同时，因为它和主站是同一个服务器，所以对它的请求都会带上 cookie。这张图片也会影响到加载队列，比如，在 IE 中加载一个页面时，请求额外的组件，那么 favicon.ico 必需在加载额外组件之前加载。

为了减少 favicon.ico 带来的弊端，必需确保：

- 足够小，最好小于 1K
- 设置合适的过期时间（因为想要改变这个图片，无法重命名）。或许设置过期时间为未来的几个月比较合适，也可以通过比较修改时间来确定使用缓存的还是加载新的

[Imagemagick](http://www.imagemagick.org/) 可以帮助创建一个小的 favicon.ico。


### 保持组件小于 25K (#mobile)

这条约束是因为 iPhone 无法缓存大于 25K 的组件。注意，这是指*未压缩*时的尺寸。从这可以看出压缩的重要性，因为单独的 gzip 压缩可能压缩的还不够。

更多信息查看：[Performance Research, Part 5: iPhone Cacheability - Making it Stick](http://yuiblog.com/blog/2008/02/06/iphone-cacheability/)。


### 打包组件到多部件的文档 (#mobile)

打包组件到多个组成部分的文档中，类似有附件的邮件，它可以在一次请求中取回多个组件（记住：HTTP 请求是昂贵的）。当使用这项技术时，需要首先检查用户的客户端是否支持（iPhone 不支持）。


### 避免空的图片 `src` 属性 (#server)

图片的 src 属性是空字符串时，会发生几种出乎意料的后果。表现上有两种形式：

1. HTML： `<img src="">`
2. JavaScript： `var img = new Image(); img.src = "";`

两种形式会产生同样的结果：浏览器产生一次服务器请求。

- **IE** 发送一次到当前页面所在目录的请求。
- **Safari & Chrome** 发起一次到当前页面本身的请求。
- **Firefox** 3 及之前的版本表现和 Safari 与 Chrome 一致，但是 3.5 记录了这个问题\[[bug 444931](https://bugzilla.mozilla.org/show_bug.cgi?id=444931)\]，并不再发送请求。
- **Opera** 遇到一个空 src 的图片时不会做任何事。

**为什么这个行为不好？**

1. 大量无意义的传输会削弱服务器，尤其是页面每天有上百万次的访问时。
2. 创建永远不会被浏览的页面会浪费服务器的计算资源。
3. 可能损坏用户数据。只要追踪请求的状态，不管通过 cookie 还是其他方式，都可能损坏数据。即使图片请求并没有返回图片，所有的头部信息还是被浏览器读取并接收，包括所有的 cookie，同时其他的响应被抛弃，损害可能已经发生。

产生这一行为的根源是浏览器处理 URI 的方式。这一行为被定义在 “RFC 3986 - 统一资源定位符” 中，当遇到空字符串作为 URI 时，会被认为是一个相对的 URI，应该按照定义在 5.2 节中的算法进行处理，空字符串具体的例子被列在 5.4 节。Firefox、Safari、Chrome 都按找规则说明实现了正确的对空字符串的解析，而 IE 的解析是错误的，应该是依据 “RFC 2396 - 统一资源定位符” 中的早期规则（在 RFC 3986 中被废弃）。从技术上讲，浏览器做的是假定空字符串是一个相对的 URI，然后去处理这个相对URI。这里，我们说的空的 src 是非故意造成的。

HTML5 在 4.8.2 节中添加了对元素 src 属性的描述，以告诉浏览器不要发出额外的请求：

> src 属性必须存在，并且包含一个有效的 URL，这个 URL 引用一个非交互的、可以是动画的图片资源，既不能是分页，也不可以是脚本。如果元素的基础 URI 与文档的地址相同，那么 src 属性值必须是一个空的字符串。

可以看到，未来浏览器不会有这个问题。不幸的是，现在还没有对 `<script src="">`、`<link href="">` 的条款，或许现在就应该做出决定，避免浏览器意外地实现对他们的特殊行为。

这条规则是被 Yahoo! 的 JavaScript 专家 Nicolas C. Zakas 提出来的，更多信息参见：[Empty image src can destroy your site](http://www.nczonline.net/blog/2009/11/30/empty-image-src-can-destroy-your-site/)。




[CssSprites]: http://alistapart.com/articles/sprites "Css Sprites"
[ImageMaps]: http://www.w3.org/TR/html401/struct/objects.html#h-13.6 "Image maps"
[UrlScheme]: http://tools.ietf.org/html/rfc2397 "data: URL scheme"

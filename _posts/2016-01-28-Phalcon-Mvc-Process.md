---
layout: post
title: Phalcon Framework的Mvc结构及启动流程（部分源码分析）
excerpt: Phalcon本身有支持创建多种形式的Web应用项目以应对不同场景，包括迷你应用、单模块标准应用、以及较复杂的多模块应用，本次以最复杂的多模块应用为例，Phalcon版本为1.3。2，用一个Phalcon所创建的标准项目来分析
---

####Phalcon本身有支持创建多种形式的Web应用项目以应对不同场景，包括<a href="http://docs.phalconphp.com/en/latest/reference/micro.html">迷你应用</a>、<a href="http://docs.phalconphp.com/en/latest/reference/applications.html#single-module">单模块标准应用</a>、以及较复杂的<a href="http://docs.phalconphp.com/en/latest/reference/applications.html#multi-module">多模块应用</a>

####本次以最复杂的多模块应用为例，Phalcon版本为1.3.2，用一个Phalcon所创建的标准项目来分析

---

##创建项目

<p>Phalcon环境配置安装后，可以通过命令行生成一个标准的Phalcon多模块应用</p>

<pre><code class="sql"> phalcon project eva <span class="comment">--type modules</span>
</code></pre>

<p>入口文件为<code>public/index.php</code>，简化后一共5行，包含了整个Phalcon的启动流程，以下将按顺序说明</p>

<pre><code class="php"><span class="keyword">require</span> __DIR__ . <span class="string">'/../config/services.php'</span>;
<span class="variable">$application</span> = <span class="keyword">new</span> Phalcon\Mvc\Application();
<span class="variable">$application</span>-&gt;setDI(<span class="variable">$di</span>);
<span class="keyword">require</span> __DIR__ . <span class="string">'/../config/modules.php'</span>;
<span class="keyword">echo</span> <span class="variable">$application</span>-&gt;handle()-&gt;getContent();
</code></pre>

##DI注册阶段

<p>Phalcon的所有组件服务都是通过<a href="http://docs.phalconphp.com/en/latest/api/Phalcon_DI.html">DI（依赖注入）</a>进行组织的，这也是目前大部分主流框架所使用的方法。通过DI，可以灵活的控制框架中的服务：哪些需要启用，哪些不启用，组件的内部细节等等，因此Phalcon是一个松耦合可替换的框架，完全可以通过DI替换MVC中任何一个组件。</p>

<pre><code class="php"><span class="keyword">require</span> __DIR__ . <span class="string">'/../config/services.php'</span>;
</code></pre>

<p>这个文件中默认注册了<code>Phalcon\Mvc\Router</code>（路由）、<code>Phalcon\Mvc\Url</code>（Url）、<code>Phalcon\Session\Adapter\Files</code>（Session）三个最基本的组件。同时当MVC启动后，DI中默认注册的服务还有很多，可以通过DI得到所有当前已经注册的服务：</p>

<pre><code class="php"><span class="variable">$services</span> = <span class="variable">$application</span>-&gt;getDI()-&gt;getServices();
<span class="keyword">foreach</span>(<span class="variable">$services</span> <span class="keyword">as</span> <span class="variable">$key</span> =&gt; <span class="variable">$service</span>) {
        var_dump(<span class="variable">$key</span>);
        var_dump(get_class(<span class="variable">$application</span>-&gt;getDI()-&gt;get(<span class="variable">$key</span>)));
}
</code></pre>

<p>打印看到Phalcon还注册了以下服务：</p>

<ul>
<li><code>dispatcher</code> : <code>Phalcon\Mvc\Dispatcher</code> 分发服务，将路由命中的结果分发到对应的Controller</li>
<li><code>modelsManager</code> : <code>Phalcon\Mvc\Model\Manager</code> Model管理</li>
<li><code>modelsMetadata</code> : <code>Phalcon\Mvc\Model\MetaData\Memory</code>  ORM表结构</li>
<li><code>response</code> : <code>Phalcon\Http\Response</code>  响应</li>
<li><code>cookies</code> : <code>Phalcon\Http\Response\Cookies</code>  Cookies</li>
<li><code>request</code> : <code>Phalcon\Http\Request</code>  请求</li>
<li><code>filter</code> : <code>Phalcon\Filter</code> 可对用户提交数据进行过滤</li>
<li><code>escaper</code> : <code>Phalcon\Escaper</code>  转义工具</li>
<li><code>security</code> : <code>Phalcon\Security</code>  密码Hash、防止CSRF等</li>
<li><code>crypt</code> : <code>Phalcon\Crypt</code>  加密算法</li>
<li><code>annotations</code> : <code>Phalcon\Annotations\Adapter\Memory</code>  注解分析</li>
<li><code>flash</code> : <code>Phalcon\Flash\Direct</code>  提示信息输出</li>
<li><code>flashSession</code> : <code>Phalcon\Flash\Session</code> 提示信息通过Session延迟输出</li>
<li><code>tag</code> : <code>Phalcon\Tag</code> View的常用Helper</li>
</ul>

<p>而每一个服务都可以通过DI进行替换。接下来实例化一个标准的MVC应用，然后将我们定义好的DI注入进去</p>

<pre><code class="php"><span class="variable">$application</span> = <span class="keyword">new</span> Phalcon\Mvc\Application();
<span class="variable">$application</span>-&gt;setDI(<span class="variable">$di</span>);
</code></pre>

##模块注册阶段

<p>与DI一样，Phalcon建议通过引入一个独立文件的方式注册所有需要的模块：</p>

<pre><code class="php"><span class="keyword">require</span> __DIR__ . <span class="string">'/../config/modules.php'</span>;
</code></pre>

<p>这个文件的内容如下</p>

<pre><code class="php"><span class="variable">$application</span>-&gt;registerModules(<span class="keyword">array</span>(
    <span class="string">'frontend'</span> =&gt; <span class="keyword">array</span>(
        <span class="string">'className'</span> =&gt; <span class="string">'Eva\Frontend\Module'</span>,
        <span class="string">'path'</span> =&gt; __DIR__ . <span class="string">'/../apps/frontend/Module.php'</span>
    )
));
</code></pre>

<p>可以看到Phalcon所谓的模块注册，其实只是告诉框架MVC模块的引导文件<code>Module.php</code>所在位置及类名是什么。</p>

##MVC阶段

<p><code>$application-&gt;handle()</code>是整个MVC的核心，这个函数中处理了路由、模块、分发等MVC的全部流程，处理过程中在关键位置会通过事件驱动触发一系列<code>application:</code>事件，方便外部注入逻辑，最终返回一个<code>Phalcon\Http\Response</code>。整个<a href="https://github.com/phalcon/cphalcon/blob/1.3.2/ext/mvc/application.c#L303"><code>handle</code>方法的过程</a>并不复杂，下面按顺序介绍：</p>

###基础检查

<p>首先检查DI，如果没有任何DI注入，会抛出错误</p>

<blockquote>
  <p>A dependency injection object is required to access internal services</p>
</blockquote>

<p>然后从DI启动EventsManager，并且通过EventsManager触发事件<code>application:boot</code></p>

###路由阶段

<p>接下来进入路由阶段，从DI中获得路由服务<code>router</code>，将uri传入路由并调用路由的<a href="http://docs.phalconphp.com/en/latest/api/Phalcon_Mvc_Router.html"><code>handle()</code>方法</a>。</p>

<p>路由的handle方法负责将一个uri根据路由配置，转换为相应的Module、Controller、Action等，这一阶段接下来会检查路由是否命中了某个模块，并通过<code>Router-&gt;getModuleName()</code>获得模块名。</p>

<p>如果模块存在，则进入模块启动阶段，否则直接进入分发阶段。</p>

<p>注意到了么，在Phalcon中，<strong>模块启动是后于路由的</strong>，这意味着Phalcon的模块功能比较弱，我们无法在某个未启动的模块中注册全局服务，甚至无法简单的在当前模块中调用另一个未启动模块。这可能是Phalcon模块功能设计中最大的问题，解决方法暂时不在本文的讨论范围内，以后会另开文章介绍。</p>

###模块启动

<p>模块启动时首先会触发<code>application:beforeStartModule</code>事件。事件触发后检查模块的正确性，根据<code>modules.php</code>中定义的<code>className</code>、<code>path</code>等，将模块引导文件加载进来，并调用模块引导文件中必须存在的方法</p>

<ul>
<li><code>Phalcon\Mvc\ModuleDefinitionInterface-&gt;registerAutoloaders ()</code></li>
<li><code>Phalcon\Mvc\ModuleDefinitionInterface-&gt;registerServices (Phalcon\DiInterface $dependencyInjector)</code></li>
</ul>

<p><code>registerAutoloaders()</code>用于注册模块内的命名空间实现自动加载。<code>registerServices ()</code>用于注册模块内服务，在官方示例中<code>registerServices ()</code>注册并定义了<code>view</code>服务以及模板的路径，并且注册了数据库连接服务<code>db</code>并设置数据库的连接信息。</p>

<p>模块启动完成后触发 <code>application:afterStartModule</code>事件，进入分发阶段</p>

###分发阶段（Dispatch）

<p>分发过程由<code>Phalcon\Mvc\Dispatcher</code>（分发器）来完成，所谓分发，在Phalcon里本质上是分发器根据路由命中的结果，调用对应的Controller/Action，最终获得Action返回的结果。</p>

<p>分发开始前首先会准备View，虽然View理论上位于MVC的最后一环，但是如果在分发过程中出现任何问题，通常都需要将问题显示出来，因此View必须在这个环节就提前启动。Phalcon没有准备默认的View服务，需要从外部注入，在多模块demo中，View的注入官方推荐在模块启动阶段完成的。如果是单模块应用，则可以在最开始的DI阶段注入。</p>

<p>如果始终没有View注入，会抛出错误</p>

<blockquote>
  <p>Service 'view' was not found in the dependency injection container</p>
</blockquote>

<p>导致分发过程直接中断。</p>

<p>分发需要Dispatcher，Dispatcher同样从DI中取得。然后将router中得到的参数（NamespaceName / ModuleName / ControllerName /  ActionName / Params），全部复制到Dispatcher中。</p>

<p>分发开始前，会调用View的<code>start()</code>方法。具体可以参考<a href="http://docs.phalconphp.com/en/latest/reference/views.html#hierarchical-rendering">View相关文档</a>，其实<code>Phalcon\Mvc\View-&gt;start()</code>就是PHP的输出缓冲函数<code>ob_start</code>的一个简单封装，分发过程中所有输出都会被暂存到缓冲区。</p>

<p>分发开始前还会触发事件<code>application:beforeHandleRequest</code>。</p>

<p>正式开始分发会调用<code>Phalcon\Mvc\Dispatcher-&gt;dispatch()</code>。</p>

###Dispatcher内的分发处理

<p>进入Dispatcher后会发现Dispatcher对整个分发过程进行了进一步细分，并且在分发的过程中会按顺序触发非常多的<a href="http://docs.phalconphp.com/en/latest/reference/dispatching.html#dispatch-loop-events">分发事件</a>，可以通过这些分发事件进行更加细致的流程控制。部分事件提供了可中断的机制，只要返回<code>false</code>就可以跳过Dispatcher的分发过程。</p>

<p>由于分发中可以使用<code>Phalcon\Mvc\Dispatcher-&gt;forward()</code>来实现Action的复用，因此分发在内部会通过循环实现，通过检测一个全局的<code>finished</code>标记来决定是否继续分发。当以下几种情况时，分发才会结束：</p>

<ul>
<li>Controller抛出异常</li>
<li><code>forward</code>层数达到最大（256次）</li>
<li>所有的Action调用完毕</li>
</ul>

###渲染阶段 View Render

<p>分发结束后会触发<code>application:afterHandleRequest</code>，接下来通过<code>Phalcon\Mvc\Dispatcher-&gt;getReturnedValue()</code>取得分发过程返回的结果并进行处理。</p>

<p>由于Action的逻辑在框架外，Action的返回值是无法预期的，因此这里根据返回值是否实现<code>Phalcon\Http\ResponseInterface</code>接口进行区分处理。</p>

####当Action返回一个非<code>Phalcon\Http\ResponseInterface</code>类型

<p>此时认为返回值无效，由View自己重新调度Render过程，会触发<code>application:viewRender</code>事件，同时从Dispatcher中取得ControllerName /  ActionName / Params作为<code>Phalcon\Mvc\View-&gt;render()</code>的入口参数。</p>

<p>Render完毕后调用<code>Phalcon\Mvc\View-&gt;finish()</code>结束缓冲区的接收。</p>

<p>接下来从DI获得resonse服务，将<code>Phalcon\Mvc\View-&gt;getContent()</code>获得的内容置入response。</p>

####当Action返回一个<code>Phalcon\Http\ResponseInterface</code>类型

<p>此时会将Action返回的Response作为最终的响应，不会重新构建新的Response。</p>

###返回响应

<p>通过前面的流程，无论中间经历了多少分支，最终都会汇总为唯一的响应。此时会触发<code>application:beforeSendResponse</code>，并调用</p>

<ul>
<li><code>Phalcon\Http\Response-&gt;sendHeaders()</code></li>
<li><code>Phalcon\Http\Response-&gt;sendCookies()</code></li>
</ul>

<p>将http的头部信息先行发送。至此，<code>Application-&gt;handle()</code>对于请求的处理过程全部结束，对外返回一个<code>Phalcon\Http\Response</code>响应。</p>

###发送响应

<p>HTTP头部发送后一般把响应的内容也发送出去：</p>

<pre><code class="php"><span class="keyword">echo</span> <span class="variable">$application</span>-&gt;handle()-&gt;getContent();
</code></pre>

<p>这就是Phalcon Framework的完整MVC流程。</p>

##流程控制

<p>分析MVC的启动流程，无疑是希望对流程有更好的把握和控制，方法有两种：</p>

###自定义启动

<p>按照上面的流程，我们其实完全可以自己实现<code>$application-&gt;handle()-&gt;getContent()</code>这一流程，下面就是一个简单的替代方案，代码中暂时没有考虑事件的触发。</p>

<pre><code class="php"><span class="comment">//Roter</span>
<span class="variable">$router</span> = <span class="variable">$di</span>[<span class="string">'router'</span>];
<span class="variable">$router</span>-&gt;handle();

<span class="comment">//Module handle</span>
<span class="variable">$modules</span> = <span class="variable">$application</span>-&gt;getModules();
<span class="variable">$routeModule</span> = <span class="variable">$router</span>-&gt;getModuleName();
<span class="keyword">if</span> (<span class="keyword">isset</span>(<span class="variable">$modules</span>[<span class="variable">$routeModule</span>])) {
    <span class="variable">$moduleClass</span> = <span class="keyword">new</span> <span class="variable">$modules</span>[<span class="variable">$routeModule</span>][<span class="string">'className'</span>]();
    <span class="variable">$moduleClass</span>-&gt;registerAutoloaders();
    <span class="variable">$moduleClass</span>-&gt;registerServices(<span class="variable">$di</span>);
}

<span class="comment">//dispatch</span>
<span class="variable">$dispatcher</span> = <span class="variable">$di</span>[<span class="string">'dispatcher'</span>];
<span class="variable">$dispatcher</span>-&gt;setModuleName(<span class="variable">$router</span>-&gt;getModuleName());
<span class="variable">$dispatcher</span>-&gt;setControllerName(<span class="variable">$router</span>-&gt;getControllerName());
<span class="variable">$dispatcher</span>-&gt;setActionName(<span class="variable">$router</span>-&gt;getActionName());
<span class="variable">$dispatcher</span>-&gt;setParams(<span class="variable">$router</span>-&gt;getParams());

<span class="comment">//view</span>
<span class="variable">$view</span> = <span class="variable">$di</span>[<span class="string">'view'</span>];
<span class="variable">$view</span>-&gt;start();
<span class="variable">$controller</span> = <span class="variable">$dispatcher</span>-&gt;dispatch();
<span class="comment">//Not able to call render in controller or else will repeat output</span>
<span class="variable">$view</span>-&gt;render(
    <span class="variable">$dispatcher</span>-&gt;getControllerName(),
    <span class="variable">$dispatcher</span>-&gt;getActionName(),
    <span class="variable">$dispatcher</span>-&gt;getParams()
);
<span class="variable">$view</span>-&gt;finish();

<span class="variable">$response</span> = <span class="variable">$di</span>[<span class="string">'response'</span>];
<span class="variable">$response</span>-&gt;setContent(<span class="variable">$view</span>-&gt;getContent());
<span class="variable">$response</span>-&gt;sendHeaders();
<span class="keyword">echo</span> <span class="variable">$response</span>-&gt;getContent();
</code></pre>

###流程清单

<p>为了方便查找，将整个流程整理为一个树形清单如下：</p>

<ul>
<li>初始化DI (config/services.php) <code>$di = new FactoryDefault();</code>

<ul>
<li>设置路由 <code>$di['router'] = function () {}</code></li>
<li>设置URL <code>$di['url'] = function () {}</code></li>
<li>设置Session <code>$di['session'] = function () {}</code></li>
</ul></li>
<li>初始化Application (public/index.php)

<ul>
<li>实例化App <code>$application = new Application();</code></li>
<li>注入DI <code>$application-&gt;setDI($di);</code></li>
<li>注册模块 (config/modules.php) <code>$application-&gt;registerModules()</code></li>
</ul></li>
<li>启动Application (ext/mvc/application.c)  <code>$application-&gt;handle()</code>

<ul>
<li>检查DI</li>
<li><span class="label label-info">E</span> 触发事件  <code>application:boot</code></li>
<li>路由启动 <code>$di['router']-&gt;handle()</code></li>
<li>获得模块名 <code>$moduleName = $di['router']-&gt;getModuleName()</code>，如果没有则从 <code>$application-&gt;getDefaultModule</code>获取</li>
<li>模块启动 (如果路由命中)

<ul>
<li><span class="label label-info">E</span> 触发事件 <code>application:beforeStartModule</code></li>
<li>调用模块初始化方法 (Module.php) <code>registerAutoloaders()</code> 以及 <code>registerServices()</code></li>
<li><span class="label label-info">E</span> 触发事件 <code>application:afterStartModule</code></li>
</ul></li>
<li>分发

<ul>
<li>初始化View</li>
<li>初始化Dispatcher，将Router中的参数复制到Dispatcher</li>
<li>调用View <code>View-&gt;start()</code>开启缓冲区</li>
<li><span class="label label-info">E</span> 触发事件 <code>application:beforeHandleRequest</code></li>
<li>开始分发 (etc/dispatcher.c) <code>Dispatcher-&gt;dispatch()</code> 

<ul>
<li><span class="label label-info">E</span> 触发事件 <code>dispatch:beforeDispatchLoop</code></li>
<li>循环开始单次分发

<ul>
<li><span class="label label-info">E</span> 触发事件 <code>dispatch:beforeDispatch</code></li>
<li>根据Dispatcher携带的Module、Namespace、Controller、Action获得完整的类与方法名，如果找不到则触发事件 <span class="label label-info">E</span> <code>dispatch:beforeException</code></li>
<li><span class="label label-info">E</span> 触发事件 <code>dispatch:beforeExecuteRoute</code></li>
<li>调用<code>Controller-&gt;beforeExecuteRoute()</code></li>
<li>调用<code>Controller-&gt;initialize()</code></li>
<li><span class="label label-info">E</span> 触发事件 <code>dispatch:afterInitialize</code></li>
<li>调用Action方法</li>
<li><span class="label label-info">E</span> 触发事件 <code>dispatch:afterExecuteRoute</code></li>
<li><span class="label label-info">E</span> 触发事件 <code>dispatch:afterDispatch</code></li>
</ul></li>
<li>Action内如果有forward()，开始下一次分发</li>
</ul></li>
<li><span class="label label-info">E</span> 全部分发结束，触发事件 <code>dispatch:afterDispatchLoop</code></li>
<li>Application获得分发后的输出 <code>$dispatcher-&gt;getReturnedValue()</code></li>
<li><span class="label label-info">E</span> 触发事件 <code>application:afterHandleRequest</code> 分发结束</li>
</ul></li>
<li>渲染，Appliction如果从分发拿到<code>Phalcon\Http\ResponseInterface</code>类型的返回，则渲染直接结束

<ul>
<li><span class="label label-info">E</span> 触发事件 <code>application:viewRender</code> 分发结束</li>
<li>调用 <code>Phalcon\Mvc\View-&gt;render()</code>，入口参数为Dispatcher的 ControllerName / ActionName / Params</li>
<li>调用 <code>Phalcon\Mvc\View-&gt;finish()</code>结束缓冲区的接收</li>
</ul></li>
<li>准备响应

<ul>
<li>将<code>Phalcon\Mvc\View-&gt;getContent()</code>通过<code>Phalcon\Http\Response-&gt;setContent()</code>放入Response</li>
<li><span class="label label-info">E</span> 触发事件 <code>application:beforeSendResponse</code> </li>
<li>调用<code>Phalcon\Http\Response-&gt;sendHeaders()</code>发送头部</li>
<li>调用<code>Phalcon\Http\Response-&gt;sendCookies()</code>发送Cookie</li>
<li>将准备好的响应作为<code>$application-&gt;handle()</code>的返回值返回</li>
</ul></li>
</ul></li>
<li>发送响应

<ul>
<li><code>echo $application-&gt;handle()-&gt;getContent();</code></li>
</ul></li>
</ul>

###MVC事件

<p>Phalcon作为C扩展型的框架，其优势就在于高性能，虽然我们可以通过上一种方法自己实现整个启动，但更好的方式仍然是避免替换框架本身的内容，而使用事件驱动。</p>

<p>下面梳理了整个MVC流程中所涉及的可被监听的事件，可以根据不同需求选择对应事件作为切入点：</p>

<ul>
<li>DI注入

<ul>
<li><code>application:boot</code>  应用启动</li>
</ul></li>
<li>路由阶段

<ul>
<li>模块启动</li>
<li><code>application:beforeStartModule</code> 模块启动前</li>
<li><code>application:afterStartModule</code> 模块启动后</li>
</ul></li>
<li>分发阶段

<ul>
<li><code>application:beforeHandleRequest</code>进入分发器前</li>
<li>开始分发

<ul>
<li><code>dispatch:beforeDispatchLoop</code> 分发循环开始前</li>
<li><code>dispatch:beforeDispatch</code>  单次分发开始前</li>
<li><code>dispatch:beforeExecuteRoute</code>  Action执行前</li>
<li><code>dispatch:afterExecuteRoute</code>  Action执行后</li>
<li><code>dispatch:beforeNotFoundAction</code>  找不到Action</li>
<li><code>dispatch:beforeException</code>  抛出异常前</li>
<li><code>dispatch:afterDispatch</code> 单次分发结束</li>
<li><code>dispatch:afterDispatchLoop</code> 分发循环结束</li>
</ul></li>
<li><code>application:afterHandleRequest</code> 分发结束</li>
</ul></li>
<li>渲染阶段

<ul>
<li><code>application:viewRender</code> 渲染开始前</li>
</ul></li>
<li>发送响应

<ul>
<li><code>application:beforeSendResponse</code>  最终响应发送前</li>
</ul></li>
</ul>



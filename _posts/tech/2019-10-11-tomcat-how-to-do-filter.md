---
layout: post
category : Tech
title : Tomcat源码阅读笔记 - Java Web Filter 调用链
tags : [java, tomcat, sourcecode]
excerpt: Tomcat源码阅读笔记总结，记录在阅读过程中的要点和相关内容分析
---
{% include JB/setup %}

环境:

* Tomcat 9.0.17

`Filter`通过`FilterChain` 调用链调用下一个"filter"，应用可在"filter"的`public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException`方法中调用"filterChain"的相关方法来传递请求到下一个"filter"中。
```java
public interface FilterChain {

    public void doFilter(ServletRequest request, ServletResponse response)
            throws IOException, ServletException;

}
```

`Filter` 调用过程如下图所示，发起请求时`ApplicationFilterChain`负责处理`Filter`和`Servlet`相关逻辑。

<!-- ```plantuml
StandardWrapperValve -> ApplicationFilterFactory: invoke(request, response)
ApplicationFilterFactory -> ApplicationFilterChain: createFilterChain(request, wrapper, servlet)
ApplicationFilterChain -> Filter: doFilter
Filter -> ApplicationFilterChain: doFilter
ApplicationFilterChain -> Servlet: doFilter(调用全部 filter 后)
Servlet -> Servlet:service
```  -->
<!--?xml version="1.0" encoding="UTF-8" standalone="no"?--><svg xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="287px" preserveAspectRatio="none" style="width:829px;height:287px;" version="1.1" viewBox="0 0 829 287" width="829px" zoomAndPan="magnify"><defs><filter height="300%" id="f1vs5839ij43wt" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="95" x2="95" y1="38.4883" y2="247.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="282" x2="282" y1="38.4883" y2="247.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="575" x2="575" y1="38.4883" y2="247.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="697" x2="697" y1="38.4883" y2="247.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="766" x2="766" y1="38.4883" y2="247.3516"></line><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="170" x="8" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="156" x="15" y="23.5352">StandardWrapperValve</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="170" x="8" y="246.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="156" x="15" y="266.8867">StandardWrapperValve</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="177" x="192" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="163" x="199" y="23.5352">ApplicationFilterFactory</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="177" x="192" y="246.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="163" x="199" y="266.8867">ApplicationFilterFactory</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="167" x="490" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="153" x="497" y="23.5352">ApplicationFilterChain</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="167" x="490" y="246.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="153" x="497" y="266.8867">ApplicationFilterChain</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="49" x="671" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="35" x="678" y="23.5352">Filter</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="49" x="671" y="246.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="35" x="678" y="266.8867">Filter</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="60" x="734" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="46" x="741" y="23.5352">Servlet</text><rect fill="#FEFECE" filter="url(#f1vs5839ij43wt)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="60" x="734" y="246.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="46" x="741" y="266.8867">Servlet</text><polygon fill="#A80036" points="270.5,65.7988,280.5,69.7988,270.5,73.7988,274.5,69.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="95" x2="276.5" y1="69.7988" y2="69.7988"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="162" x="102" y="65.0566">invoke(request, response)</text><polygon fill="#A80036" points="563.5,95.1094,573.5,99.1094,563.5,103.1094,567.5,99.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="282.5" x2="569.5" y1="99.1094" y2="99.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="269" x="289.5" y="94.3672">createFilterChain(request, wrapper, servlet)</text><polygon fill="#A80036" points="685.5,124.4199,695.5,128.4199,685.5,132.4199,689.5,128.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="575.5" x2="691.5" y1="128.4199" y2="128.4199"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="48" x="582.5" y="123.6777">doFilter</text><polygon fill="#A80036" points="586.5,153.7305,576.5,157.7305,586.5,161.7305,582.5,157.7305" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="580.5" x2="696.5" y1="157.7305" y2="157.7305"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="48" x="592.5" y="152.9883">doFilter</text><polygon fill="#A80036" points="754,183.041,764,187.041,754,191.041,758,187.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="575.5" x2="760" y1="187.041" y2="187.041"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="159" x="582.5" y="182.2988">doFilter(调用全部 filter 后)</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="766" x2="808" y1="216.3516" y2="216.3516"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="808" x2="808" y1="216.3516" y2="229.3516"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="767" x2="808" y1="229.3516" y2="229.3516"></line><polygon fill="#A80036" points="777,225.3516,767,229.3516,777,233.3516,773,229.3516" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="44" x="773" y="211.6094">service</text></g></svg>



`ApplicationFilterChain`的`internalDoFilter`方法实现上述逻辑。`pos`记录当前"filter"序号，每执行一次`filter.doFilter`，`pos`加1，直到执行完全部的"filter"，然后执行`servlet.service`相关逻辑。也就是说应用可在执行`filterChian.doFilter`完后添加自定义逻辑，即可保证在`servlet.service`之后执行，从而达到修改`response`等对象的目的。

```java
// 调用 filter 和 servlet
// (省略无关代码, 完整代码查看 org.apache.catalina.core.ApplicationFilterChain)
private void internalDoFilter(ServletRequest request,
                                ServletResponse response)
    throws IOException, ServletException {

    // 判断是否需要调用下一个 filter
    // pos 为当前执行到的 filter 序号; n 为 filter 数量
    if (pos < n) {
        // 取出对应序号的 filter，然后加 1
        ApplicationFilterConfig filterConfig = filters[pos++];
        try {
            Filter filter = filterConfig.getFilter();
            ...
            if( Globals.IS_SECURITY_ENABLED ) {
                final ServletRequest req = request;
                final ServletResponse res = response;
                Principal principal =
                    ((HttpServletRequest) req).getUserPrincipal();

                Object[] args = new Object[]{req, res, this};
                // 调用 filter.doFilter
                SecurityUtil.doAsPrivilege ("doFilter", filter, classType, args, principal);
            } else {
                // 调用 filter.doFilter
                filter.doFilter(request, response, this);
            }
        } catch (IOException | ServletException | RuntimeException e) {
            throw e;
        } catch (Throwable e) {
            e = ExceptionUtils.unwrapInvocationTargetException(e);
            ExceptionUtils.handleThrowable(e);
            throw new ServletException(sm.getString("filterChain.filter"), e);
        }
        return;
    }

    try {
        ...
        if ((request instanceof HttpServletRequest) &&
                (response instanceof HttpServletResponse) &&
                Globals.IS_SECURITY_ENABLED ) {
            final ServletRequest req = request;
            final ServletResponse res = response;
            Principal principal =
                ((HttpServletRequest) req).getUserPrincipal();
            Object[] args = new Object[]{req, res};
            // 调用 servlet.service
            SecurityUtil.doAsPrivilege("service",
                                        servlet,
                                        classTypeUsedInService,
                                        args,
                                        principal);
        } else {
            // 调用 servlet.service
            servlet.service(request, response);
        }
    } catch (IOException | ServletException | RuntimeException e) {
        throw e;
    } catch (Throwable e) {
        e = ExceptionUtils.unwrapInvocationTargetException(e);
        ExceptionUtils.handleThrowable(e);
        throw new ServletException(sm.getString("filterChain.servlet"), e);
    } finally {
        if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
            lastServicedRequest.set(null);
            lastServicedResponse.set(null);
        }
    }
}
```
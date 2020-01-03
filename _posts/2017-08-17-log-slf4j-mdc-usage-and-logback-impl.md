---
layout: post
title: "Slf4j MDC使用和基于Logback的实现分析"
date: 2017-08-17 14:01:52
categories: java
tags:
- java
- logging
---
### 一、前言
　　上一篇[Java日志管理](http://zhangyuyu.github.io/log-java/)里面对Java相关的日志库进行了梳理，
了解了常见的日志框架（Log Implement）和日志门面(Log Facade)。

* 使用日志门面，开发者只需要针对门面接口开发，而调用组件的应用程序则可以搭配自己喜好的日志实现工具，目前来看
Slf4j + LogBack组合方式是比较通用的方式。
* 对于web应用而言，多线程的调用是很常见的，我们可能需要对一个用户的操作流程进行归类标记，用来区分哪个行为和哪个日志事件有关，
MDC则很好的处理这个需求。

本篇将介绍Slf4j + LogBack的组合方式，介绍MDC的使用，并结合源码对实现原理进行分析。

### 二、背景
　　起源于最近看到项目使用了MDC，好奇这是个什么东西的情况下，系统的了解了Java的日志管理，并借此机会分析其实现原理，记录下来，理清思路。

<!-- more -->
### 三、Slf4j MDC 介绍
　　介绍Slf4j和MDC的由来，可以参考上一篇[Java日志管理](http://zhangyuyu.github.io/log-java/)。

#### 1. Slf4j是什么？
　　Slf4j全称为Simple Logging Facade for Java (简单日志门面)，作为各种日志框架的简单门面或者抽象，包括JUL、Log4j、Logback。
SLF4J允许用户在部署期间加入自己希望使用的日志系统。其实Slf4j与Log4j, Logback都是同一作者。

#### 2. MDC是什么?
　　MDC全称为Mapped Diagnostic Context（映射调试上下文）是 log4j 和 logback 提供的一种方便在多线程条件下记录日志的功能。
　　典型的例子是 Web 应用服务器。当用户访问某个页面时，应用服务器可能会创建一个新的线程来处理该请求，也可能从线程池中复用已有的线程。
在一个用户的会话存续期间，可能有多个线程处理过该用户的请求。这使得比较难以区分不同用户所对应的日志。
当需要追踪某个用户在系统中的相关日志记录时，就会变得很麻烦。

>　　虽然，Slf4j 是用来适配其他的日志具体实现包的，但是针对 MDC功能，目前只有Logback 以及 Log4j 支持。

#### 3. MDC的简单使用

##### 3.1 使用MDC

```java
package com.daimler.otr;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class LogTest {

    private static final Logger logger = LoggerFactory.getLogger(LogTest.class);

    public static void main(String[] args) {
        MDC.put("THREAD_ID", String.valueOf(Thread.currentThread().getId()));
        logger.info("纯字符串信息的info级别日志");
    }
}

```

##### 3.2 logback的配置

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>[%d{yyyy-MM-dd HH:mm:ss} %highlight(%-5p) %logger.%M\(%F:%L\)] %X{THREAD_ID} %msg%n</pattern>
    </encoder>
  </appender>
  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

##### 3.3 运行结果
```
[2017-08-24 09:08:17 INFO  com.daimler.otr.LogTest.main(LogTest.java:13)] 1 纯字符串信息的info级别日志
```

##### 3.4 其他说明
　　在笔者的web项目中，是将MDC的操作放在Filter中，对所有请求前进行filter拦截，然后加上自定义的唯一标识到MDC中，
就可以在所有日志输出中，清楚看到某用户的操作流程。  
1. 首先有一个单独的MDCRegister定义了对于key需要的put操作。
```java
public class MDCRegister {
    private void registerUUID(String requestId){
        MDC.put("uuid", requestId);
    }

    private void registerModuleName(String requestUri){
        MDC.put("module_name", requestUri);
    }

    private void registerUserName(String userName){
        MDC.put("user_name", userName);
    }

    private void registerAppVersion(String appVersion){
        MDC.put("app_version", appVersion);
    }

    private void registerSessionID(String token){
        String sessionID = Base64.encodeBase64String((token + "300Fi67wJ4A4E0h").getBytes()).substring(0, 8);
        MDC.put("session_id", sessionID);
    }

    public void registerLogInformationFromRequest(String token, String requestId, String appVersion, String requestUri){
        MDC.clear();
        registerUUID(requestId);
        registerAppVersion(appVersion);
        registerSessionID(token);
        registerModuleName(requestUri);
    }
}
```
2. 然后自定义Filter，使用MDC相关操作
```java
@Component
public class CustomAuthenticationFilter extends AbstractPreAuthenticatedProcessingFilter {

    @Autowired
    private MDCRegister mdcRegister;
    
    @Override
    protected Object getPreAuthenticatedPrincipal(HttpServletRequest request) {
        String token = getRequestToken(request);
        String requestID = request.getHeader("X-SERVICE-REQUEST-ID");
        String appVersion = request.getHeader("X-CLIENT-VERSION");
        String requestUri = request.getRequestURI();
        String sessionId = token;
       
        mdcRegister.registerLogInformationFromRequest(sessionId, requestID, appVersion, requestUri);
        
        return request;
    }
}
```
3. 定义logback.xml中的pattern
```xml
<pattern>[V][%d][%-5p][%t][%c{0}][%M][%X{uuid}][%X{app_version}][%X{user_name}][%X{session_id}] - %m%n</pattern>
```

>在日志模板logback.xml 中，使用 %X{ }来占位，替换到对应的 MDC 中 key 的值。

### 四、Log MDC 实现分析
#### 1. Slf4j MDC 实现分析
　　查看Slf4j MDC的实现源码，可以发现Slf4j MDC内部实现很简单：实现一个单例对应实例，获取具体的MDC实现类，然后其对外接口，
就是对参数进行校验，然后调用 MDCAdapter 的方法实现。

　　如下只显示了与mdcAdapter相关的Slf4j MDC源码：
```java
public class MDC {

    static MDCAdapter mdcAdapter;
    
    private MDC() {
    }
    
    static {
        try {
            mdcAdapter = bwCompatibleGetMDCAdapterFromBinder();
        } catch (NoClassDefFoundError ncde) {
            ...
        } catch (Exception e) {
            ...
        }
    }

    public static void put(String key, String val) throws IllegalArgumentException {
        if (key == null) {
            throw new IllegalArgumentException("key parameter cannot be null");
        }
        if (mdcAdapter == null) {
            throw new IllegalStateException("MDCAdapter cannot be null. See also " + NULL_MDCA_URL);
        }
        mdcAdapter.put(key, val);
    }

    public static String get(String key) throws IllegalArgumentException {
        if (key == null) {
            throw new IllegalArgumentException("key parameter cannot be null");
        }

        if (mdcAdapter == null) {
            throw new IllegalStateException("MDCAdapter cannot be null. See also " + NULL_MDCA_URL);
        }
        return mdcAdapter.get(key);
    }

    public static void remove(String key) throws IllegalArgumentException {
        if (key == null) {
            throw new IllegalArgumentException("key parameter cannot be null");
        }

        if (mdcAdapter == null) {
            throw new IllegalStateException("MDCAdapter cannot be null. See also " + NULL_MDCA_URL);
        }
        mdcAdapter.remove(key);
    }

    public static void clear() {
        if (mdcAdapter == null) {
            throw new IllegalStateException("MDCAdapter cannot be null. See also " + NULL_MDCA_URL);
        }
        mdcAdapter.clear();
    }

   
    public static MDCAdapter getMDCAdapter() {
        return mdcAdapter;
    }

}
```

MDCAdapter接口源码如下：
```java
public interface MDCAdapter {
  
    public void put(String key, String val);

    public String get(String key);

    public void remove(String key);

    public void clear();

    public Map<String, String> getCopyOfContextMap();

    public void setContextMap(Map<String, String> contextMap);
}
```

#### 2. Logback MDC 实现分析
　　Logback 中，用LogbackMDCAdapter实现了Sl4j里面的MDCAdapter接口。  
　　下面是get 和 put 的代码实现：

```java
public class LogbackMDCAdapter implements MDCAdapter {
    final ThreadLocal<Map<String, String>> copyOnThreadLocal = new ThreadLocal<Map<String, String>>();

    private static final int WRITE_OPERATION = 1;
    private static final int MAP_COPY_OPERATION = 2;

    final ThreadLocal<Integer> lastOperation = new ThreadLocal<Integer>();

    private Integer getAndSetLastOperation(int op) {
        Integer lastOp = lastOperation.get();
        lastOperation.set(op);
        return lastOp;
    }

    private boolean wasLastOpReadOrNull(Integer lastOp) {
        return lastOp == null || lastOp.intValue() == MAP_COPY_OPERATION;
    }

    private Map<String, String> duplicateAndInsertNewMap(Map<String, String> oldMap) {
        Map<String, String> newMap = Collections.synchronizedMap(new HashMap<String, String>());
        if (oldMap != null) {
            // we don't want the parent thread modifying oldMap while we are
            // iterating over it
            synchronized (oldMap) {
                newMap.putAll(oldMap);
            }
        }

        copyOnThreadLocal.set(newMap);
        return newMap;
    }

    public void put(String key, String val) throws IllegalArgumentException {
        if (key == null) {
            throw new IllegalArgumentException("key cannot be null");
        }

        Map<String, String> oldMap = copyOnThreadLocal.get();
        Integer lastOp = getAndSetLastOperation(WRITE_OPERATION);

        if (wasLastOpReadOrNull(lastOp) || oldMap == null) {
            Map<String, String> newMap = duplicateAndInsertNewMap(oldMap);
            newMap.put(key, val);
        } else {
            oldMap.put(key, val);
        }
    }

    public String get(String key) {
        final Map<String, String> map = copyOnThreadLocal.get();
        if ((map != null) && (key != null)) {
            return map.get(key);
        } else {
            return null;
        }
    }
    // ...
}
```

### 五、 Logback 日志输出实现
　　MDC 的功能实现很简单，就是在线程上下文中，维护一个 Map<String,String> 属性来支持日志输出的时候，
当我们在配置文件logback.xml中配置了%X{key}，则后台日志打印出对应的 key 的值。

#### 1. 初始化
　　所谓初始化，就是我们构建logger的时候。在LoggerFactory.getLogger()，调用的是 slf4j 的方法，而底层使用的是logback的实现。
因此，初始化的重点就是找到底层具体的实现接口，然后构建具体类。

Sl4j中LoggerFactory如下：
```java
package org.slf4j;
public final class LoggerFactory {
    public static Logger getLogger(String name) {
        ILoggerFactory iLoggerFactory = getILoggerFactory();
        return iLoggerFactory.getLogger(name);
    }

    public static ILoggerFactory getILoggerFactory() {
        if (INITIALIZATION_STATE == UNINITIALIZED) {
            synchronized (LoggerFactory.class) {
                if (INITIALIZATION_STATE == UNINITIALIZED) {
                    INITIALIZATION_STATE = ONGOING_INITIALIZATION;
                    performInitialization();
                }
            }
        }
        switch (INITIALIZATION_STATE) {
        case SUCCESSFUL_INITIALIZATION:
            return StaticLoggerBinder.getSingleton().getLoggerFactory();
        case NOP_FALLBACK_INITIALIZATION:
            return NOP_FALLBACK_FACTORY;
        case FAILED_INITIALIZATION:
            throw new IllegalStateException(UNSUCCESSFUL_INIT_MSG);
        case ONGOING_INITIALIZATION:
            return SUBST_FACTORY;
        }
        throw new IllegalStateException("Unreachable code");
    }

    private final static void performInitialization() {
        bind();
        if (INITIALIZATION_STATE == SUCCESSFUL_INITIALIZATION) {
            versionSanityCheck();
        }
    }

     private final static void bind() {
        try {
            Set<URL> staticLoggerBinderPathSet = null;
            if (!isAndroid()) {
                staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
            }
            // the next line does the binding
            StaticLoggerBinder.getSingleton();
            INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
            reportActualBinding(staticLoggerBinderPathSet);
            fixSubstituteLoggers();
            replayEvents();
            SUBST_FACTORY.clear();
        } catch (NoClassDefFoundError ncde) {
            // ...  
        } catch (java.lang.NoSuchMethodError nsme) {
           // ...
        } catch (Exception e) {
           // ...
        }
    }
    private static String STATIC_LOGGER_BINDER_PATH = "org/slf4j/impl/StaticLoggerBinder.class"
}

```
　　上面的部分代码，可以很明显看出，slf4j 会去调用classloader获取当前加载的类中，实现了指定的接口`org/slf4j/impl/StaticLoggerBinder.class`的类，如果多余1个，则会抛出异常。

　　直接在自己的包中实现一个和Slf4j要求路径一样的类，实现对应的接口，然后就可以调用了。

　　例如Logback中，则实现了一个 org.slf4j.impl.StaticLoggerBinder 类，而这个类，在上面的Slf4j的LogFactory中直接被使用`StaticLoggerBinder.getSingleton();`

　　Logback中StaticLoggerBinder如下：
```java
package org.slf4j.impl;

import ch.qos.logback.core.status.StatusUtil;
import org.slf4j.ILoggerFactory;
import org.slf4j.LoggerFactory;
import org.slf4j.helpers.Util;
import org.slf4j.spi.LoggerFactoryBinder;

import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.classic.util.ContextInitializer;
import ch.qos.logback.classic.util.ContextSelectorStaticBinder;
import ch.qos.logback.core.CoreConstants;
import ch.qos.logback.core.joran.spi.JoranException;
import ch.qos.logback.core.util.StatusPrinter;
public class StaticLoggerBinder implements LoggerFactoryBinder {
    private static StaticLoggerBinder SINGLETON = new StaticLoggerBinder();
    static {
        SINGLETON.init();
    }

    private LoggerContext defaultLoggerContext = new LoggerContext();
    private final ContextSelectorStaticBinder contextSelectorBinder = ContextSelectorStaticBinder.getSingleton();

    private StaticLoggerBinder() {
        defaultLoggerContext.setName(CoreConstants.DEFAULT_CONTEXT_NAME);
    }

    public static StaticLoggerBinder getSingleton() {
        return SINGLETON;
    }

    void init() {
        try {
            try {
                new ContextInitializer(defaultLoggerContext).autoConfig();
            } catch (JoranException je) {
               // ...
            }
           // ...
        } catch (Throwable t) {
           // ...
        }
    }
}
```

#### 2. 输出日志模板解析
　　关于logback.xml的解析工作，也是在初始化的时候完成的。

2.1 首先StaticLoggerBinder.init()会执行ContextInitializer的autoConfig():

```java
public class ContextInitializer {
    public void autoConfig() throws JoranException {
            StatusListenerConfigHelper.installIfAsked(loggerContext);
            URL url = findURLOfDefaultConfigurationFile(true);
            if (url != null) {
                configureByResource(url);
            } else {
                // ...
            }
        }

    public void configureByResource(URL url) throws JoranException {
            // ...
            if (urlString.endsWith("groovy")) {
                // ...
            } else if (urlString.endsWith("xml")) {
                JoranConfigurator configurator = new JoranConfigurator();
                configurator.setContext(loggerContext);
                configurator.doConfigure(url);
            } else {
               // ...
            }
        }
}
```

2.2 此后，会执行GenericConfigurator的下述doConfigure方法：

```java
public void doConfigure(final List<SaxEvent> eventList) throws JoranException {
        buildInterpreter();
        // disallow simultaneous configurations of the same context
        synchronized (context.getConfigurationLock()) {
            interpreter.getEventPlayer().play(eventList);
        }
    }
```

2.3 紧接着，执行EventPlayer的play方法，解析xml：

```java
 public void play(List<SaxEvent> aSaxEventList) {
        eventList = aSaxEventList;
        SaxEvent se;
        for (currentIndex = 0; currentIndex < eventList.size(); currentIndex++) {
            se = eventList.get(currentIndex);

            if (se instanceof StartEvent) {
                interpreter.startElement((StartEvent) se);
                // invoke fireInPlay after startElement processing
                interpreter.getInterpretationContext().fireInPlay(se);
            }
            if (se instanceof BodyEvent) {
                // invoke fireInPlay before characters processing
                interpreter.getInterpretationContext().fireInPlay(se);
                interpreter.characters((BodyEvent) se);
            }
            if (se instanceof EndEvent) {
                // invoke fireInPlay before endElement processing
                interpreter.getInterpretationContext().fireInPlay(se);
                interpreter.endElement((EndEvent) se);
            }

        }
    }
```
　　在 Logback 中，解析xml的工作，最后都是交给 Action 和其继承类来完成。在 Action 类中提供了三个方法begin、body和end三个方法，这三个抽象方法中：

* begin 方法负责处理ElementSelector元素的解析；
* body 方法，一般为空，处理文本的；
* end 方法则是处理模板解析的，所以我们的Logback.xml的模板解析实在end方法中。具体是在 NestedComplexPropertyIA类中来解析。其继承Action类，并且其会调用具体的模板解析工具类：PatternLayoutEncoder类和PatternLayout类。

2.4 PatternLayoutEncoder会创建一个PatternLayout对象，然后获取到logback.xml中配置的模板字符串，即`[%d{yyyy-MM-dd HH:mm:ss} %highlight(%-5p) %logger.%M\(%F:%L\)] %X{THREAD_ID} %msg%n`，如配置的节点名一样，其在代码中同样赋值给pattern变量。

```java
public class PatternLayoutEncoder extends PatternLayoutEncoderBase<ILoggingEvent> {
    @Override
    public void start() {
        PatternLayout patternLayout = new PatternLayout();
        patternLayout.setContext(context);
        patternLayout.setPattern(getPattern());
        patternLayout.setOutputPatternAsHeader(outputPatternAsHeader);
        patternLayout.start();
        this.layout = patternLayout;
        super.start();
    }
}
```
　　PatternLayoutEncoder会执行start()方法，然后调用相关方法对pattern进行解析，然后构建一个节点链表，保存这个链表会在日志输出的时使用到。

```java
    public void start() {
        if (pattern == null || pattern.length() == 0) {
           // ...
        }
        try {
            Parser<E> p = new Parser<E>(pattern);
            if (getContext() != null) {
                p.setContext(getContext());
            }
            Node t = p.parse();
            this.head = p.compile(t, getEffectiveConverterMap());
            // ...
        } catch (ScanException sce) {
            // ...
        }
    }
```
2.5 Parse依次遍历pattern字符串，然后把符合要求的字符串放进tokenList中，这个list就维护了我们最终需要输出的模板的格式化模式了。

```java
    public Parser(String pattern, IEscapeUtil escapeUtil) throws ScanException {
        try {
            TokenStream ts = new TokenStream(pattern, escapeUtil);
            this.tokenList = ts.tokenize();
        } catch (IllegalArgumentException npe) {
            throw new ScanException("Failed to initialize Parser", npe);
        }
    }
```

2.6 Compiler会将这个tokenList进行转换，成为我们需要的Node类型的拥有head 和 tail 的链表。
```java
class Compiler<E> extends ContextAwareBase {
     Converter<E> compile() {
        head = tail = null;
        for (Node n = top; n != null; n = n.next) {
            switch (n.type) {
            case Node.LITERAL:
                addToList(new LiteralConverter<E>((String) n.getValue()));
                break;
            case Node.COMPOSITE_KEYWORD:
                CompositeNode cn = (CompositeNode) n;
                CompositeConverter<E> compositeConverter = createCompositeConverter(cn);
                if (compositeConverter == null) {
                    addError("Failed to create converter for [%" + cn.getValue() + "] keyword");
                    addToList(new LiteralConverter<E>("%PARSER_ERROR[" + cn.getValue() + "]"));
                    break;
                }
                compositeConverter.setFormattingInfo(cn.getFormatInfo());
                compositeConverter.setOptionList(cn.getOptions());
                Compiler<E> childCompiler = new Compiler<E>(cn.getChildNode(), converterMap);
                childCompiler.setContext(context);
                Converter<E> childConverter = childCompiler.compile();
                compositeConverter.setChildConverter(childConverter);
                addToList(compositeConverter);
                break;
            case Node.SIMPLE_KEYWORD:
                SimpleKeywordNode kn = (SimpleKeywordNode) n;
                DynamicConverter<E> dynaConverter = createConverter(kn);
                if (dynaConverter != null) {
                    dynaConverter.setFormattingInfo(kn.getFormatInfo());
                    dynaConverter.setOptionList(kn.getOptions());
                    addToList(dynaConverter);
                } else {
                    // if the appropriate dynaconverter cannot be found, then replace
                    // it with a dummy LiteralConverter indicating an error.
                    Converter<E> errConveter = new LiteralConverter<E>("%PARSER_ERROR[" + kn.getValue() + "]");
                    addStatus(new ErrorStatus("[" + kn.getValue() + "] is not a valid conversion word", this));
                    addToList(errConveter);
                }

            }
        }
        return head;
    }
}
```

#### 3. 日志输出分析
　　前面部分进行了初始化配置，紧接着在`logger.info()`的时候，就可以根据初始化得到的Node链表head来解析，遇到%X的时候，
从MDC中获取对应的key值，然后append到日志字符串中，然后输出。

3.1 Logger会执行buildLoggingEventAndAppend方法：
```java
public final class Logger implements org.slf4j.Logger, LocationAwareLogger, AppenderAttachable<ILoggingEvent>, Serializable {
    private void filterAndLog_0_Or3Plus(final String localFQCN, final Marker marker, final Level level, final String msg, final Object[] params,
                        final Throwable t) {

            final FilterReply decision = loggerContext.getTurboFilterChainDecision_0_3OrMore(marker, this, level, msg, params, t);

            if (decision == FilterReply.NEUTRAL) {
                if (effectiveLevelInt > level.levelInt) {
                    return;
                }
            } else if (decision == FilterReply.DENY) {
                return;
            }

            buildLoggingEventAndAppend(localFQCN, marker, level, msg, params, t);
        }

      private void buildLoggingEventAndAppend(final String localFQCN, final Marker marker, final Level level, final String msg, final Object[] params,
                        final Throwable t) {
            LoggingEvent le = new LoggingEvent(localFQCN, this, level, msg, t, params);
            le.setMarker(marker);
            callAppenders(le);
        }

        public void callAppenders(ILoggingEvent event) {
        int writes = 0;
        for (Logger l = this; l != null; l = l.parent) {
            writes += l.appendLoopOnAppenders(event);
            if (!l.additive) {
                break;
            }
        }
        // No appenders in hierarchy
        if (writes == 0) {
            loggerContext.noAppenderDefinedWarning(this);
        }
    }

    private int appendLoopOnAppenders(ILoggingEvent event) {
        if (aai != null) {
            return aai.appendLoopOnAppenders(event);
        } else {
            return 0;
        }
    }
}
```

3.2 继而调用OutputStreamAppender的append方法（由于配置文件配置的是Appender模式）
```java
public class OutputStreamAppender<E> extends UnsynchronizedAppenderBase<E> {
   @Override
    protected void append(E eventObject) {
        if (!isStarted()) {
            return;
        }

        subAppend(eventObject);
    }
    protected void subAppend(E event) {
        if (!isStarted()) {
            return;
        }
        try {
            if (event instanceof DeferredProcessingAware) {
                ((DeferredProcessingAware) event).prepareForDeferredProcessing();
            }
            lock.lock();
            try {
                writeOut(event);
            } finally {
                lock.unlock();
            }
        } catch (IOException ioe) {
            this.started = false;
            addStatus(new ErrorStatus("IO failure in appender", this, ioe));
        }
    }
  protected void writeOut(E event) throws IOException {
        this.encoder.doEncode(event);
    }   
}
```

3.3 按照模板获取值然后转换成字节流输出到后台
```java
    public void doEncode(E event) throws IOException {
        String txt = layout.doLayout(event);
        outputStream.write(convertToBytes(txt));
        if (immediateFlush)
            outputStream.flush();
    }

        public String doLayout(ILoggingEvent event) {
        if (!isStarted()) {
            return CoreConstants.EMPTY_STRING;
        }
        return writeLoopOnConverters(event);
    }

    // 这里的head是上述2.4里面PatternLayoutEncoder里面start方法之中得到的。
    // 这里输出就是按照解析后的链表进行分析输出的。然后根据c类型不同，获取字符串方法也不同
        protected String writeLoopOnConverters(E event) {
        StringBuilder buf = new StringBuilder(128);
        Converter<E> c = head;
        while (c != null) {
            c.write(buf, event);
            c = c.getNext();
        }
        return buf.toString();
    }
```

　　在writeLoopOnConverters方法中，获取对应字符串是不同的，其根据不同的Converter，输出也不同。而Converter的判断，就是根据我们配置的map映射来的。

3.4 MDCConverter的convert实现:
```java
public class MDCConverter extends ClassicConverter {
    public String convert(ILoggingEvent event) {
        Map<String, String> mdcPropertyMap = event.getMDCPropertyMap();

        if (mdcPropertyMap == null) {
            return defaultValue;
        }

        if (key == null) {
            return outputMDCForAllKeys(mdcPropertyMap);
        } else {

            String value = event.getMDCPropertyMap().get(key);
            if (value != null) {
                return value;
            } else {
                return defaultValue;
            }
        }
    }
}
```
　　下面是LoggingEvent中的getMDCPropertyMap，可以看到转换类型是LogbackMDCAdapter。  
　　因此，上述event.getMDCPropertyMap().get(key)就可以从LogbackMDCAdapter（MDC Logback实现）中调用get方法了。
```java
public Map<String, String> getMDCPropertyMap() {
        // populate mdcPropertyMap if null
        if (mdcPropertyMap == null) {
            MDCAdapter mdc = MDC.getMDCAdapter();
            if (mdc instanceof LogbackMDCAdapter)
                mdcPropertyMap = ((LogbackMDCAdapter) mdc).getPropertyMap();
            else
                mdcPropertyMap = mdc.getCopyOfContextMap();
        }
        // mdcPropertyMap still null, use emptyMap()
        if (mdcPropertyMap == null)
            mdcPropertyMap = Collections.emptyMap();

        return mdcPropertyMap;
    }
```

　　自此，伴随着从SLF4J到Logback的运行流程，介绍了Logback MDC的使用：
- 从LoggerFactory.getLogger()讲起
- 初始化找到底层具体的实现接口
- 日志模板解析
- logger.info()实际调用
- 日志输出（打印MDC对应key的value）

### 六、Reference
* [Slf4j MDC 使用和 基于 Logback 的实现分析](https://ketao1989.github.io/2015/04/29/LogBack-Implemention-And-Slf4j-Mdc/)
* [Java日志终极指南](http://www.importnew.com/16331.html)
* https://my.oschina.net/dabird/blog/821868

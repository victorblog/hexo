---
title: Java的常识总结
date: 2017.07.03 10:06
tags:
  - Java
description: AOP的使用
toc: true
copyright: true
---

## 一、实际开发中AOP的使用

背景，为了打印controller的接口信息，以及传参等到log日志文件，方便后期查找问题。

### 1、首先定义一个注解类Log

首先在项目的结构下，创建一个package叫做annotation包

然后新建一个注解类@interface,在IDEA中的new->class中选Annotation项目

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}
```

### 2、指定@Log注解使用的切面

在项目的结构下，新建一个package叫做aop

然后新建一个类LogAspect

```java
/**
 * @Description:日志切面
 * @Author: VictorDan
 * @Date: 19-7-30 上午10:07
 * @Version: 1.0
 */
@Component
@Aspect
public class LogAspect {
    public final static Logger logger = LoggerFactory.getLogger(LogAspect.class);

    /**
     * controller层切点
     */
    @Pointcut("@annotation(com.anjuke.ai.annotation.Log)")
    public void controllerAspect(){

    }

    /**
     * 前置通知，用于拦截controller层记录用户的操作
     * @param joinPoint 切点
     * @throws ClassNotFoundException
     */
    @Before("controllerAspect()")
    public void before(JoinPoint joinPoint) throws Exception {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        logger.info("请求IP：{}",request.getRemoteAddr());
        logger.info("请求路径：{}",request.getRequestURL());
        logger.info("请求方式：{}",request.getMethod());
        logger.info("方法描述：{}",getMethodDescription(joinPoint));
    }

    /**
     * 环绕通知，用于方法执行结束，返回值的打印
     * @param joinPoint
     * @return
     * @throws Throwable
     */
    @Around("controllerAspect()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime=System.currentTimeMillis();
        Object[] args = joinPoint.getArgs();
        Object proceed = joinPoint.proceed(args);
        String result = "null";
        if (proceed != null) {
            result = proceed.toString();
            if (result.length() > 90) {
                result = result.substring(0, 90);
            }
        }
        long endTime=System.currentTimeMillis();
        logger.info("执行时间：{}ms",endTime-startTime);
        logger.info("返回值：{}\n\t", result);
        return proceed;
    }

    /**
     * 异常通知，用来拦截controller层的异常日志
     * @param ex
     */
    @AfterThrowing(throwing = "ex",pointcut = "controllerAspect()")
    public void afterThrowing(Throwable ex){
        logger.error("发生异常：{}",ex.toString());
    }

    /**
     * 获取注解中对方法的描述信息
     * @param point 切点
     * @return 方法描述
     * @throws ClassNotFoundException
     */
    private String getMethodDescription(JoinPoint point) throws ClassNotFoundException {
        String targetName = point.getTarget().getClass().getName();
        String methodName = point.getSignature().getName();
        Object[] args = point.getArgs();
        Class<?> targetClass = Class.forName(targetName);
        Method[] methods = targetClass.getMethods();

        String description="";
        for (Method method:methods) {
            if(method.getName().equals(methodName)){
                Class<?>[] clazz = method.getParameterTypes();
                if(clazz.length==args.length){
                    description=method.getAnnotation(Log.class).value();
                    break;
                }
            }
        }
        return description;
    }

    /**
     * @RequestBody的数据，只能通过流来读取，而controller读取过request的数据，流就会关闭，所以log打印不出。
     * @param httpServletRequest
     * @return
     */
    private String getRequestData(HttpServletRequest httpServletRequest){
        HttpServletRequestWrapper httpServletRequestWrapper = new HttpServletRequestWrapper(httpServletRequest);
        StringBuilder sb = new StringBuilder();
        BufferedReader reader = null;
        InputStreamReader inputStreamReader=null;
        ServletInputStream servletInputStream =null;
        try {
            servletInputStream =  httpServletRequestWrapper.getInputStream();
            inputStreamReader=new InputStreamReader (servletInputStream, Charset.forName("UTF-8"));
            reader = new BufferedReader(inputStreamReader);
            String line = "";
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        } catch (IOException e) {
            logger.error("IO异常：{}",e.getMessage());
        }finally {
            try {
                if(servletInputStream!=null){
                    servletInputStream.close();
                }
                if(inputStreamReader!=null){
                    inputStreamReader.close();
                }
                if(reader!=null){
                    reader.close();
                }
            } catch (IOException e) {
                logger.error("IO异常：{}",e.getMessage());
            }
        }
        return sb.toString ();
    }
}
```

### 3、在Controller中方法的注解使用就行

```java
@PostMapping("/key-data")
    @Log(value = "关键数据")
    public ResponseEntity<Map<String, Object>> getKeyDataDaily(@RequestBody KeyDataDailyRequest request) {
        BrokerUserSession brokerUserSession = getBrokerUserSession();
        Long brokerId = brokerUserSession.getUserId();
        request.setBrokerId(brokerId);
        Map<String, Object> data = service.getKeyDataDaily(request);
        boolean flag = service.isThirtyDailyPmPropZero(request);
        data.put("isZero",flag);
        return resultOK(data);
    }
```


---
title: controller自定义验证
comments: true
aside: true
top_img: false
date: 2022-01-20 10:09:24
tags:
description: controller自定义验证
mathjax:
katex:
categories:
cover: false
---
> 此方法为了解决多个接口需要验证参数时，需要建立多个接参实体类的问题

### 自定义验证
#### 1.代码实现
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

* 代码实现
```java
@Aspect
@Component
@Slf4j
public class ReqApo {

    @Before("@annotation(com.hxy.stepnode.anno.NeVd) && @annotation(n)")
    public void before(JoinPoint point, NeVd n) throws Exception {
        if (!n.vd()) {
            return;
        }
        Object[] domainList = point.getArgs();
        List<Object> checkList;
        if (domainList == null || (checkList = checkDomainType(domainList)).isEmpty()) {
            return;
        }

        CommonVd musicVd = n.ne();
        Class<?> clazz = musicVd.clazz();
        CommonVd.CommonVdEnum commonVdEnum = musicVd.vdName();
        String[] vdMainList = commonVdEnum.getVdMainList();
        if (commonVdEnum.equals(CommonVd.CommonVdEnum.UN) || vdMainList == null) {
            return;
        }
        StringBuilder sb = new StringBuilder();
        for (String name : vdMainList) {
            Field field = clazz.getDeclaredField(name);
            // field 可以在业务层类上定义注解，自定义校验格式，进行拓展
            field.setAccessible(true);
            sb.append(checkMusic(checkList, field, n));
        }
        if (StringUtils.isNotEmpty(sb.toString())) {
            throw new VdException(sb.toString(), ErrorCodeEnum.ERROR_500003);
        }
    }

    private List<Object> checkDomainType(Object[] domainList) {
        List<Object> checkList = new ArrayList<>();
        for (Object domain : domainList) {
            if (domain instanceof BaseEntity) {
                checkList.add(domain);
            }
        }
        return checkList;
    }

    private String checkMusic(List<Object> checkList, Field field, NeVd n) throws Exception {
        field.setAccessible(true);
        for (Object music : checkList) {
            if (field.get(music) == null) {
                return "[" + n.method() + "]" + "[请求参数]:" + "[" + field.getName() + "]";
            }
        }
        return "";
    }
}
public @interface CommonVd {
    Class<? extends BaseEntity> clazz() default BaseEntity.class;

    CommonVdEnum vdName() default CommonVdEnum.UN;

    @Getter
    enum CommonVdEnum {
        music("id", "uix"),
        music2("a", "b"),
        UN,
        ;
        String[] vdMainList;

        CommonVdEnum(String... name) {
            vdMainList = name;
        }
    }
}
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface NeVd {
    boolean vd() default true;

    String method() default "";

    CommonVd ne() default @CommonVd();

}
@NeVd(method = "MusicController.mL()", ne = @CommonVd(clazz = Music.class,
            vdName = CommonVd.CommonVdEnum.music))
@GetMapping("/mL")
public List<Music> mL(Music music) throws Exception {
   log.info("get music list:{}", JSON.toJSONString(music));
   return musicService.musicList();
}

@RestControllerAdvice
public class MyGlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public BaseResponse<String> customException(Exception e) {
        e.printStackTrace();
        return BaseResponse.error();
    }

    @ExceptionHandler(StepException.class)
    @ResponseBody
    public BaseResponse<String> stepException(StepException stepException) {
        stepException.printStackTrace();
        return BaseResponse.error(stepException.getCode(), stepException.getMessage());
    }

    //非检查异常
    //运行时异常
    @ExceptionHandler(VdException.class)
    @ResponseBody
    public BaseResponse<String> vdException(VdException vdException) {
        vdException.printStackTrace();
        return BaseResponse.error(vdException.getCode(), vdException.getMessage());
    }
}
@Data
public class Music extends BaseEntity{
    private Long id;
    private String uix;
    private String name;
    private String path;
    private String like;
}
@Data
public class VdException extends RuntimeException {
    String code;
    String message;

    public VdException(String code, String message) {
        this.code = code;
        this.message = message;
    }

    public VdException(ErrorCodeEnum errorCodeEnum) {
        this.code = errorCodeEnum.getCode();
        this.message = errorCodeEnum.getDesc();
    }

    public VdException(String prefix, ErrorCodeEnum errorCodeEnum) {
        this.code = errorCodeEnum.getCode();
        this.message = prefix + "[异常：]" + errorCodeEnum.getDesc();
    }

    public VdException() {
    }
}
```

#### 2.注意
> 定义全局异常获取RestControllerAdvice时，自定义异常应该继承RuntimeException运行时异常；

* 原因
我们的异常处理类，实际是 动态代理的一个实现。
如果一个异常是检查型异常并且没有在动态代理的接口处声明，那么它将会被包装成UndeclaredThrowableException.
而我们定义的自定义异常，被定义成了检查型异常，导致被包装成了UndeclaredThrowableException

* 解决
抛 java.lang.RuntimeException or java.lang.Error 非检查性异常, 要么接口要声明异常。


# Have fun ^_^
---
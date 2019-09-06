# ffhystrix
#### 介绍
快火熔断器，基于spring boot 2.x 开箱即用
联系方式 qq：625995897 

#### 使用说明
   1、引入jar包
   
        <dependency>
            <groupId>com.smallstep</groupId>
            <artifactId>ffhystrix-boot-starter</artifactId>
            <version>1.0</version>
        </dependency>
        
   2、设置熔断回调
   
    @FFHystrixBean
    public class TestFallback {
    
        @FFHystrixFallback(service = "http://127.0.0.1:8080", pathPatterns = {"GET$/test"}, responseTypeSensitive = false)
        public Object getWithffHystrix() {
            Map<String, String> account = new HashMap<>();
            account.put("name", "李俊");
            return account;
        }
    }
    
    
    
  3、使用
  
    FFHttpClient ffHttpClient = new FFHttpClient();
    // 这里我们设置返回的类型为String，来模拟测试responseTypeSensitive的功效
    Object account = ffHttpClient.getForObject("http://127.0.0.1:8080/test", String.class, name);
    
  4、注解参数说明
        
        @FFHystrixBean：标注在类上，表示该类用于提供hystrix降级回调方法
        @FFHystrixFallback：标注在降级类中的方法上，有2层含义：
        只有符合@ffHystrixFallback注解中的接口地址，FFHttpClient才会使用hystrix方式发起调用。
        如果FFHttpClient调用触发降级，会回调用该注解所在的方法。
        @FFHystrixFallback详细说明：
        service：
        service要和FFHttpClient请求时，参数中url的host一致；
        service支持"${key}"的方式配置，这样会自动引用properties中配置的值，
        service如果配置为空，表示匹配任意的目标服务
        pathPatterns：
        pathPatterns的配置方式为pathPattern（会匹配任意的method）或者method$pathPattern
        pathPatterns支持通配符，？匹配单个字符,*匹配任意字符（不包含/），**匹配任意字符（包含/）
        如果pathPatterns值为空数组，表示匹配任意的path
        responseTypeSensitive：
        默认情况下fallback方法的回参类型，需要和FFHttpClient请求时传入的responseType一致（即使是继承类，也认为是不一样），才会认为该fallback方法匹配本次请求（为了避免类型强制转换错误）。
        但是有些场景下，业务需要做一些通用的fallback来统一处理，这样可以将responseTypeSensitive设置为false，框架就不会校验回参类型是否一致。
        priority：
        该值主要用于FFHttpClient请求的url匹配到多个fallback方法时，通过优先级判断最终使用哪个fallback方法（priority值越大，优先级越高）。
        fallback方法的入参FFHystrixFallbackParam：
        如果fallback方法定义了FFHystrixFallbackParam类型的入参，框架在调用该fallback方法时，就会传入相应的值（主要是一些FFHttpClient请求的入参） 
        circuitBreakerEnabled:
        是否开启熔断器，默认不启动
        circuitBreakerErrorThresholdPercentage:
        开启熔断器时有效。默认:50。当出错率超过50%后熔断器启动
        circuitBreakerRequestVolumeThreshold:
        开启熔断器时有效。熔断器在整个统计时间内是否开启的阀值，默认20。也就是在默认10s内至少请求20次，熔断器才发挥起作用
        circuitBreakerSleepWindowInMilliseconds 
        开启熔断器时有效。熔断时间窗口，默认:5秒.熔断器中断请求5秒后会进入半打开状态,放下一个请求进来重试，如果该请求成功就关闭熔断器，否则继续等待一个熔断时间窗口
#### 配置参数说明
    
    #是否打开熔断降级功能（默认为false）
    ff.hystrix.client.enabled=false 
    #配置>=400的http状态中，哪些不需要触发fallback
    ff.hystrix.client.fallback.ignore.httpStatus=400,403-405
    #FFHttpClient执行出错时，是否打印错误日志（默认为true）
    ff.hystrix.client.log.failedExecution.enabled=true
    #FFHttpClient执行超时时，是否打印错误日志（默认为false）
    ff.hystrix.client.log.timeout.enabled=false
    #FFHttpClient执行被熔断时，是否打印错误日志（默认为false）
    ff.hystrix.client.log.shortCircuit.enabled=false
    #FFHttpClient执行被拒绝时（队列已满），是否打印错误日志（默认为false）
    ff.hystrix.client.log.rejection.enabled=false

#### 错误捕获自定义扩展
    //实现FFCustomErrorHandler类
    //clientHttpResponse为resttemplate返回response
    public class MyCustomErrorHandler implements FFCustomErrorHandler {
        @Override
        public void handleError(ClientHttpResponse clientHttpResponse) throws IOException {
            //todo
        }
    }

#### 结果忽略自定义扩展
    //实现IFallbackIgnore，Exception为请求返回的异常，可进行个性化逻辑判断
    //返回true表示忽略，false进入熔断
    public class MyFallbackIgnore implements IFallbackIgnore {
        @Override
        public boolean isFallbackIgnore(Exception e) {
            //todo 
            return false;
        }
    }


    @Bean
    public MyFallbackIgnore fallbackIgnore() {
            return new MyFallbackIgnore();
        }

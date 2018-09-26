---
layout: post
title: springcloud(ʮһ)����������Zuul�߼�ƪ
category: springcloud
tags: [springcloud]
keywords: springcloud, zuul��·�ɣ�����
excerpt: Spring Cloud Zuul���� Filter���۶ϡ����ԡ��߿��õ�ʹ�÷�ʽ��
---

ʱ����ĺܿ죬д[springcloud(ʮ)����������zuul����ƪ](http://www.ityouknow.com/springcloud/2017/06/01/gateway-service-zuul.html)���ڰ���ǰ�������Ѿ���2018���ˣ����Ǽ���̽��Zuul���߼���ʹ�÷�ʽ��

��ƪ������Ҫ������Zuul����ʹ��ģʽ���Լ��Զ�ת�����ƣ�����ʵZuul���и����Ӧ�ó��������磺��Ȩ������ת��������ͳ�Ƶȵȣ���Щ���ܶ�����ʹ��Zuul��ʵ�֡�

## Zuul�ĺ���

Filter��Zuul�ĺ��ģ�����ʵ�ֶ������Ŀ��ơ�Filter������������4�����ֱ��ǡ�PRE������ROUTING������POST������ERROR���������������ڿ�������ͼ����ʾ��

![](http://www.ityouknow.com/assets/images/2018/springcloud/zuul-core.png)

Zuul�󲿷ֹ��ܶ���ͨ����������ʵ�ֵģ���Щ���������Ͷ�Ӧ������ĵ����������ڡ�

- **PRE��** ���ֹ�����������·��֮ǰ���á����ǿ��������ֹ�����ʵ�������֤���ڼ�Ⱥ��ѡ�������΢���񡢼�¼������Ϣ�ȡ�
- **ROUTING��**���ֹ�����������·�ɵ�΢�������ֹ��������ڹ������͸�΢��������󣬲�ʹ��Apache HttpClient��Netfilx Ribbon����΢����
- **POST��**���ֹ�������·�ɵ�΢�����Ժ�ִ�С����ֹ�����������Ϊ��Ӧ��ӱ�׼��HTTP Header���ռ�ͳ����Ϣ��ָ�ꡢ����Ӧ��΢�����͸��ͻ��˵ȡ�
- **ERROR��**�������׶η�������ʱִ�иù�������
����Ĭ�ϵĹ��������ͣ�Zuul���������Ǵ����Զ���Ĺ��������͡����磬���ǿ��Զ���һ��STATIC���͵Ĺ�������ֱ����Zuul��������Ӧ������������ת������˵�΢����


### Zuul��Ĭ��ʵ�ֵ�Filter

| ���� | ˳�� | ������ | ���� 
| --- | --- | --- | --- 
| pre | -3 | ServletDetectionFilter | ��Ǵ���Servlet������ 
| pre | -2 | Servlet30WrapperFilter | ��װHttpServletRequest���� 
| pre | -1 | FormBodyWrapperFilter | ��װ������ 
| route | 1 | DebugFilter | ��ǵ��Ա�־ 
| route | 5 | PreDecorationFilter | �������������Ĺ�����ʹ�� 
| route | 10 | RibbonRoutingFilter | serviceId����ת�� 
| route | 100 | SimpleHostRoutingFilter | url����ת�� 
| route | 500 | SendForwardFilter | forward����ת�� 
| post | 0 | SendErrorFilter | �����д����������Ӧ 
| post | 1000 | SendResponseFilter | ����������������Ӧ 

**����ָ����Filter**

������application.yml��������Ҫ���õ�filter����ʽ��

``` xml
zuul:
	FormBodyWrapperFilter:
		pre:
			disable: true
```

## �Զ���Filter

ʵ���Զ���Filter����Ҫ�̳�ZuulFilter���࣬���������е�4��������


``` java
public class MyFilter extends ZuulFilter {
    @Override
    String filterType() {
        return "pre"; //����filter�����ͣ���pre��route��post��error����
    }

    @Override
    int filterOrder() {
        return 10; //����filter��˳������ԽС��ʾ˳��Խ�ߣ�Խ��ִ��
    }

    @Override
    boolean shouldFilter() {
        return true; //��ʾ�Ƿ���Ҫִ�и�filter��true��ʾִ�У�false��ʾ��ִ��
    }

    @Override
    Object run() {
        return null; //filter��Ҫִ�еľ������
    }
}
```


## �Զ���Filterʾ��

���Ǽ���������һ����������Ϊ��������Ӧ�Ե����ⲿ����������Ϊ�˱��������ȫ������������Ҫ��������һ�������ƣ����������к���Token����������������ߣ�������󲻴�Token��ֱ�ӷ��ز�������ʾ��

�����Զ���һ��Filter����run()��������֤�����Ƿ���Token��

``` java
public class TokenFilter extends ZuulFilter {

    private final Logger logger = LoggerFactory.getLogger(TokenFilter.class);

    @Override
    public String filterType() {
        return "pre"; // ����������·��֮ǰ����
    }

    @Override
    public int filterOrder() {
        return 0; // filterִ��˳��ͨ������ָ�� ,���ȼ�Ϊ0������Խ�����ȼ�Խ��
    }

    @Override
    public boolean shouldFilter() {
        return true;// �Ƿ�ִ�иù��������˴�Ϊtrue��˵����Ҫ����
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        logger.info("--->>> TokenFilter {},{}", request.getMethod(), request.getRequestURL().toString());

        String token = request.getParameter("token");// ��ȡ����Ĳ���

        if (StringUtils.isNotBlank(token)) {
            ctx.setSendZuulResponse(true); //���������·��
            ctx.setResponseStatusCode(200);
            ctx.set("isSuccess", true);
            return null;
        } else {
            ctx.setSendZuulResponse(false); //���������·��
            ctx.setResponseStatusCode(400);
            ctx.setResponseBody("token is empty");
            ctx.set("isSuccess", false);
            return null;
        }
    }

}
```

��TokenFilter���뵽�������ض��У�����������������´��룺

``` java
@Bean
public TokenFilter tokenFilter() {
	return new TokenFilter();
}
```

�����ͽ������Զ���õ�Filter���뵽�����������С�


**����**

������������ʾ����Ŀ��`spring-cloud-eureka`��`spring-cloud-producer`��`spring-cloud-zuul`�����������Ŀ��Ϊ��һƪʾ����Ŀ��`spring-cloud-zuul`��΢���и��졣

���ʵ�ַ��`http://localhost:8888/spring-cloud-producer/hello?name=neo`�����أ�token is empty ���������ط��ء�  
���ʵ�ַ��`http://localhost:8888/spring-cloud-producer/hello?name=neo&token=xx`�����أ�hello neo��this is first messge��˵������������Ӧ��

ͨ���������������ǿ��Կ��������ǿ���ʹ�á�PRE"���͵�Filter���ܶ����֤��������ʵ��ʹ�������ǿ��Խ��shiro��oauth2.0�ȼ���ȥ����Ȩ����֤��


## ·���۶�

�����ǵĺ�˷�������쳣��ʱ�����ǲ�ϣ�����쳣�׳�������㣬������������Զ�����һ������Zuul�������ṩ��������֧�֡���ĳ����������쳣ʱ��ֱ�ӷ�������Ԥ�����Ϣ��

����ͨ���Զ����fallback���������ҽ���ָ����ĳ��route��ʵ�ָ�route���ʳ�������۶ϴ�����Ҫ�̳�ZuulFallbackProvider�ӿ���ʵ�֣�ZuulFallbackProviderĬ��������������һ������ָ���۶������ĸ�����һ�����Ʒ������ݡ�

``` java
public interface ZuulFallbackProvider {
   /**
	 * The route this fallback will be used for.
	 * @return The route the fallback will be used for.
	 */
	public String getRoute();

	/**
	 * Provides a fallback response.
	 * @return The fallback response.
	 */
	public ClientHttpResponse fallbackResponse();
}
```

ʵ����ͨ��ʵ��getRoute����������Zuul���Ǹ����ĸ�route������۶ϡ���fallbackResponse�������Ǹ��� Zuul ��·����ʱ�������ṩһ��ʲô����ֵ����������

����Spring����չ�˴��࣬�ḻ�˷��ط�ʽ���ڷ��ص�������������쳣��Ϣ��������°汾����ֱ�Ӽ̳���`FallbackProvider` ��

�����������spring-cloud-producer����Ϊ�������������۶Ϸ������ݡ�

``` java
@Component
public class ProducerFallback implements FallbackProvider {
    private final Logger logger = LoggerFactory.getLogger(FallbackProvider.class);

    //ָ��Ҫ����� service��
    @Override
    public String getRoute() {
        return "spring-cloud-producer";
    }

    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("The service is unavailable.".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }

    @Override
    public ClientHttpResponse fallbackResponse(Throwable cause) {
        if (cause != null && cause.getCause() != null) {
            String reason = cause.getCause().getMessage();
            logger.info("Excption {}",reason);
        }
        return fallbackResponse();
    }
}
```

����������쳣ʱ����ӡ����쳣��Ϣ��������"The service is unavailable."��

������Ŀspring-cloud-producer-2����ʱ��������Ļ�������spring-cloud-producer��Ŀ����������Zuul��Ŀ�����ֶ��ر�spring-cloud-producer-2��Ŀ����η��ʵ�ַ��`http://localhost:8888/spring-cloud-producer/hello?name=neo&token=xx`���ύ�淵�أ�

```
hello neo��this is first messge
The service is unavailable.
...
```

���ݷ��ؽ�����Կ�����spring-cloud-producer-2��Ŀ�Ѿ��������۶ϣ�����:```The service is unavailable.```

> Zuul Ŀǰֻ֧�ַ��񼶱���۶ϣ���֧�־��嵽ĳ��URL�����۶ϡ�

## ·������

��ʱ����Ϊ�����������ԭ�򣬷�����ܻ���ʱ�Ĳ����ã����ʱ������ϣ�������ٴζԷ���������ԣ�ZuulҲ������ʵ���˴˹��ܣ���Ҫ���Spring Retry һ����ʵ�֡������������������ĿΪ������ʾ��

**���Spring Retry����**

������spring-cloud-zuul��Ŀ�����Spring Retry������

``` xml
<dependency>
	<groupId>org.springframework.retry</groupId>
	<artifactId>spring-retry</artifactId>
</dependency>
```

**����Zuul Retry**

�������ļ�����������Zuul Retry

``` properties
#�Ƿ������Թ���
zuul.retryable=true
#�Ե�ǰ��������Դ���
ribbon.MaxAutoRetries=2
#�л���ͬServer�Ĵ���
ribbon.MaxAutoRetriesNextServer=0
```

�������ǾͿ�����Zuul�����Թ��ܡ�

**����**

���Ƕ�spring-cloud-producer-2���и��죬��hello��������Ӷ�ʱ�������������һ��ʼ��ӡ������

``` java
@RequestMapping("/hello")
public String index(@RequestParam String name) {
    logger.info("request two name is "+name);
    try{
        Thread.sleep(1000000);
    }catch ( Exception e){
        logger.error(" hello two error",e);
    }
    return "hello "+name+"��this is two messge";
}
```

���� spring-cloud-producer-2��spring-cloud-zuul��Ŀ��

���ʵ�ַ��`http://localhost:8888/spring-cloud-producer/hello?name=neo&token=xx`����ҳ�淵�أ�`The service is unavailable.`ʱ�鿴��Ŀspring-cloud-producer-2��̨��־���£�

```
2018-01-22 19:50:32.401  INFO 19488 --- [io-9001-exec-14] o.s.c.n.z.f.route.FallbackProvider       : request two name is neo
2018-01-22 19:50:33.402  INFO 19488 --- [io-9001-exec-15] o.s.c.n.z.f.route.FallbackProvider       : request two name is neo
2018-01-22 19:50:34.404  INFO 19488 --- [io-9001-exec-16] o.s.c.n.z.f.route.FallbackProvider       : request two name is neo
```

˵�����������ε�����Ҳ���ǽ��������ε����ԡ�����Ҳ����֤�����ǵ�������Ϣ�������Zuul�����Թ��ܡ�

**ע��**

����������ĳЩ�������������ģ����統ѹ������һ��ʵ��ֹͣ��Ӧʱ��·�ɽ�����ת����һ��ʵ�������п��ܵ����������е�ʵ��ȫ��ѹ�塣˵���ף���·��������һ�����þ��Ƿ�ֹ���ϻ���ѹ����ɢ������retry����·����ֻ���ڸ÷��������ʵ�����޷�����������²��������á�����ʱ�򣬶�·������ʽ�������ṩһ���ѺõĴ�����Ϣ�����߼�װ�����������еļ����ʹ���ߡ�

����retry����ʹ�ø��ؾ�����۶ϣ��ͱ��뿼�ǵ��Ƿ��ܹ����ܵ�������ʵ���رպ�eurekaˢ�·����б�֮������Ķ�ʱ����۶ϡ�������Խ��ܣ�������ʹ��retry��


## Zuul�߿���

![](http://www.ityouknow.com/assets/images/2018/springcloud/zuul-case.png)

����ʵ��ʹ��Zuul�ķ�ʽ����ͼ����ͬ�Ŀͻ���ʹ�ò�ͬ�ĸ��ؽ�����ַ�����˵�Zuul��Zuul��ͨ��Eureka���ú�˷�����������������Ϊ�˱�֤Zuul�ĸ߿����ԣ�ǰ�˿���ͬʱ�������Zuulʵ�����и��أ���Zuul��ǰ��ʹ��Nginx����F5���и���ת���Դﵽ�߿����ԡ�

**[ʾ������-github](https://github.com/ityouknow/spring-cloud-examples)**

**[ʾ������-����](https://gitee.com/ityouknow/spring-cloud-examples)**


**�ο���**

[Spring Cloud���ߣ��������� Zuul Filter ʹ��](http://www.ymq.io/2017/12/11/spring-cloud-zuul-filter/)       
[Spring Cloud����������4��- spring cloud zuul](http://tech.lede.com/2017/05/16/rd/server/SpringCloudZuul/)     
[Zuul ·��ʹ��](https://xli1224.github.io/2017/09/09/use-zuul/)    
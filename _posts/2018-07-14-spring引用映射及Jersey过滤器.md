---
layout:     post
title:      Spring & Jersey
subtitle:   Spring Mappers/Jersey Filter
date:       2018-07-14
author:     Young
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - spring
    - jersey
---

>Spring 多模块引用Mapper

# 直接贴代码，简单粗暴

在Spring FrameWork中，子模块依赖于父模块，当子模块调用父模块的Dao来进行增删改查等数据库操作时，会出错为空指针异常或其他异常，主要原因是**在spring-mybatis配置文件中没有显式的将父模块的mapper文件和注解扫描添加进去，将上述两项添加进去之后，子模块即可调用父模块的CRUD操作**，如下是代码

<!-- 自动扫描 -->
     <context:component-scan base-package="org.edi.stocktask.*" />子模块注解扫描
   1.<context:component-scan base-package="org.edi.initialfantasy.*"/>**父模块注解扫描（在配置文件中需要添加)**

<!-- DAO接口所在包名，Spring会自动查找其下的类 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="org.edi.stocktask.mapper" />
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
	</bean>
子模块映射文件所在区域
	2.<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="org.edi.initialfantasy.mapper" />
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
	</bean>
**父模块映射文件所在区域（在配置文件中需要添加)**

（项目多模块利用Maven进行整合，子模块要在pom.xml中添加对父模块的依赖）


>Jersey Filter

# Jersey注解式过滤器（此案例是对请求Token的验证）


@Provider（注解绑定过滤器）
@UserRequest（这个注解是对过滤器的声明，将注解放在其他类或者方法上就能实现对过滤器的引用）
@Priority(Priorities.USER)
public class RequestFilter implements ContainerRequestFilter,ContainerResponseFilter {

    @Autowired
    private TokenVerification tokenVerificate;

    @Override
    public void filter(ContainerRequestContext containerRequestContext) throws IOException {
         //记录请求日志
        try{
            MultivaluedMap<String, String> params = containerRequestContext.getUriInfo().getQueryParameters();
            tokenVerificate.verification(params.getFirst(ServicePath.TOKEN_NAMER));
        }catch (AuthrizationException e){
            Response response = Response.ok(new Result(e)).type(MediaType.APPLICATION_JSON).build();
            throw new WebApplicationException(response);
        }
    }

    @Override
    public void filter(ContainerRequestContext containerRequestContext, ContainerResponseContext containerResponseContext) throws IOException {
        //记录返回日志


    }

}

*注意细节*





















*这篇文章没有引用*


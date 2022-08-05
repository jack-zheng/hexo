---
title: Java Unified Authorization Architecture
date: 2022-07-27 16:57:48
categories:
- java
tags:
- 权限校验
---

前段时间，组内连续出了两个和 SCA 的 role/feature config 相关的 P1 issue，由于不知道它的实现原理，只能通过对比 PR 来找问题，太 low 了，特意读一下他的源码，学习一下下次再出现这种情况要怎么处理

## Unified Authorization Architecture 是什么

这套框架的最终目的是阻止没有授权的用户执行某些 service command。从 MVC 分层上来说，它是在 controller 和 service 之间新加了一层逻辑，每次 service 执行之前都会检测一下，当前用户是否有权执行该 service。

检测分两层，第一层是 company feature test, 第二层是 Role test

## 怎么配置

每个 repo 的 resource 文件夹中都有两个配置文件，分别叫做 featuer-access.xml 和 role-access.xml。他们就是两层检测的配置文件

### Feature-protected Service Command

feature 控制配置文件如下

```xml
<activity-feature-map>
    <module>
        <activity id="com.path.CreateUser">
            <feature id="Manage User"/>
        </activity>
        <!-- ... -->
    </module>
</activity-feature-map>
```

他表示的意思是，对于当前 company，只有开启的 Manage User 这个 feature 才被允许执行 CreateUser 这个 service。所有可用的 feature 都存在 FeatureEnum.java 中。他位于 provisioning 文件夹下，那我估计他是和 provisioning 里的 company feature 挂钩的，之前还以为是和 RBP 的 permission 有关呢。

PS: 但是当前几乎所有的 feature 配置都会默认配置 `<feature id="*"/>`，也就是说，feature check 名存实亡

### Role-protected Service Command

基于 role 的校验可以达到没有配置的用户不能执行 service 的效果

```xml
<activity-permission-map>
    <module>
        <activity-group id="myGroup01">
            <activity id="com.path.CreateUser"/>
        </activity-group>
        <activity-group id="myGroup02">
            <activity id="com.path.DeleteUser"/>
        </activity-group>
    
        <role id="processJob">
            <activity-group refid="myGroup01"/>
        </role>
        <role id="user">
            <activity-group refid="myGroup02"/>
        </role>
    </module>
</activity-permission-map>
```

按照上述配置，login user 只能执行 delete user 这个 service 而不能执行 create user，如果执行了会抛出异常

所有支持爹 role 定义在 PermissionBean 中，也可以去 Permission table 的 permission_type 中查看

### 特殊的 role 

当一个用户通过 /login, /samllogin 登陆到系统中后，对应的 permission 会存到 PermissionListBean 中。但还有其他特殊情况

| User type                 | Role        | Notes                                                |
| :------------------------ | :---------- | :--------------------------------------------------- |
| Provisioners              | provisioner | users authenticated via /provisioning_login.         |
| SFV4 Client or Quartz Job | processJob  | allows the SFV4 client/Quartz Job to execute service |
| Any Login User            | user        | any user authenticated via /login or /samllogin      |

还有一个很神奇的 User type, Anonymous, ParamBean 有一个方法 getAnonymousPrincipal() 注释说，这个 type 表示一个没有经过授权的 user, 猜测，比如，登陆前需要做一些测试，比如看  account 是否存在，这个时候就要用到这种类型的 user type 了。

## 工作流程

https://confluence.successfactors.com/display/ENG/Unified+Authorization+Architecture+Technical+Specification

AppSec, short for application secirity. 看他的描述，是参考了 OWASP ESAPI 这个项目。

This API will provide initially just two things:

* A thread local to store the current user
* A set of access control APIs

The ThreadLocal principal is designed to be populated and cleared by a Servlet filter

逻辑推测，server 启动的时候会加载分析 role/feature 配置文件，并存到一个静态类中，作为共用部分。后续当 SCA engine 调用 impl 之前，都会有一个 auth check 的过程。过程中会使用 AppSec 作为入口检测权限。大致过程就是这样，再细致的分析就得涉及到 RBP 那部分的内容了。最主要的参考文档应该是这个

https://confluence.successfactors.com/display/ENG/Unified+Authorization+Architecture+Technical+Specification

他告诉这个过程的规范，涉及到的类和关系等

public class AppsecConfigListener implements ServletContextListener 这个 class 被注册到 web.xml 中，看实现的接口应该是 servlet 启动结束后，到各个 module 中的加载 feature 和 role 的配置信息。

ProvisioningLoginServlet.java - process() - provisionerBean.createProvisionerParamBean()

  public ParamBean createProvisionerParamBean(CompanyBean cb) {
	ParamBean p = ParamBean.createDefaultParamBean(cb, PROVISIONER_PARAMBEAN_ROLE);
	p.getAppSecRoles().addAll(getAppSecRoles());
	return p;
  }

ParamBean role as provisioner, 并将 sf_provisioner 中 flag 这个 column 里解析出来的 permission 加上 APP_SEC_PREFIX 添加到 role 中

UiAuthenticationProcessorImpl.java - process() - start login

GenericAuthorizationFilter 每次结束之后会将 appsec current user 置为 null


```log
09:35:44,710 ERROR [AuditConfigExecutor] getConfigsFromDb error, company:QAAUTOCAND_RCMGoldenApp7.
sf_class=com.successfactors.appsec.AbstractAccessController line=39 method=assertAuthorizedFor depth=1 
AccessDeniedException{user=[RCMGoldenApp7,RCMGoldenApp7,QAAUTOCAND_RCMGoldenApp7.,dbPool1,null,null,en_US] , activity=Activity{type=SERVICE, name='com.successfactors.auditloggingservice.service.audit.command.AuditConfigQueryCmd', context=null}}
	at com.successfactors.appsec.AbstractAccessController.assertAuthorizedFor(AbstractAccessController.java:39)
	at com.successfactors.appsec.AppSec.assertAuthorizedForService(AppSec.java:231)
	at com.successfactors.sca.ServiceCommandEngine.authorizeService(ServiceCommandEngine.java:312)
	at com.successfactors.sca.ServiceCommandEngine.execute(ServiceCommandEngine.java:238)
	at com.successfactors.sca.service.spring.ServiceCommandProcessorSpring.execute(ServiceCommandProcessorSpring.java:451)
	at com.successfactors.sca.service.handler.spring.SpringAppSCAHandler.execute(SpringAppSCAHandler.java:36)
	at com.successfactors.auditloggingservice.service.audit.executor.AuditConfigExecutor.getConfigsFromDb(AuditConfigExecutor.java:85)
	at com.successfactors.auditloggingservice.service.audit.executor.AuditConfigExecutor.getConfigValue(AuditConfigExecutor.java:55)
	at com.successfactors.auditloggingservice.service.audit.executor.AuditConfigExecutor.getConfigValue(AuditConfigExecutor.java:49)
	at com.successfactors.auditloggingservice.service.audit.executor.AuditKafkaSaveExecutor.needSave(AuditKafkaSaveExecutor.java:118)
	at com.successfactors.auditloggingservice.service.audit.executor.AuditKafkaSaveExecutor.update(AuditKafkaSaveExecutor.java:80)
	at com.successfactors.auditloggingservice.service.audit.executor.AuditKafkaSaveExecutor.lambda$0(AuditKafkaSaveExecutor.java:69)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.base/java.lang.Thread.run(Unknown Source)
```


## 怎么测试

针对之前的两个 P1 issue 应该怎么写测试

## 市面上的同类型产品


https://confluence.successfactors.com/display/CST/12+SCA+Home 这个页面可以应该是 最全的 sca 文档了

ESAPI

<feature id="Generic Objects"/>
<activity-feature-map>
    <module>
        <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.SavePendingData">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.RemovePendingData">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.domainevent.incoming.personassignment.sca.ProcessPersonCreate">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.domainevent.incoming.personassignment.sca.SyncPersonDataAndClearPendingData">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.FindScimPendingDataEntityByAttributeNameAndValue">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.FindScimPendingDataEntitiesByUserResourceId">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.FindScimUserResourceByAccountId">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.FindScimUserResourceByPerPersonUuid">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.CreateScimUserResource">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.UpdateScimUserResource">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.DeleteScimUserResource">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.sca.SearchUserResourceV2">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.sca.CreateUserResourceV2">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.sca.UpdateUserResourceV2">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.sca.FindUserResourceV2ById">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.service.sca.DeleteUserResourceV2">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.ScimUserResourceMigration">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.realtimeidentitysync.identityconfiguration.dbmigration.CreateIdentityRealTimeSyncIgsConfiguration">
            <feature id="*"/>
        </activity>
        <activity id="com.successfactors.identityprofile.technicaluser.service.sca.IPSRealTimeSyncTechnicalUserPermissions">
            <feature id="*"/>
        </activity>
    </module>
</activity-feature-map>


<activity-permission-map>
    <module>
        <activity-group id="PendingData">
            <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.SavePendingData"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.RemovePendingData"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.FindScimPendingDataEntityByAttributeNameAndValue"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.pendingdata.FindScimPendingDataEntitiesByUserResourceId"/>
        </activity-group>
        <activity-group id="personAssignmentDataHandler">
            <activity id="com.successfactors.identityprofile.scimv2.domainevent.incoming.personassignment.sca.SyncPersonDataAndClearPendingData"/>
            <activity id="com.successfactors.identityprofile.scimv2.domainevent.incoming.personassignment.sca.ProcessPersonCreate" />
        </activity-group>
        <activity-group id="ScimUserResoure">
            <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.FindScimUserResourceByAccountId"/>
            <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.FindScimUserResourceByPerPersonUuid"/>
            <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.CreateScimUserResource"/>
            <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.UpdateScimUserResource"/>
            <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.DeleteScimUserResource"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.sca.SearchUserResourceV2"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.sca.CreateUserResourceV2"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.sca.UpdateUserResourceV2"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.sca.FindUserResourceV2ById"/>
            <activity id="com.successfactors.identityprofile.scimv2.service.sca.DeleteUserResourceV2"/>
            <activity id="com.successfactors.identityprofile.scimv2.scimuserresource.sca.ScimUserResourceMigration"/>
        </activity-group>
        <activity-group id="igsintegration">
            <activity id="com.successfactors.identityprofile.realtimeidentitysync.identityconfiguration.dbmigration.CreateIdentityRealTimeSyncIgsConfiguration"/>
            <activity id="com.successfactors.identityprofile.technicaluser.service.sca.IPSRealTimeSyncTechnicalUserPermissions"/>
        </activity-group>

        <role id="processJob">
            <activity-group refid="PendingData"/>
            <activity-group refid="ScimUserResoure"/>
            <activity-group refid="personAssignmentDataHandler"/>
            <activity-group refid="igsintegration"/>
        </role>
        <role id="user">
            <activity-group refid="PendingData"/>
            <activity-group refid="ScimUserResoure"/>
            <activity-group refid="personAssignmentDataHandler"/>
            <activity-group refid="igsintegration"/>
        </role>
        <role id="anonymous">
            <activity-group refid="PendingData"/>
            <activity-group refid="ScimUserResoure"/>
            <activity-group refid="personAssignmentDataHandler"/>
        </role>
    </module>
</activity-permission-map>


a user which has not been authenticated.

curl --location --request GET 'http://localhost:8080/rest/iam/scim/v2/Users/0bfa4ec0-a35d-4f64-881f-ed759108ff47'


AppsecConfigListener

ServiceCommandEngine

<activity-group refid="DynamicGroups"/>
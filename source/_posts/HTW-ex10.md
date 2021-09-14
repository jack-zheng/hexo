---
title: Ex10 安全
date: 2021-09-13 16:42:16
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 10** covers web application security constraints for restricting access to certain contents. You will learn entities related to security such as principals, roles, login config, authenticators, etc. You will also write two applications that install an authenticator valve in the StandardContext object and uses basic authentication to authenticate users.

主体和之前的一样，新加的内容是安全相关的东西，更具体来说，是授权相关。可以根据配置的账户密码信息限制用户访问。这个功能现在应该挺鸡肋了，因为一般的 App 都是将这部分功能坐在内部的 login service 中的，哪里会通过这种方式作授权啊，除非买现成的但是不提供授权服务，这也太蠢了吧。。。

问题：Bootstrap2 中貌似做了一次授权之后，会将信息 cache 起来，看看它是存在哪里的

## Overview

有些网站服务需要有访问限制，Tomcat 可以通过配置文件达到这种效果，访问页面时只有输入正确的用户名密码之后才能访问。

Tomcat 有一个 authenticator valve 可以用来做授权，他在系统启动后加入 context 的 pipeline 中，他会在 wrapper valve 之前被调用，做用户验证。

## Realm

Tomcat 中 realm 模块可以做用户验证。一个 context 只能有一个 realm 服务，我们可以通过 context 的 setRealm() 方法设置它。

Realm 中用户信息存放的地址由配置决定，默认情况下，Tomcat 会拿 conf/tomcat-users.xml 中的用户信息做比对。当然我们也可以配置其他数据源，比如 DB。

Catalina 中使用 org.apache.catalina.Realm 这个接口表示这个概念，核心就那四个授权方法

```java
public interface Realm {
    public Principal authenticate(String username, String credentials);
    public Principal authenticate(String username, byte[] credentials);
    public Principal authenticate(String username, String digest,
                                  String nonce, String nc, String cnonce,
                                  String qop, String realm,
                                  String md5a2);
    public Principal authenticate(X509Certificate certs[]);
}
```

同时这个接口还包含 `public boolean hasRole(Principal principal, String role);` 方法。这个接口有一个抽象实现 org.apache.catalina.realm.RealmBase 还有几个具体实现都在同一个包下：JDBCRealm, JNDIRealm, MemoryRealm, and UserDatabaseRealm。默认使用的是 MemoryRealm，当 server 启动时，他会读取 tomcat-users.xml。

## GenericPrincipal

java.security.Principal 代表 Principal 这个概念，具体实现为 org.apache.catalina.realm.GenericPrincipal。GenericPrincipal 必须关联一个 realm, 构造函数如下

```java
public GenericPrincipal(Realm realm, String name, String password) {
    this(realm, name, password, null);
}

public GenericPrincipal(Realm realm, String name, String password, List roles) {
    super();
    this.realm = realm;
    this.name = name;
    this.password = password;
    if (roles != null) {
        this.roles = new String[roles.size()];
        this.roles = (String[]) roles.toArray(this.roles);
        if (this.roles.length > 0)
            Arrays.sort(this.roles);
    }
}
```

Principal 中也包含 `hasRole()` 方法，你可以传入 `*` 作为参数检测是否包含任意 role 的意思。

## LoginConfig

login config 包含 realm name 由 org.apache.catalina.deploy.LoginConfig 这个 final class 表示. LoginConfig 包含 realm 和 authentication 的信息，auth name 必须是 BASIC, DIGEST, FORM, or CLIENT-CERT。

当服务器启动的时候，Tomcat 会读取 web.xml 信息，如果 xml 包含 login-config 元素，tomcat 就会创建一个 LoginConfig 对象并为他设置属性。authentication valve 会调用 LoginConfig 的 getRealmName() 方法并传送给浏览器的登陆界面。

## Authenticator

org.apache.catalina.Authenticator 是 authenticator 的表现类，他没有任何方法，只是一个壳子。有一个抽象的实现类 org.apache.catalina.authenticator.AuthenticatorBase，它还集策划给你了 ValveBase 表明它是一个 valve。具体的实现类由 BasicAuthenticator, FormAuthenticator, DigestAuthentication 和 SSLAuthenticator。如果没有具体指明 authentication 类型，则会默认使用 NonLoginAuthenticator。它表示只检测安全限制而不需要授权。

{% plantuml %}
interface Valve
class ValveBase
interface Authenticator

Valve <|.. ValveBase
ValveBase -[hidden] Authenticator

class SingleSignOn
class AuthenticatorBase

ValveBase <|-- SingleSignOn
ValveBase <|-- AuthenticatorBase
Authenticator <|.. AuthenticatorBase

AuthenticatorBase <|-- BasicAuthenticator
AuthenticatorBase <|-- SSLAuthenticator
AuthenticatorBase <|-- FormAuthenticator
AuthenticatorBase <|-- DigestAuthenticator
AuthenticatorBase <|-- NonLoginAuthenticator
{% endplantuml %}

## Installing the Authenticator Valve

login-config element 在 deployment 文件中只能出现一次，其中包含有 auth-method 元素。这意味着 context 中只能有一个 LoginConfig 实例并且只能有一个 authentication class 实现。

| Type        | Impl                |
| :---------- | :------------------ |
| BASIC       | BasicAuthenticator  |
| FORM        | FormAuthenticator   |
| DIGEST      | DigestAuthenticator |
| CLIENT-CERT | SSLAuthenticator    |

如果 auth-method 没有设置，就表示使用的是 NonLoginAuthenticator。org.apache.catalina.startup.ContextConfig 是 Context 的配置类，包含 authentication 信息。下面的例子中，我们使用 SimpleContextConfig 动态加载 BasicAuthenticator 作为 StandardContext 的配置项。

## The Applications

### Bootsrap1

第一个例子，没有使用配置文件，而是直接在 Bootstrap 类中做了设置。新建了一个 SimpleContextConfig，它是一个 Listener，添加到 context 的监听器列表中，当 context start 时，接受到 event 并配置 context

```java
// Bootstrap1 中代码如下
LifecycleListener listener = new SimpleContextConfig();
((Lifecycle) context).addLifecycleListener(listener);
//...
((Lifecycle) context).start();
```

实现如下,  context start 后，event 触发，在 listener 中拿到对应的 context，并进行 auth 相关的设置，内容包括

* 设置 login config
* 设置 authenticator 到 context 的 pipeline

```java
public class SimpleContextConfig implements LifecycleListener {

    private Context context;

    public void lifecycleEvent(LifecycleEvent event) {
        if (Lifecycle.START_EVENT.equals(event.getType())) {
            context = (Context) event.getLifecycle();
            authenticatorConfig();
            context.setConfigured(true);
        }
    }

    private synchronized void authenticatorConfig() {
        // Does this Context require an Authenticator?
        SecurityConstraint constraints[] = context.findConstraints();
        if ((constraints == null) || (constraints.length == 0))
            return;
        LoginConfig loginConfig = context.getLoginConfig();
        if (loginConfig == null) {
            loginConfig = new LoginConfig("NONE", null, null, null);
            context.setLoginConfig(loginConfig);
        }

        // Has an authenticator been configured already?
        Pipeline pipeline = ((StandardContext) context).getPipeline();
        if (pipeline != null) {
            Valve basic = pipeline.getBasic();
            if ((basic != null) && (basic instanceof Authenticator))
                return;
            Valve valves[] = pipeline.getValves();
            for (int i = 0; i < valves.length; i++) {
                if (valves[i] instanceof Authenticator)
                    return;
            }
        } else { // no Pipeline, cannot install authenticator valve
            return;
        }

        // Has a Realm been configured for us to authenticate against?
        if (context.getRealm() == null) {
            return;
        }

        // Identify the class name of the Valve we should configure
        String authenticatorName = "org.apache.catalina.authenticator.BasicAuthenticator";
        // Instantiate and install an Authenticator of the requested class
        Valve authenticator = null;
        try {
            Class authenticatorClass = Class.forName(authenticatorName);
            authenticator = (Valve) authenticatorClass.newInstance();
            ((StandardContext) context).addValve(authenticator);
            System.out.println("Added authenticator valve to Context");
        } catch (Throwable t) {
        }
    }
}
```

接着在 Bootstrap1 中设置 security constraint 相关的配置. 这里指定了 constraint 中只有 role 是 manager 的可以访问

```java
// add constraint
SecurityCollection securityCollection = new SecurityCollection();
securityCollection.addPattern("/");
securityCollection.addMethod("GET");

SecurityConstraint constraint = new SecurityConstraint();
constraint.addCollection(securityCollection);
constraint.addAuthRole("manager");
LoginConfig loginConfig = new LoginConfig();
loginConfig.setRealmName("Simple Realm");
// add realm
Realm realm = new SimpleRealm();

context.setRealm(realm);
context.addConstraint(constraint);
context.setLoginConfig(loginConfig);
```

SimpleRealm 实现如下, 它其实就是模拟了一个内存中的 DB，当开启 security constrain 后，通过 GET 访问页面就会跳出验证弹窗。输入账户信息，就会调用到下面 authenticate() 方法中做判断了

```java
public class SimpleRealm implements Realm {

    public SimpleRealm() {
        createUserDatabase();
    }

    private Container container;
    private ArrayList users = new ArrayList();

    public Container getContainer() {
        return container;
    }

    public void setContainer(Container container) {
        this.container = container;
    }

    public String getInfo() {
        return "A simple Realm implementation";
    }

    public void addPropertyChangeListener(PropertyChangeListener listener) {
    }

    public Principal authenticate(String username, String credentials) {
        System.out.println("SimpleRealm.authenticate()");
        if (username == null || credentials == null)
            return null;
        User user = getUser(username, credentials);
        if (user == null)
            return null;
        return new GenericPrincipal(this, user.username, user.password, user.getRoles());
    }

    public Principal authenticate(String username, byte[] credentials) {
        return null;
    }

    public Principal authenticate(String username, String digest, String nonce,
                                  String nc, String cnonce, String qop, String realm, String md5a2) {
        return null;
    }

    public Principal authenticate(X509Certificate certs[]) {
        return null;
    }

    public boolean hasRole(Principal principal, String role) {
        if ((principal == null) || (role == null) ||
                !(principal instanceof GenericPrincipal))
            return (false);
        GenericPrincipal gp = (GenericPrincipal) principal;
        if (!(gp.getRealm() == this))
            return (false);
        boolean result = gp.hasRole(role);
        return result;
    }

    public void removePropertyChangeListener(PropertyChangeListener listener) {
    }

    private User getUser(String username, String password) {
        Iterator iterator = users.iterator();
        while (iterator.hasNext()) {
            User user = (User) iterator.next();
            if (user.username.equals(username) && user.password.equals(password))
                return user;
        }
        return null;
    }

    private void createUserDatabase() {
        User user1 = new User("ken", "blackcomb");
        user1.addRole("manager");
        user1.addRole("programmer");
        User user2 = new User("cindy", "bamboo");
        user2.addRole("programmer");

        users.add(user1);
        users.add(user2);
    }

    class User {

        public User(String username, String password) {
            this.username = username;
            this.password = password;
        }

        public String username;
        public ArrayList roles = new ArrayList();
        public String password;

        public void addRole(String role) {
            roles.add(role);
        }

        public ArrayList getRoles() {
            return roles;
        }
    }

}
```

启动服务器，当我们用 role 是 manager 的 user 去访问，页面显示正常，当我们用 programer 去访问，页面不显示

流程大致描述如下

1. 启动 server 加载配置
2. 访问页面
3. context 调用 pipeline
4. pipeline 调用 valve
5. 调用 BasicAuthenticator 的 auth 方法验证 - 在 listener 中指定

## Bootstrap2

第二个例子和第一个例子很想，唯一区别就是将 Realm 的配置指定到了 tomcat-users.xml 文件

```java
// add realm
Realm realm = new SimpleUserDatabaseRealm();
((SimpleUserDatabaseRealm) realm).createDatabase("conf/tomcat-users.xml");
```

SimpleUserDatabaseRealm 实现如下, 里面一个比较有意思的点是 MemoryUserDatabase 这个类，它会默认加载 conf/tomcat-users.xml 的内容，解析出来，格式是 hard code 的，挺有意思。逻辑和之前的基本一样，没什么新鲜的。

```java
public class SimpleUserDatabaseRealm extends RealmBase {

    protected UserDatabase database = null;
    protected static final String name = "SimpleUserDatabaseRealm";

    protected String resourceName = "UserDatabase";

    public Principal authenticate(String username, String credentials) {
        // Does a user with this username exist?
        User user = database.findUser(username);
        if (user == null) {
            return (null);
        }

        // Do the credentials specified by the user match?
        // FIXME - Update all realms to support encoded passwords
        boolean validated = false;
        if (hasMessageDigest()) {
            // Hex hashes should be compared case-insensitive
            validated = (digest(credentials).equalsIgnoreCase(user.getPassword()));
        }
        else {
            validated = (digest(credentials).equals(user.getPassword()));
        }
        if (!validated) {
            return null;
        }

        ArrayList combined = new ArrayList();
        Iterator roles = user.getRoles();
        while (roles.hasNext()) {
            Role role = (Role) roles.next();
            String rolename = role.getRolename();
            if (!combined.contains(rolename)) {
                combined.add(rolename);
            }
        }
        Iterator groups = user.getGroups();
        while (groups.hasNext()) {
            Group group = (Group) groups.next();
            roles = group.getRoles();
            while (roles.hasNext()) {
                Role role = (Role) roles.next();
                String rolename = role.getRolename();
                if (!combined.contains(rolename)) {
                    combined.add(rolename);
                }
            }
        }
        return (new GenericPrincipal(this, user.getUsername(),
                user.getPassword(), combined));
    }

    // ------------------------------------------------------ Lifecycle Methods


    /**
     * Prepare for active use of the public methods of this Component.
     */
    protected Principal getPrincipal(String username) {
        return (null);
    }

    protected String getPassword(String username) {
        return null;
    }

    protected String getName() {
        return this.name;
    }

    public void createDatabase(String path) {
        database = new MemoryUserDatabase(name);
        ((MemoryUserDatabase) database).setPathname(path);
        try {
            database.open();
        }
        catch (Exception e)  {
        }
    }
}
```

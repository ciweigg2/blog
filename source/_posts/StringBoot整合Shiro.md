cover: http://ciwei2.cn-sh2.ufileos.com/125.jpg
title: SpringBoot整合Shiro
date: 2018-07-26 17:33:00
tags: [shiro]
categories: [综合]
---
### 首先第一步引入
  <!--shiro权限控制框架-->
  ```java
        <dependency>           
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.3.2</version>
        </dependency>
```
<!--more-->
### 添加配置类
* 安全管理器(在管理器中添加自己的验证密码和权限的方法)
```java
   @Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(myShiroRealm());
        return securityManager;
    }
```

* 配置拦截链
拦截链的意思，就是给url赋值权限

```java
/**
     * ShiroFilterFactoryBean 处理拦截资源文件问题。
     * 注意：单独一个ShiroFilterFactoryBean配置是或报错的，以为在
     * 初始化ShiroFilterFactoryBean的时候需要注入：SecurityManager
     *
     * Filter Chain定义说明 1、一个URL可以配置多个Filter，使用逗号分隔 2、当设置多个过滤器时，全部验证通过，才视为通过
     * 3、部分过滤器可指定参数，如perms，roles
     *
     */
    @Bean
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 必须设置 SecurityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 如果不设置默认会自动寻找Web工程根目录下的"admin登录页面"页面
        shiroFilterFactoryBean.setLoginUrl("/admin/login");
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 未授权界面;
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");

        // 拦截器.
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        filterChainDefinitionMap.put("/admin/login", "anon");//登录页面
        //TODO 跟登录权限,添加权限test测试。
        filterChainDefinitionMap.put("/admin/index", "authc,perms[" +"test" + "]");//校验密码和权限
        // 配置退出过滤器,其中的具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/logout", "logout");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }
```

### 实现Realm
* doGetAuthenticationInfo
校验密码
```java
/**
     * 校验用户名和密码
     *
     * @param authcToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
        logger.debug("身份认证方法：MyShiroRealm.doGetAuthenticationInfo()");
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) authcToken;
        //TODO 根据用户名和用户密码判断用户，用户验证成功，就把用户名和用户密码放行
        String userName = usernamePasswordToken.getUsername();
        Admin user = mongoDao.findOneByQuery(Admin.class, "userName", usernamePasswordToken.getUsername());
        String pwd = String.valueOf(usernamePasswordToken.getPassword());
        if (ObjectUtils.isEmpty(user)){
            throw new IncorrectCredentialsException();
        }
        if (StringUtils.endsWithIgnoreCase(user.getPassword(), pwd)) {
            return new SimpleAuthenticationInfo(userName, pwd, getName());
        }
        return null;
    }
```

* doGetAuthorizationInfo
在本方法中,查询用户的所有权限，然后添加

```java
   /**
     * 权限链配置
     * 在shiro配置类中把资源对应的权限都加载到应用中
     *
     * 在本方法中,查询用户的所有权限，然后添加
     *
     * @param principals
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        logger.debug("##################执行Shiro权限认证##################");
        //获取当前登录输入的用户名，等价于
        String userName = (String) super.getAvailablePrincipal(principals);
        logger.debug("##################开始查询用户【" + userName + "】的权限##################");
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //根据每个用户名获得对应的权限列表
        //根据用户名获取用户的权限
        info.addStringPermission("test");
        return info;
    }
```
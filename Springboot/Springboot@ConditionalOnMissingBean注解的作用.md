# Springboot~@ConditionalOnMissingBean注解的作用

1. @ConditionalOnMissingBean

   　　它是修饰bean的一个注解，主要实现的是，当你的bean被注册之后，如果而注册相同类型的bean，就不会成功，它会保证你的bean只有一个，即你的实例只有一个，当你注册多个相同的bean时，会出现异常，以此来告诉开发人员。

2. @Primary标识哪个是默认的bean


   ```java
　 @Bean
    public AMapper aMapper1(AConfig aConfig) {
        return n@Bean
    public AMapper aMapper1(AConfig aConfig) {
        return new AMapperImpl1(aConfig);
    }

    @Bean
    @Primary
    public AMapper aMapper2(AConfig aConfig) {
        return new AMapperImpl2(aConfig);
   ```

3. @ConditionalOnProperty

   通过其三个属性prefix,name以及havingValue来实现的，其中prefix表示配置文件里节点前缀，name用来从application.properties中读取某个属性值，havingValue表示目标值。

   - 如果该值为空，则返回false;
   - 如果值不为空，则将该值与havingValue指定的值进行比较，如果一样则返回true;否则返回false。
   - 返回值为false，则该configuration不生效；为true则生效。

   例如在配置文件中配置ocp.fast.rabbitmq.enable=true,才会注册ConnectionFactory这个bean
   
   ![img](https://img2020.cnblogs.com/blog/1546302/202103/1546302-20210304104456470-1708441949.png)

4. 其它注释及总结

   - @ConditionalOnBean // 当给定的在bean存在时,则实例化当前Bean，这个bean可能由于某种原因而没有注册到ioc里，这时@ConditionalOnBean可以让当前bean也不进行注册
   - @ConditionalOnMissingBean // 当给定的在bean不存在时,则实例化当前Bean，感觉这个是在多态环境下使用，当一个接口有多个实现类时，如果只希望它有一个实现类，那就在各个实现类上加上这个注解
   - @ConditionalOnClass // 当给定的类名在类路径上存在，则实例化当前Bean
   - @ConditionalOnMissingClass // 当给定的类名在类路径上不存在，则实例化当前Bean

参考：

- https://www.cnblogs.com/lori/p/13490005.html

- [Springboot~@ConditionalOnMissingBean注解的作用 - donleo123 - 博客园 (cnblogs.com)](https://www.cnblogs.com/donleo123/p/14478730.html)
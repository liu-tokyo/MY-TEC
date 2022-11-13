# Spring Boot - AOP（面向切面）-切入点表达式

> https://www.cnblogs.com/fnlingnzb-learner/p/16636129.html

参考：

- https://www.cnblogs.com/li3807/p/9002683.html
- https://blog.csdn.net/ycf921244819/article/details/106599489/

## 1. 类型匹配通配符

切入点指示符用来指示切入点表达式目的，在 Spring AOP 中目前只有执行方法这一个连接点，Spring AOP 支持的 AspectJ 切入点指示符，切入点表达式可以使用 &&、||、！来组合切入点表达式，还可以使用类型匹配的通配符来进行匹配。

- 类型通配符如下：

  | 类型匹配通配符 | 说明                                                         |
  | -------------- | ------------------------------------------------------------ |
  | *              | 表示匹配任何数量字符。示例：java.*.String，表示匹配 java 包下的任何"一级子包"下的 String 类型； 如匹配 java.lang.String，但不匹配java.lang.ss.String |
  | ..             | 表示任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。示例：java..*  ，表示匹配java包及任何子包下的任何类型;  如匹配java.lang.String、java.lang.annotation.Annotation |
  | +              | 仅能作为后缀放在类型模式后边，匹配指定类型的子类型；         |

## 2. 通配符详细说明

- **execution**：用于匹配方法执行的连接点，配置切入点示例 @Pointcut("execution(切入点表达式)")，切入点表达式格式如下：

  ```
  execution(annotation? modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
  ```

  | 切入点表达式示例                         | 说明                                                         |
  | ---------------------------------------- | ------------------------------------------------------------ |
  | public * *(..)                           | 任何公共方法的                                               |
  | @org.lixue.EnableLogTrace public * *(..) | 使用 org.lixue.EnableLogTrace 注解标注的任何公共方法         |
  | * org.lixue..LogTrace+.*()               | org.lixue 包及所有子包下 LogTrace接口及子类型的的任何无参方法 |

- **within**：用于匹配指定类型内的方法执行；配置切入点示例：

  ```
  @Pointcut("within(切入点表达式)")
  ```

  | 切入点表达式示例          | 说明                                                        |
  | ------------------------- | ----------------------------------------------------------- |
  | org.lixue..*              | 在 org.lixue 包或所有子包的任何方法执行                     |
  | org.lixue..AccountService | 在 org.lixue 包或所有子包下 AccountService 类型的任何方法   |
  | org.lixue..LogTrace+      | 在 org.lixue 包或所有子包下 LogTrace 类型及子类型的任何方法 |

- **this**：用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；注意this中使用的表达式必须是类型全限定名，不支持通配符。配置切入点示例：

  ```
  @Pointcut("this(org.lixue.LogTrace)")
  ```

  | 切入点表达式示例   | 说明                                                      |
  | ------------------ | --------------------------------------------------------- |
  | org.lixue.LogTrace | AOP代理对象的类型实现了 org.lixue.LogTrace 接口的任何方法 |

- **target**：用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；注意target中使用的表达式必须是类型全限定名，不支持通配符；配置切入点示例：

  ```
  @Pointcut("target(org.lixue.LogTrace)")
  ```

  | 切入点表达式示例   | 说明                                     |
  | ------------------ | ---------------------------------------- |
  | org.lixue.LogTrace | 实现了 org.lixue.LogTrace 接口的任何方法 |

- **args**：用于匹配当前执行的方法传入的参数为指定类型的执行方法；参数类型列表中的参数必须是类型全限定名，通配符不支持；args属于动态切入点，这种切入点开销非常大，非特殊情况最好不要使用；注意：匹配传入的参数类型，不是匹配方法签名的参数类型。，配置切入点示例：

  ```
  @Pointcut("args(java.lang.String,java.lang.String)")
  ```

  | 切入点表达式示例                        | 说明                                                         |
  | --------------------------------------- | ------------------------------------------------------------ |
  | args (java.io.Serializable,..)          | 任何一个以接受"传入参数类型为 java.io.Serializable" 开头，且其后可跟任意个任意类型的参数的方法执行 |
  | args(java.lang.String,java.lang.String) | 任何一个以接受传入两个参数并且类型为 java.lang.String        |

- **@within**：用于匹配所以持有指定注解类型内的方法；配置切入点示例 @Pointcut("@within(注解类型)")，注解类型也必须是全限定类型名

  | 切入点表达式示例         | 说明                                                         |
  | ------------------------ | ------------------------------------------------------------ |
  | org.lixue.EnableLogTrace | 使用 org.lixue.EnableLogTrace 注解的任何类型的任何方法必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 |

- **@target**：用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；配置切入点示例@Pointcut("@target(注解类型)")，注解类型也必须是全限定类型名

  | 切入点表达式示例         | 说明                                                         |
  | ------------------------ | ------------------------------------------------------------ |
  | org.lixue.EnableLogTrace | 使用 org.lixue.EnableLogTrace 注解的任何类型的任何方法必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 |

- **@args**：用于匹配当前执行的方法传入的参数持有指定注解的执行；

  | 切入点表达式示例         | 说明                                                         |
  | ------------------------ | ------------------------------------------------------------ |
  | org.lixue.EnableLogTrace | 使用 org.lixue.EnableLogTrace 注解的任何类型的任何方法必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 |

- **@annotation**：用于匹配当前执行方法持有指定注解的方法，配置切入点示例 @Pointcut("@annotation(注解类型)")，注解类型也必须是全限定类型名

  | 切入点表达式示例         | 说明                                         |
  | ------------------------ | -------------------------------------------- |
  | org.lixue.EnableLogTrace | 使用 org.lixue.EnableLogTrace 注解的任何方法 |
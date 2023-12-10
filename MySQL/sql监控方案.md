# 方式一：实现agent进行方法代理

https://blog.csdn.net/it__zhou/article/details/130089318

该方式较为底层，不用修改源代码，但是实现更为复杂

# 方式二：使用AOP方式监控SQL

https://blog.51cto.com/u_16213448/7215631

该方式需要对监控的方法加上注解，但是实现较为简单

# 方式三：基于MySQL自身的慢SQL监控

https://zhuanlan.zhihu.com/p/612358167?utm_id=0

https://blog.csdn.net/lovelichao12/article/details/123465147

https://www.xjx100.cn/news/9005.html?action=onClick

监控内容较为全面，比如可以查看哪些查询没用到索引

# 方式四：基于Druid监控

https://zhuanlan.zhihu.com/p/612358167?utm_id=0
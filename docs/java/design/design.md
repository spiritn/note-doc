# 工厂模式

工厂模式，简单工厂模式，抽象工厂模式

角色：抽象产品，具体产品；抽象工厂，具体工厂；上下文。

## 简单工厂

产品有抽象层，但是工厂没有抽象层，只有一个工厂中根据传参创建相应的产品。

如果要新增产品，需要修改工厂方法逻辑，不符合开闭原则。因为工厂方法是静态的，也叫静态工厂模式。

## 工厂模式

产品有抽象层，工厂也有抽象层，即一个工厂只能创建一个产品类别。

如果新增产品，只需要新增工厂实现，不需要对工厂逻辑进行修改，符合开闭原则。

![工厂模式](images/工厂模式.png)

相比简单工厂模式内部决定生产哪种产品，工厂模式中决定生成哪种产品的逻辑在外部客户端选择了什么样的工厂。



缺点：

工厂模式中，如果要多一个抽象产品就需要新建立一个抽象工厂，且需要组合使用多个抽象产品时，会比较混乱。



## 抽象工厂

产品有抽象层，工厂有抽象层，一个工厂可以创建多个产品。这多个产品是有关联的

用于解决有多产品类别(芯片、主板、显卡)情况下，产品族(A产品族、B产品族)的切换替代问题。



![抽象工厂模式](images/抽象工厂模式.png)

产品族：如java课程 ，Python课程。如果要增加一个C语言课程，会很容易。

产品类别：如article，Video。如果要增加一个产品等级如source，那改动的地方就很多，原来的工厂java，Python都需要增加source的方法。不符合开闭原则。

总结：工厂方法模式针对的是一个产品类别，而抽象工厂模式针对的是多个产品类别。

```java
// java.sql.Connection接口 就是抽象工厂
public interface Connection  extends Wrapper, AutoCloseable {
	//...    	
    	//返回普通的sql执行器
        Statement createStatement() throws SQLException;
		//返回具有参数化预编译功能的sql执行器
    	PreparedStatement prepareStatement(String sql) throws SQLException;
    	//返回可以执行存储过程的sql执行器
        CallableStatement prepareCall(String sql) throws SQLException;   
    //...         
}
```


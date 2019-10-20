# DesignMode
### 单例模式

#### 优点：

—— 由于单例模式只生成一个实例，减少了系统性能开销，当一个对象的产生需要比较多的资源时，如读取配置、产生其他依赖对象时，则可以通过在应用启动时直接产生一个单例对象，然后永久驻留内存的方式来解决

——单例模式可以在系统设置全局的访问点，优化环共享资源访问，例如可以设计一个单例类，负责所有数据表的映射处理



常见的五种实现方式：

——主要：

- 饿汉式（线程安全，调用频率高。但是，不能延时加载。）	

- 懒汉式（线程安全，调用频率不高。但是，可以延时加载。）

——其他：

- 双重检测锁式（由于JVM底层内部模型原因，偶尔会出现问题。不建议使用）
- 静态内部类式（线程安全，调用效率高。但是，可以延时加载）
- 枚举单例（线程安全，调用效率高，不能延时加载.可以天然地防止反射和反序列化漏洞！）

```Java
/**
 * 测试饿汉式单例模式
 */
public class SingletonDemo1 {
    //类初始化时，立即加载这个对象.加载类时，天生就是线程安全的！
    private static SingletonDemo1 instance = new SingletonDemo1();

    //私有化构造器
    private SingletonDemo1(){
    }

    //方法没有同步，调用效率高
    public static SingletonDemo1 getInstance(){
        return instance;
    }
}
```

```Java
/**
 * 测试懒汉式单例模式
 */
public class SingletonDemo2 {
    //类初始化时，不初始化这个对象。（延时加载，真正用的时候再创建）
    private static SingletonDemo2 instance;

    //私有化构造器
    private SingletonDemo2(){
    }

    //方法同步，调用效率低
    public static synchronized SingletonDemo2 getInstance(){
        if (instance == null){
            instance = new SingletonDemo2();
        }
        return instance;
    }
}
```

```Java
/**
 * 测试静态内部类实现单例模式
 */
public class SingletonDemo3 {

    private static class SingletonClassInstance{
        private static SingletonDemo3 instance;
    }

    //私有化构造器
    private SingletonDemo3(){
    }

    //方法没有同步，调用效率高
    public static SingletonDemo3 getInstance(){
        return SingletonClassInstance.instance;
    }
}
```

```Java
/**
 * 测试枚举式实现单例模式（没有延时加载）
 */
public enum SingletonDemo4 {

    //此枚举本身就是一个单例对象
    INSTANCE;

    //添加自己所需要的操作！
    public void singletonOperation(){

    }
}
```

#### 问题：

—— 反射可以破解上面几种（不包含枚举式）实现方式（可以在构造方法中手动抛出异常控制）

——反序列化可以破解上面集中（不包含枚举式）实现方式

- 可以通过定义readResolve()防止获得不同对象。
  - 反序列化时，如果对象所在类定义了readResolve()，（实际是一种回调），定义返回哪个对象。

```Java
public class Client {
    public static void main(String[] args) throws Exception {
        SingletonDemo5 s1 = SingletonDemo5.getInstance();
        SingletonDemo5 s2 = SingletonDemo5.getInstance();

        System.out.println(s1);
        System.out.println(s2);

        //通过反射的方式直接调用私有构造器
//        Class<SingletonDemo5> clazz = (Class<SingletonDemo5>) Class.forName("test.SingletonDemo5");
//        Constructor<SingletonDemo5> c = clazz.getDeclaredConstructor(null);
//        c.setAccessible(true);
//        SingletonDemo5 s3 = c.newInstance();
//        SingletonDemo5 s4 = c.newInstance();
//
//        System.out.println(s3);
//        System.out.println(s4);

        //通过反序列化的方式构造多个对象
        FileOutputStream fos = new FileOutputStream("d:/a.txt");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(s1);
        oos.close();
        fos.close();

        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("d:/a.txt"));
        SingletonDemo5 s3 = (SingletonDemo5) ois.readObject();
        System.out.println(s3);
    }
}



/**
 * 测试懒汉式单例模式（如何防止反射和反序列化漏洞）
 */
public class SingletonDemo5 implements Serializable {
    //类初始化时，不初始化这个对象。（延时加载，真正用的时候再创建）
    private static SingletonDemo5 instance;

    //私有化构造器
    private SingletonDemo5(){
        if (instance != null){
            throw new RuntimeException();
        }
    }

    //方法同步，调用效率低
    public static synchronized SingletonDemo5 getInstance(){
        if (instance == null){
            instance = new SingletonDemo5();
        }
        return instance;
    }

    //反序列化时，如果定义了readResolve()，则直接返回此方法指定的对象。而不需要单独在创建新对象！
    private Object readResolve() throws ObjectStreamException{
        return instance;
    }
}
```


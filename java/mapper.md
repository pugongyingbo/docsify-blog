

#### 解释 
* 在Mybatis提供的编程接口中，开发人员只需要定义好Mapper接口(如：UserDao)，开发人员无需去实现。Mybatis会利用JDK的动态代理实现 Mapper接口。
* 在Mybatis中，每个Mapper接口都会对应一个MapperProxyFactory对象实例，这个对应关系在Configuration.mapperRegistry.knownMappers中。
* 当getMapper()方法被调用时，Mybatis会找到相对应的MapperProxyFactory对象实例，利用这个工厂来创建一个jdk动态代理对象，是这个Mapper接口的实现类,当Mapper定义的方法被调用时，会调用MapperProxy来处理。
* MapperProxy会根据方法找到对应的MapperMethod对象来实现这次调用。
MapperMethod对应会读取方法中的注解，从Configuration中找到相对应的MappedStatement对象，再执行。

#### 源码
* DefaultSqlSession.getMapper()方法最终会调用MapperRegistry.getMapper()

```
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        //这个MapperProxyFactory是调用addMapper方法时加到knownMappers中的，
        MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory)this.knownMappers.get(type);
        if (mapperProxyFactory == null) {
        //生成一个MapperProxy对象 
            throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
        } else {
            try {
            //生成一个MapperProxy对象 
                return mapperProxyFactory.newInstance(sqlSession);
            } catch (Exception var5) {
                throw new BindingException("Error getting mapper instance. Cause: " + var5, var5);
            }
        }
    }
```

* MapperProxyFactory的newInstance()

```
  protected T newInstance(MapperProxy<T> mapperProxy) {
      //mapperInterface，说明Mapper接口被代理了，这样子返回的对象就是Mapper接口的子类，方法被调用时会被mapperProxy拦截,也就是执行mapperProxy.invoke()方法  
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
    }

    public T newInstance(SqlSession sqlSession) {
    //创建一个Mapperxy对象，这个方法实现了JDK动态代理中的InvocationHandler接口
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }
```

* MapperProxy

```
public class MapperProxy<T> implements InvocationHandler, Serializable {
    private static final long serialVersionUID = -6424540398559729838L;
    private final SqlSession sqlSession;
    private final Class<T> mapperInterface;
    
    //Mapper接口中的每个方法都会生成一个MapperMethod对象, methodCache维护着他们的对应关系  
  //这个methodCache是在MapperProxyFactory中持有的，MapperProxyFactory又是在Configuration中持有的  
  //所以每个Mapper接口类对应的MapperProxyFactory和methodCache在整个应用中是共享的，一般只会有一个实例 
    private final Map<Method, MapperMethod> methodCache;

    public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
        this.sqlSession = sqlSession;
        this.mapperInterface = mapperInterface;
        this.methodCache = methodCache;
    }
    //这里会拦截Mapper接口(UserDao)的所有方法
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
        //如果是Object中定义的方法，直接执行。如toString(),hashCode()等。
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);
            }

            if (this.isDefaultMethod(method)) {
                return this.invokeDefaultMethod(proxy, method, args);
            }
        } catch (Throwable var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
        //其他Mapper接口定义的方法交由mapperMethod来执行。
        MapperMethod mapperMethod = this.cachedMapperMethod(method);
        return mapperMethod.execute(this.sqlSession, args);
    }

    private MapperMethod cachedMapperMethod(Method method) {
        MapperMethod mapperMethod = (MapperMethod)this.methodCache.get(method);
        if (mapperMethod == null) {
            mapperMethod = new MapperMethod(this.mapperInterface, method, this.sqlSession.getConfiguration());
            this.methodCache.put(method, mapperMethod);
        }

        return mapperMethod;
    }

    @UsesJava7
    private Object invokeDefaultMethod(Object proxy, Method method, Object[] args) throws Throwable {
        Constructor<Lookup> constructor = Lookup.class.getDeclaredConstructor(Class.class, Integer.TYPE);
        if (!constructor.isAccessible()) {
            constructor.setAccessible(true);
        }

        Class<?> declaringClass = method.getDeclaringClass();
        return ((Lookup)constructor.newInstance(declaringClass, 15)).unreflectSpecial(method, declaringClass).bindTo(proxy).invokeWithArguments(args);
    }

    private boolean isDefaultMethod(Method method) {
        return (method.getModifiers() & 1033) == 1 && method.getDeclaringClass().isInterface();
    }
}
```

* MapperMethod的excute()

```
//所有Mapper接口中方法被调用里，都会执行这个方法.这里实际上是调用SqlSession中的相关方法,
    public Object execute(SqlSession sqlSession, Object[] args) {
        Object param;
        Object result;
         //判断这个方法为注解里的哪个Sql类型
        switch(this.command.getType()) {
        case INSERT:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.insert(this.command.getName(), param));
            break;
        case UPDATE:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.update(this.command.getName(), param));
            break;
        case DELETE:
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.delete(this.command.getName(), param));
            break;
        case SELECT:
            if (this.method.returnsVoid() && this.method.hasResultHandler()) {
               //没有返回值，并且有ResultHandler的情况 this.executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (this.method.returnsMany()) {
            //返回一个List  
                result = this.executeForMany(sqlSession, args);
            } else if (this.method.returnsMap()) {
            //返回一个Map 
                result = this.executeForMap(sqlSession, args);
            } else if (this.method.returnsCursor()) {
                result = this.executeForCursor(sqlSession, args);
            } else {
              //返回一个对象 
                param = this.method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(this.command.getName(), param);
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + this.command.getName());
        }

        if (result == null && this.method.getReturnType().isPrimitive() && !this.method.returnsVoid()) {
        //结果为空，是否为原始类型，有返回值
            throw new BindingException("Mapper method '" + this.command.getName() + " attempted to return null from a method with a primitive return type (" + this.method.getReturnType() + ").");
        } else {
            return result;
        }
    }

```

* 参考 https://blog.csdn.net/ashan_li/article/details/50404350
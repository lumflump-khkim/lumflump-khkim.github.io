# Comment Pre Interceptora
- mapper.sqlid 를 자동을 달아주는 interceptor

```java
@Slf4j
@Intercepts({@Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class})})
public class CommentPreInterceptor implements Interceptor {
 
 
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
 
        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
                 // Elegant access to the properties of the object through MetaObject, here is the property to access the StatementHandler;: MetaObject is a one provided by Mybatis for convenience,
                 // Elegant access to the object properties of the object, through which you can simplify the code, do not need try / catch various reflective exceptions, and it supports the operation of three types of objects JavaBean, Collection, Map.
        MetaObject metaObject = MetaObject
            .forObject(statementHandler, SystemMetaObject.DEFAULT_OBJECT_FACTORY, SystemMetaObject.DEFAULT_OBJECT_WRAPPER_FACTORY,
                new DefaultReflectorFactory());
                 // First intercept the RoutingStatementHandler, which has a delegateHandler type delegate variable, its implementation class is BaseStatementHandler, and then to the BaseStatementHandler member variable mappedStatement
        MappedStatement mappedStatement = (MappedStatement) metaObject.getValue("delegate.mappedStatement");
                 //id is the full path name of the executed mapper method, such as com.uv.dao.UserMapper.insertUser
        String id = mappedStatement.getId();
                 //sql statement type select, delete, insert, update
//		log.info("sql Id : "+id);
//        log.info("sql Id : "+id.substring(id.lastIndexOf(".")-1)+"."+id.substring(id.lastIndexOf(".")));
        String[] arrId = id.split("\\.");
        String comment = arrId[arrId.length-2]+"."+arrId[arrId.length-1];
//        log.info("comment : "+comment);
//        String sqlCommandType = mappedStatement.getSqlCommandType().toString();
//		log.info("sql type : "+sqlCommandType);

                 // Database connection information
//        Configuration configuration = mappedStatement.getConfiguration();
//        ComboPooledDataSource dataSource = (ComboPooledDataSource)configuration.getEnvironment().getDataSource();
//        dataSource.getJdbcUrl();
 
        BoundSql boundSql = statementHandler.getBoundSql();
                 // Get the original sql statement
        String sql = boundSql.getSql();

        String[] arrSql = sql.split("\\n");

//        for(int i=0;i<arrSql.length;i++){
//            log.info(i+":"+arrSql[i]);
//        }
        arrSql[0] = arrSql[0]+" /* " + comment + " */ " ;
        String mSql = StringUtils.join(arrSql,"\n");
//        log.info("mSql = " +mSql);
                 // Modify the sql statement by reflection
        Field field = boundSql.getClass().getDeclaredField("sql");
        field.setAccessible(true);
        field.set(boundSql, mSql);
 
        return invocation.proceed();
    }
 
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
 
    @Override
    public void setProperties(Properties properties) {
                 // Here you can receive the property parameter of the configuration file
//        System.out.println(properties.getProperty("name"));
    }
    ```

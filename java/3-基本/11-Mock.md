
# Mock 测试实际运用场景
- 解决前后端依赖，或者 架构中每一层之间的依赖
- 解决外部接口依赖
- 当需要调用某层API并且这个API现在还不可用

# 为什么会有mock
TDD：测试驱动开发，其思想为根据需求抽象接口，在编写测试用例，然后再开始编写开发代码。在TDD开发模式中总是先写单元测试模块，之后再进行开发。
DDD：领域驱动开发


创建一个虚假的对象。

# 常用注解
```java
    // 修饰被Mock的类，Mock对象自动注入
    // 快速的创建mock
    @Mock
```

```java
    // 修饰被测试的类，不能是接口，创建这个类的对象并自动标记为 @Mock,@Spy等注解的属性值 注入到这个类中
    @InjectMocks
```

```java
    // 在执行测试之前执行被注接的方法
    @BeforeEach //junit5
    @Before //junit4
```

```java
    // 在执行测试之后执行被注解的方法
    @AfterEach //junit5
    @After //junit4
```

```java
    // Spy对象会走对象真是的方法，而mock对象不会
    // Spy作用于对象实例，而Mock作用于 .class
    @Spy
```

# mock static，finnal方法
mvn依赖: inline 与 core 不能同时存在，会有问题
```xml
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-inline</artifactId>
            <version>4.6.1</version>
            <scope>test</scope>
        </dependency>
```
## static 方法
```java
public class StaticMethod {
    public static List<Integer> range(int s, int e) {
        List<Integer> list = new ArrayList<>();
        for (int i = s; i <= e; i++) {
            list.add(i);
        }
        return list;
    }
    public static String name() {
        return "name";
    }
}
```
```java
public class StaticMethodTest {
    @Test
    public void range() {
        // Mockito.mockStatic 使用完了需要关闭，否则启动多个Test会产生问题。原因是Mockito使用了ThreadLocal中，用完必须得释放
        try (MockedStatic<StaticMethod> mockStatic = Mockito.mockStatic(StaticMethod.class)) {
            // stubbing
            mockStatic.when(() -> StaticMethod.range(2, 5)).thenReturn(new ArrayList<Integer>(Arrays.asList(2, 3, 4, 5)));
            // using
            Assert.assertTrue(StaticMethod.range(2, 5).contains(3));
        }
    }
    @Test
    public void name() {
        try (final MockedStatic<StaticMethod> mockStatic = Mockito.mockStatic(StaticMethod.class)) {
            mockStatic.when(StaticMethod::name).thenReturn("hello you");

            System.out.println(StaticMethod.name());
        }
    }
}
```

## final 方法


# mock 问题
mock 知识拷贝一个，拷贝和创建有什么区别？


# 代码覆盖率的问题
注意：打桩的api及代码参数 要和 执行的代码参数完全一直，否则就会报错，mock执行后的返回值就会是默认值。

```java
public class DaoException extends RuntimeException {
}
```
```java
public class FindUtils {
    public static String getAreaCode(String number) {
        return "a";
    }

    public static String getOperatorCode(String number) {
        return "b";
    }
}
```
```java
public interface RegistrationService {
    String register(String name, String phone) throws ValidationException;
}
```
```java
public class RegistrationServiceImpl implements RegistrationService {
    private static final String pattern = "";
    @Resource
    SalesDao salesDao;
    @Resource
    UserDao userDao;

    @Override
    public String register(String name, String phone) throws ValidationException {
        // 参数校验
        if (name == null || name.length() == 0) {
            throw new ValidationException("length 不能为空");
        }
        if (phone == null || !isValid(phone)) {
            throw new ValidationException("phone 不能为空");
        }
        // 获取手机号归属地编号 和 运营商编号 然后通过编号找到区域内是 SalesRep
        String areaCode = FindUtils.getAreaCode(phone);
        String operatorCode = FindUtils.getOperatorCode(phone);

        String user = "";
        try {
            int repId = salesDao.findRep(areaCode, operatorCode);
            user = userDao.save(name, phone, repId);
        } catch (SQLException e) {
            throw new DaoException();
        }
        return user;
    }

    public boolean isValid(String phone) {
        return phone.equals("12345678911");
//        return phone.matches(pattern);
    }
}
```
```java
public class SalesDao {
    public int findRep(String areaCode, String operationCode) throws SQLException {
        return 1;
    }
}
```
```java
public class UserDao {
    public String save(String v1, String v2, int v3) throws SQLException {
        return "real 1";
    }
}
```
# mock的大致原理——父子类/派生类(cglib)
```java
public class User {
}
```
```java
public class SimpleDao {
    public void insertUser(User user) {
        throw new RuntimeException("the database is not connect");
    }
}
```
```java
public class SimpleService {
    SimpleDao simpleDao;

    public SimpleService(SimpleDao simpleDao) {
        this.simpleDao = simpleDao;
    }

    public void save(User user) {
        // ...

        // save to db：数据不可用，是原创的，则需要mock技术来测试
        simpleDao.insertUser(user);

        // ...
    }
}
```
```java
public class SimpleServiceTest {
    SimpleDaoMock mock;

    public SimpleServiceTest(SimpleDaoMock simpleDao) {
        mock = simpleDao;
    }

    public static void main(String[] args) {
        // 创建一个mock
        SimpleDaoMock mock = new SimpleDaoMock();
        // 测试 SimpleService 的 save方法
        User user = new User();
        SimpleService service = new SimpleService(mock);
        // 用mock来替换api里使用的成员变量。调用api。 如果要mock的对象是局部变量，则mockito无法做，只有通过powermock来修改字节码的方式做
        service.save(user);
    }

    // 父子类技术，派生一个Mock对象
    static class SimpleDaoMock extends SimpleDao {
        @Override
        public void insertUser(User user) {
            System.out.println("mock insert user");
        }
    }
}
```
# mock存在的问题
如果要测试的API是这样的，需要mock的对象是局部变量，则mock无能为力
```java
public class SimpleService {
    public void save(User user) {
        // ...

        // save to db：数据不可用，是原创的
        simpleDao.insertUser(user);

        // ...
    }
}
```
解决方法：使用powermock，修改字节码

# powermock 实战
- 排除bug
- 程序依赖某些其他库/组件，但是本机没有
powermock 依赖于 easymock mockito

pom依赖
```xml
        <!-- https://mvnrepository.com/artifact/org.powermock/powermock-module-junit4 -->
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-module-junit4</artifactId>
            <version>2.0.9</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.powermock/powermock-api-mockito2 -->
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-api-mockito2</artifactId>
            <version>2.0.9</version>
            <scope>test</scope>
        </dependency>
```
# powermock 基本用法
约定：`service` 调用 `dao` 并且 `dao` 不可用

## mock 局部变量
```java
public class UserDao {
    public int getCount() {
        return 1;
    }

    public void insert(User user) {

    }
}
```
```java
public class UserServiceLocalVariable {

    public int queryUserCount() {
        UserDao userDao = new UserDao();
        return userDao.getCount();
    }

    public void insertUser(User user) {
        UserDao userDao = new UserDao();
        userDao.insert(user);
    }
}
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserServiceLocalVariable.class})
public class UserServiceLocalVariableTest {

    @Test
    public void testCount() {
        try {
            UserServiceLocalVariable service = new UserServiceLocalVariable();
            // 先mock一个useDao 以备用，等下要注入到要测试的api的局部变量里面
            UserDao userDaoMock = PowerMockito.mock(UserDao.class);
            // 模拟 UserDao的 (无参)构造函数
            PowerMockito.whenNew(UserDao.class).withNoArguments().thenReturn(userDaoMock);
            // 模拟 userDaoMock 的具体API 的行为
            PowerMockito.doReturn(10).when(userDaoMock).getCount();
            int count = service.queryUserCount();
            Assert.assertEquals(10, count);
        } catch (Exception e) {
            Assert.fail();
        }
    }

    @Test
    public void testSave() {
        try {
            UserServiceLocalVariable service = new UserServiceLocalVariable();
            UserDao userDaoMock = PowerMockito.mock(UserDao.class);
            PowerMockito.whenNew(UserDao.class).withNoArguments().thenReturn(userDaoMock);
            User user = new User();
            PowerMockito.doNothing().when(userDaoMock).insert(user);
            service.insertUser(user);
            // 验证是否调用了 mock 的insert方法
            Mockito.verify(userDaoMock).insert(user);
        } catch (Exception e) {
            Assert.fail();
        }
    }
}
```
## mock static方法
```java
public class UserService {

    public int queryUserCount() {
        return UserDao.getCount();
    }

    public void insertUser(User user) {
        UserDao.insert(user);
    }
}
```
```java
public class UserDao {
    public static int getCount() {
        return 1;
    }

    public static void insert(User user) {
    }
}
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserDao.class}) //https://blog.csdn.net/qaz296490709/article/details/72880370
public class UserServiceTest extends TestCase {

    public void testQueryUserCount() throws Exception {
        PowerMockito.mockStatic(UserDao.class);
        UserService userService = new UserService();
        PowerMockito.when(UserDao.getCount()).thenReturn(10);
        int count = userService.queryUserCount();
        Assert.assertEquals(count, 10);
    }

    public void testInsertUser() {
        PowerMockito.mockStatic(UserDao.class);
        User user = new User();
        PowerMockito.doNothing().when(UserDao.class);
        UserService userService = new UserService();
        userService.insertUser(user);
        PowerMockito.mockStatic(UserDao.class);
    }
}
```



## mock final 类
无法继承，无法重写方法。
```java
// 不管是 final 类 还是 final方法都可以
public final class UserDao {

    public int getCount() {
        return 1;
    }
    public void insert(User user) {
    }

}
```
```java
public class UserService {

    private UserDao userDao;

    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }

    public int queryUserCount() {
        return userDao.getCount();
    }

    public void insertUser(User user) {
        userDao.insert(user);
    }
}
```
```java
// 使用注解和使用new的方式都可以
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserDao.class})
public class UserServiceTest {

    @Mock
    UserDao userDao;
    @InjectMocks
    UserService userService;

    @Test
    public void count() {
//        UserDao userDao = PowerMockito.mock(UserDao.class);
        System.out.println(userDao.getClass());
        PowerMockito.doReturn(10).when(userDao).getCount();
//        UserService userService = new UserService(userDao);
        int count = userService.queryUserCount();
        Assert.assertEquals(count, 10);
    }

    @Test
    public void insert() {
//        UserDao userDao = PowerMockito.mock(UserDao.class);
        System.out.println(userDao.getClass());
        User user = new User();
        PowerMockito.doNothing().when(userDao).insert(user);
//        UserService userService = new UserService(userDao);
        userService.insertUser(user);
        Mockito.verify(userDao).insert(user);
    }
}
```

## verify 方法
```java
public class UserDao {
    public int getCount() {
        throw new UnsupportedOperationException();
    }

    public void insert(User user) {
        throw new UnsupportedOperationException();
    }

    public void updateUser(User user) {
        throw new UnsupportedOperationException();
    }
}
```
```java
public class UserServiceLocalVariable {

    public void saveOrUpdate(User user) {
        UserDao userDao = new UserDao();
        if (userDao.getCount() > 0) {
            userDao.updateUser(user);
        } else {
            userDao.insert(user);
        }
    }
}

```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserServiceLocalVariable.class})
public class UserServiceLocalVariableTest {

    @Mock
    UserDao userDao;

    @InjectMocks
    UserServiceLocalVariable userServiceLocalVariable;

    @Test
    public void test() throws Exception {
        //
        PowerMockito.whenNew(UserDao.class).withNoArguments().thenReturn(userDao);
        User user = new User();
        PowerMockito.doReturn(1).when(userDao).getCount();
        PowerMockito.doNothing().when(userDao).updateUser(user);
        PowerMockito.doNothing().when(userDao).insert(user);

        userServiceLocalVariable.saveOrUpdate(user);

        Mockito.verify(userDao).getCount();
        Mockito.verify(userDao, Mockito.never()).insert(user);
        Mockito.verify(userDao).updateUser(user);
    }
}
```





## mock 构造函数
```java
public class UserService {
    public void save(String username, String password) {
        UserDao userDao = new UserDao(username, password);
        System.out.println(userDao.getClass());
        userDao.insert();
    }
}
```
```java
public class UserDao {

    private String username;
    private String password;
    public UserDao() {}
    public UserDao(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public void insert() {
        throw new UnsupportedOperationException();
    }
}
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest extends TestCase {

    @InjectMocks
    UserService userService;

    @Mock
    UserDao userDao;

    @Test
    public void testInsertUser() throws Exception {
        String username = "11", password = "22";
        PowerMockito.whenNew(UserDao.class).withArguments(username, password).thenReturn(userDao);
        System.out.println(userDao.getClass());
        // 没有打桩成功
        PowerMockito.doNothing().when(userDao).insert();
        userService.save(username, password);
        Mockito.verify(userDao).insert();
    }
}
```
## `ArgumentsMatchers` 接口
对于多个参数都返回一个值的情况们可以用 `Matcher` 和 `ArgumentsMatchers` 接口
```java
public class UserDao {

    public String queryByName(String name) {
        return "ss";
    }
}
```
```java
public class UserService {
    public String find(String name) {
        return new UserDao().queryByName(name);
    }
}
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest extends TestCase {

    @Mock
    UserDao userDao;
    @InjectMocks
    UserService userService;

    @Test
    public void testFind() throws Exception {
        PowerMockito.whenNew(UserDao.class).withNoArguments().thenReturn(userDao);
        PowerMockito.when(userDao.queryByName(Matchers.argThat(new MyArgMatchers()))).thenReturn("wdh");
        Assert.assertEquals("wdh", userService.find("11"));
        Assert.assertEquals("wdh", userService.find("22"));
        Assert.assertEquals("wdh", userService.find("33"));
    }

    static class MyArgMatchers implements ArgumentMatcher<String> {
        @Override
        public boolean matches(String s) {
            switch (s) {
                case "11":
                case "22":
                case "33":
                    return true;
                default:
                    return false;
            }
        }
    }
}
```

## Answer 接口
Matcher的增强版，不同的参数对应不同的结果。
```java
public class UserDao {
    public String queryByName(String name) {
        return "ss";
    }
}
```
```java
public class UserService {
    public String find(String name) {
        return new UserDao().queryByName(name);
    }
}
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest {
    @Mock
    UserDao userDao;
    @InjectMocks
    UserService userService;

    @Test
    public void testFind() throws Exception {
        PowerMockito.whenNew(UserDao.class).withNoArguments().thenReturn(userDao);

        PowerMockito.when(userDao.queryByName(Mockito.anyString())).then((invocation) -> {
            String arg = (String) invocation.getArguments()[0];
            switch (arg) {
                case "11":
                    return "I am 11";
                case "22":
                    return "I am 22";
                case "33":
                    return "I am 33";
                default:
                    throw new RuntimeException("Not support: " + arg);
            }
        });
        Assert.assertEquals("I am 11", userService.find("11"));
        Assert.assertEquals("I am 22", userService.find("22"));
        Assert.assertEquals("I am 33", userService.find("33"));
    }
}
```


## Spy 以及 private 方法
Spy 创建的对象是执行真实的方法。
如果打桩的行为和真实测试的行为是一致的，则不会执行真实的方法。否则执行的是真实的方法
```java
public class UserService {
    public void foo(String msg) {
        log(msg);
    }

    public void log(String msg) {
        System.out.println("I am a service:" + msg);
    }

    public void exit(String msg) {
        checkExit(msg);
    }

    private void checkExit(String msg) {
        System.out.println("msg:"+ msg);
        throw new UnsupportedOperationException();
    }
}
```
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({UserService.class})
public class UserServiceTest {

    @Test
    public void testFoo() {
        UserService spy = PowerMockito.spy(new UserService());
        PowerMockito.doNothing().when(spy).foo("bb");
        spy.foo("bb"); // 没有任何输出
//        spy.foo("ss"); // 有输出
    }

    @Test
    public void testPrivate() throws Exception {
        UserService spy = PowerMockito.spy(new UserService());
        PowerMockito.when(spy, "checkExit", "aa");
        spy.exit("aa");
//        spy.exit("bb");
    }
}
```

## 总结
- powermock 不是重复造轮子
- powermock是完成其他mock不能完成的任务
- 尽量减少使用powermock


## 注解 `@PrepareForTest`
- 当使用`PowerMockito.whenNew`方法时，必须加注解`@PrepareForTest`和`@RunWith`。注解`@PrepareForTest`里写的类是需要mock的new对象代码所在的类。


- 当需要mock `final`方法的时候，必须加注解`@PrepareForTest`和`@RunWith`。注解`@PrepareForTest`里写的类是`final`方法所在的类。 


- 当需要mock `static` 方法的时候，必须加注解`@PrepareForTest`和`@RunWith`。注解`@PrepareForTest`里写的类是静态方法所在的类。


- 当需要mock `private`方法的时候, 只是需要加注解`@PrepareForTest`，注解里写的类是私有方法所在的类


- 当需要mock系统类的静态方法的时候，必须加注解`@PrepareForTest`和`@RunWith`。注解里写的类是需要调用系统方法所在的类





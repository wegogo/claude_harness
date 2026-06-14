# Java 后端测试规范

> 适用于 Java 21 + Spring Boot 3.5 项目的测试编写。
> 目标：用最小的成本写出"既能防回归、又不会成为维护负担"的测试。

---

## 一、测试分层

| 层级 | 测试类型 | 框架 | 运行速度 | 测试什么 |
|------|---------|------|---------|---------|
| Service 层 | 单元测试 | JUnit 5 + Mockito | 毫秒级 | 业务逻辑正确性 |
| Controller 层 | 切片测试 | `@WebMvcTest` | 秒级 | 请求/响应、参数校验、路由 |
| Mapper 层 | 集成测试 | `@MybatisPlusTest` + Testcontainers | 秒级 | SQL 正确性、映射 |
| 端到端 | E2E 测试 | `@SpringBootTest` | 慢（秒~十秒） | 整体流程 |

**原则：测试金字塔 —— 单元测试占 70%，切片/集成测试占 20%，E2E 占 10%。**

---

## 二、测试依赖

`pom.xml` 中需包含：

```xml
<!-- Spring Boot 测试启动器（含 JUnit 5, AssertJ, Mockito, Spring Test） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- MyBatis-Plus 测试支持（提供 @MybatisPlusTest 注解） -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-test-autoconfigure</artifactId>
    <scope>test</scope>
</dependency>

<!-- Testcontainers（集成测试，按需引入） -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mysql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mongodb</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- JaCoCo 覆盖率（可选，推荐） -->
<dependency>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
</dependency>
```

---

## 三、命名约定

### 3.1 测试类

```
被测类名 + Test
```

| 被测类 | 测试类 |
|--------|--------|
| `UserService` | `UserServiceTest` |
| `UserController` | `UserControllerTest` |
| `OrderMapper` | `OrderMapperTest` |

### 3.2 测试方法

使用 **`should_预期行为_when_前置条件`** 格式（或中文描述）：

```java
// 英文格式
@Test
void should_return_user_when_id_exists() { }

@Test
void should_throw_exception_when_username_is_blank() { }

// 中文格式（推荐中文项目使用，可读性更好）
@Test
void 查询用户_当ID存在时_返回用户信息() { }

@Test
void 创建用户_当用户名已存在时_抛出异常() { }
```

### 3.3 测试包结构

测试目录结构与源码目录一一对应：

```
src/test/java/com/example/project/
├── service/
│   ├── UserServiceTest.java
│   └── OrderServiceTest.java
├── controller/
│   └── UserControllerTest.java
├── mapper/
│   └── UserMapperTest.java
└── resources/
    └── application-test.yml
```

---

## 四、测试编写规范

### 4.1 AAA 模式

每个测试方法遵循 **Arrange（准备）→ Act（执行）→ Assert（断言）** 结构，用空行分隔：

```java
@Test
void 查询用户_当ID存在时_返回用户信息() {
    // Arrange
    Long userId = 1L;
    User mockUser = new User(userId, "zhangsan", "zhangsan@test.com");
    when(userRepository.findById(userId)).thenReturn(Optional.of(mockUser));

    // Act
    UserVO result = userService.getById(userId);

    // Assert
    assertThat(result).isNotNull();
    assertThat(result.id()).isEqualTo(userId);
    assertThat(result.username()).isEqualTo("zhangsan");
    verify(userRepository).findById(userId);
}
```

### 4.2 Given-When-Then 注释

复杂测试可用注释块替代空行分隔，进一步明确结构：

```java
@Test
void 创建用户_当邮箱合法时_保存并返回() {
    // given
    UserCreateDTO dto = new UserCreateDTO("lisi", "lisi@test.com", "password123");
    when(userRepository.existsByUsername("lisi")).thenReturn(false);
    when(userRepository.save(any(User.class)))
        .thenAnswer(inv -> {
            User u = inv.getArgument(0);
            u.setId(1L);
            return u;
        });

    // when
    UserVO result = userService.create(dto);

    // then
    assertThat(result.id()).isEqualTo(1L);
    assertThat(result.username()).isEqualTo("lisi");
    verify(userRepository).save(any(User.class));
}
```

### 4.3 一个测试只验证一个行为

- 一个 `@Test` 方法只测一个场景
- 多个场景拆分为多个方法，或使用 `@ParameterizedTest`
- 断言数量不限，但都应围绕同一个行为

---

## 五、Service 层单元测试

这是**数量最多**的测试，Mock 掉所有外部依赖，只测业务逻辑。

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserMapper userMapper;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserServiceImpl userService;

    @Test
    void 查询用户_当ID不存在时_抛出USER_NOT_FOUND异常() {
        // given
        Long userId = 999L;
        when(userMapper.selectById(userId)).thenReturn(null);

        // when & then
        assertThatThrownBy(() -> userService.getById(userId))
            .isInstanceOf(BusinessException.class)
            .extracting("errorCode")
            .isEqualTo(ErrorCode.USER_NOT_FOUND);
    }

    @Test
    void 创建用户_当用户名已存在时_抛出USERNAME_EXISTS异常() {
        // given
        UserCreateDTO dto = new UserCreateDTO("zhangsan", "zhangsan@test.com", "password");
        when(userMapper.selectCount(any())).thenReturn(1L);

        // when & then
        assertThatThrownBy(() -> userService.create(dto))
            .isInstanceOf(BusinessException.class)
            .extracting("errorCode")
            .isEqualTo(ErrorCode.USERNAME_EXISTS);

        verify(userMapper, never()).insert(any());
    }

    @Test
    void 创建用户_成功后_发送欢迎邮件() {
        // given
        UserCreateDTO dto = new UserCreateDTO("wangwu", "wangwu@test.com", "password");
        when(userMapper.selectCount(any())).thenReturn(0L);
        when(userMapper.insert(any(User.class))).thenReturn(1);

        // when
        userService.create(dto);

        // then
        verify(emailService).sendWelcomeEmail("wangwu@test.com");
    }
}
```

**要点：**
- 使用 `@ExtendWith(MockitoExtension.class)`，不需要 Spring 上下文，启动快
- 所有外部依赖（Mapper、其他 Service、消息队列）都用 `@Mock`
- 用 `@InjectMocks` 自动注入 Mock 到被测对象
- 使用 AssertJ（`assertThat`）断言，比 JUnit 原生断言更流式、可读
- 验证 Mock 交互用 `verify()`，但不要过度验证（只验证关键的副作用调用）

---

## 六、Controller 层切片测试

使用 `@WebMvcTest` 加载 Web 层，Mock 掉 Service，测试 HTTP 交互。

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void 查询用户_GET_users_id_当用户存在时_返回200和用户数据() throws Exception {
        // given
        Long userId = 1L;
        UserVO userVO = new UserVO(userId, "zhangsan", "zhangsan@test.com");
        when(userService.getById(userId)).thenReturn(userVO);

        // when & then
        mockMvc.perform(get("/api/v1/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(0))
            .andExpect(jsonPath("$.data.username").value("zhangsan"))
            .andExpect(jsonPath("$.data.email").value("zhangsan@test.com"));
    }

    @Test
    void 创建用户_POST_users_当用户名非法时_返回400() throws Exception {
        // given — 用户名为空，触发 @Valid 校验
        String json = """
            {"username": "", "email": "test@test.com", "password": "12345678"}
            """;

        // when & then
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value(400));
    }

    @Test
    void 查询用户_当Service抛出业务异常时_返回对应错误码() throws Exception {
        // given
        Long userId = 999L;
        when(userService.getById(userId))
            .thenThrow(new BusinessException(ErrorCode.USER_NOT_FOUND));

        // when & then
        mockMvc.perform(get("/api/v1/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(10001));
    }
}
```

**要点：**
- `@WebMvcTest` 只加载 Controller 层，不加载 Service/Mapper，速度快
- `@MockBean` 替换 Service 层
- 重点测试：路由是否正确、参数校验是否生效、异常是否被正确处理、JSON 结构是否正确
- 不需要测业务逻辑——那是 Service 测试的事

---

## 七、Mapper 层集成测试

使用 Testcontainers 启动真实数据库容器，测试 SQL 和 MyBatis-Plus 映射。

### 7.1 MySQL 集成测试

```java
@MybatisPlusTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserMapperTest {

    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }

    @Autowired
    private UserMapper userMapper;

    @Test
    void selectCount_当用户名存在时_返回大于0() {
        // given
        User user = new User();
        user.setUsername("zhangsan");
        user.setEmail("zhangsan@test.com");
        userMapper.insert(user);

        // when
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
            .eq(User::getUsername, "zhangsan");
        Long count = userMapper.selectCount(wrapper);

        // then
        assertThat(count).isGreaterThan(0);
    }

    @Test
    void selectCount_当用户名不存在时_返回0() {
        // when
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
            .eq(User::getUsername, "nobody");
        Long count = userMapper.selectCount(wrapper);

        // then
        assertThat(count).isEqualTo(0);
    }

    @Test
    void selectList_按邮箱模糊查询_返回匹配结果() {
        // given
        User u1 = new User(); u1.setUsername("user1"); u1.setEmail("user1@test.com");
        User u2 = new User(); u2.setUsername("user2"); u2.setEmail("user2@other.com");
        userMapper.insert(u1);
        userMapper.insert(u2);

        // when
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
            .like(User::getEmail, "@test.com");
        List<User> result = userMapper.selectList(wrapper);

        // then
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getUsername()).isEqualTo("user1");
    }
}
```

### 7.2 MongoDB 集成测试

```java
@DataMongoTest
@Testcontainers
class OperationLogRepositoryTest {

    @Container
    static MongoDBContainer mongo = new MongoDBContainer<>("mongo:7.0");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongo::getReplicaSetUrl);
    }

    @Autowired
    private OperationLogRepository logRepository;

    @Test
    void 按时间段查询操作日志_返回区间内记录() {
        // given
        LocalDateTime now = LocalDateTime.now();
        logRepository.save(new OperationLog("user1", "LOGIN", now.minusHours(1)));
        logRepository.save(new OperationLog("user2", "LOGOUT", now.minusDays(2)));

        // when
        List<OperationLog> result = logRepository
            .findByCreatedAtAfter(now.minusHours(2));

        // then
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getAction()).isEqualTo("LOGIN");
    }
}
```

### 7.3 Redis 集成测试

使用 Testcontainers 或嵌入式 Redis：

```java
@Testcontainers
@SpringBootTest
class UserCacheServiceTest {

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:6.0")
        .withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
    }

    @Autowired
    private UserCacheService userCacheService;

    @Test
    void 缓存用户_写入后_可读取() {
        // given
        Long userId = 1L;
        UserVO user = new UserVO(userId, "zhangsan", "zhangsan@test.com");

        // when
        userCacheService.cache(userId, user);
        Optional<UserVO> result = userCacheService.get(userId);

        // then
        assertThat(result).isPresent();
        assertThat(result.get().username()).isEqualTo("zhangsan");
    }
}
```

---

## 八、参数化测试

当多个输入走同一逻辑时，用 `@ParameterizedTest` 替代重复的 `@Test`：

```java
@ParameterizedTest
@CsvSource({
    "zhangsan, true",
    "'', false",
    "ab, false",
    "this_username_is_way_too_long_for_limit, false"
})
void 用户名校验_参数化测试(String username, boolean expected) {
    boolean valid = username != null
        && !username.isBlank()
        && username.length() >= 3
        && username.length() <= 20;
    assertThat(valid).isEqualTo(expected);
}
```

```java
@ParameterizedTest
@EnumSource(value = OrderStatus.class, names = {"PENDING", "PROCESSING"})
void 订单状态为待处理或处理中时_允许取消(OrderStatus status) {
    assertThat(orderService.canCancel(status)).isTrue();
}
```

---

## 九、测试边界用例

每个方法至少覆盖以下场景：

| 场景类别 | 说明 | 示例 |
|---------|------|------|
| 正常路径 | 输入合法，预期成功 | 查询存在的用户 |
| 边界值 | 最小/最大/临界值 | 分页 `page=0`、`size=100` |
| 空值/null | 空字符串、空集合、null | 用户名为空 |
| 异常路径 | 输入非法或条件不满足 | 用户名已存在 |
| 并发场景 | （视需要）乐观锁、重复提交 | 乐观锁版本冲突 |

---

## 十、覆盖率要求

### 10.1 JaCoCo 配置

`pom.xml` 插件配置：

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 10.2 覆盖率标准

| 指标 | 最低要求 | 建议 |
|------|---------|------|
| 行覆盖率（Line） | ≥ 80% | ≥ 85% |
| 分支覆盖率（Branch） | ≥ 70% | ≥ 80% |
| Service 层 | ≥ 85% | ≥ 90% |
| Controller 层 | ≥ 80% | ≥ 85% |
| Mapper 层 | ≥ 70% | ≥ 80% |
| Model / DTO / Util | ≥ 90% | ≥ 95% |

**覆盖率不是目的，是底线。** 追求 100% 覆盖率而写无意义的断言比低覆盖率更糟。

### 10.3 排除规则

以下代码可排除覆盖率统计：

```xml
<configuration>
    <excludes>
        <!-- 配置类、启动类、自动生成的代码 -->
        <exclude>**/config/**</exclude>
        <exclude>**/Application.*</exclude>
        <exclude>**/model/entity/**</exclude>
        <exclude>**/*Mapper.xml</exclude>  <!-- MyBatis XML 映射文件 -->
        <!-- 注意：Mapper 接口本身仍需覆盖率统计，不排除 -->
    </excludes>
</configuration>
```

---

## 十一、测试运行命令

```bash
# 运行全部测试
./mvnw test

# 运行指定测试类
./mvnw test -Dtest=UserServiceTest

# 运行指定测试方法
./mvnw test -Dtest="UserServiceTest#查询用户_当ID存在时_返回用户信息"

# 生成覆盖率报告（target/site/jacoco/index.html）
./mvnw clean verify

# 只跑单元测试（跳过慢的集成测试）
./mvnw test -DexcludedGroups=integration

# 只跑集成测试
./mvnw test -Dgroups=integration
```

需要配合 JUnit 5 Tag 标注：

```java
@Tag("integration")
@DataJpaTest
class UserRepositoryTest { }

@Tag("unit")
@ExtendWith(MockitoExtension.class)
class UserServiceTest { }
```

---

## 十二、测试红线（禁止事项）

1. **禁止测试间互相依赖**：每个测试独立，不依赖其他测试的执行顺序或状态
2. **禁止依赖当前时间**：使用可注入的 `Clock` 或 `LocalDateTime` 参数，不要 `LocalDateTime.now()`
3. **禁止依赖外部网络**：外部 API 调用必须 Mock，或使用 WireMock
4. **禁止共享可变状态**：测试类字段如需初始化，用 `@BeforeEach` 而非依赖构造顺序
5. **禁止无断言的测试**：每个测试必须有明确的断言，"不抛异常就算通过"不是测试
6. **禁止 sleep 等待**：用 Awaitility 替代 `Thread.sleep()`
7. **禁止测试生产代码**：测试代码不 import 生产包以外的内部实现细节做白盒断言

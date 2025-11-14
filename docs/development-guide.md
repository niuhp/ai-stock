# AI Stock 开发规范文档

## 1. 代码规范

### 1.1 命名规范

#### 1.1.1 包命名
```
规则：全小写，多个单词用点分隔
示例：
  ✓ com.aistock.user.service
  ✓ com.aistock.data.mapper
  ✗ com.aistock.User.Service
  ✗ com.aistock.dataMapper
```

#### 1.1.2 类命名
```
规则：大驼峰（PascalCase），见名知义

Controller层：
  ✓ UserController
  ✓ StockDataController
  
Service层：
  ✓ UserService / UserServiceImpl
  ✓ StockAnalysisService
  
Mapper层：
  ✓ UserMapper
  ✓ StockInfoMapper
  
实体类：
  ✓ User
  ✓ StockInfo
  ✓ RecommendationRecord
  
DTO类：
  ✓ UserRegisterDTO
  ✓ StockQueryDTO
  
VO类：
  ✓ UserInfoVO
  ✓ StockDetailVO
```

#### 1.1.3 方法命名
```
规则：小驼峰（camelCase），动词开头

查询方法：
  ✓ getUserById(Long id)
  ✓ listStocksByIndustry(String industry)
  ✓ findRecommendations()
  
保存方法：
  ✓ saveUser(User user)
  ✓ insertStock(StockInfo stock)
  
更新方法：
  ✓ updateUser(User user)
  ✓ modifyStockInfo(StockInfo stock)
  
删除方法：
  ✓ deleteUser(Long id)
  ✓ removeStock(String code)
  
布尔方法：
  ✓ isValidUser(Long userId)
  ✓ hasPermission(String permission)
  ✓ existsStock(String code)
```

#### 1.1.4 变量命名
```
规则：小驼峰，见名知义

✓ String userName
✓ List<StockInfo> stockList
✓ boolean isEnabled
✓ int totalCount
✓ BigDecimal currentPrice

✗ String s
✗ List<StockInfo> list1
✗ boolean flag
✗ int temp
```

#### 1.1.5 常量命名
```
规则：全大写，下划线分隔

✓ public static final String DEFAULT_PASSWORD = "123456";
✓ public static final int MAX_RETRY_COUNT = 3;
✓ public static final long SESSION_TIMEOUT = 7200000L;

✗ public static final String defaultPassword = "123456";
✗ public static final int MAX_retry_count = 3;
```

### 1.2 注释规范

#### 1.2.1 类注释
```java
/**
 * 用户服务实现类
 * 
 * 提供用户注册、登录、信息管理等功能
 * 
 * @author zhangsan
 * @since 2024-01-01
 * @version 1.0
 */
public class UserServiceImpl implements UserService {
    // ...
}
```

#### 1.2.2 方法注释
```java
/**
 * 根据用户ID获取用户信息
 * 
 * @param userId 用户ID
 * @return 用户信息，不存在返回null
 * @throws BusinessException 业务异常
 */
public User getUserById(Long userId) {
    // ...
}
```

#### 1.2.3 代码注释
```java
// 单行注释：解释下一行代码的作用
int retryCount = 3;

/*
 * 多行注释：
 * 解释复杂逻辑
 * 第二行说明
 */
if (retryCount > MAX_RETRY_COUNT) {
    throw new BusinessException("重试次数超限");
}

// TODO: 待实现功能
// FIXME: 需要修复的bug
// IMPORTANT: 重要提示
```

### 1.3 代码格式

#### 1.3.1 缩进与换行
```java
// 使用4个空格缩进，不使用Tab
public class Example {
    private String name;
    
    public void method() {
        if (condition) {
            // 代码
        }
    }
}

// 大括号不单独占一行（K&R风格）
✓ if (condition) {
    // code
}

✗ if (condition) 
{
    // code
}

// 链式调用换行
User user = User.builder()
    .name("张三")
    .age(25)
    .email("zhangsan@example.com")
    .build();
```

#### 1.3.2 空行规范
```java
public class Example {
    // 字段之间无需空行
    private String name;
    private Integer age;
    
    // 方法之间空一行
    public void method1() {
        // ...
    }
    
    public void method2() {
        // ...
    }
}
```

#### 1.3.3 import规范
```java
// 顺序：java -> javax -> org -> com -> 自己的包
// 每个分组之间空一行
// 禁止使用 import xxx.*;

import java.util.List;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.aistock.common.utils.DateUtils;
import com.aistock.user.entity.User;
```

### 1.4 编码规范

#### 1.4.1 常量使用
```java
// ✓ 使用常量
private static final int MAX_USER_COUNT = 1000;
if (userCount > MAX_USER_COUNT) {
    // ...
}

// ✗ 魔法值
if (userCount > 1000) {
    // ...
}
```

#### 1.4.2 空值处理
```java
// ✓ 使用Optional
public Optional<User> findUserById(Long id) {
    return Optional.ofNullable(userMapper.selectById(id));
}

// ✓ 返回空集合而非null
public List<User> listUsers() {
    List<User> users = userMapper.selectList(null);
    return users != null ? users : Collections.emptyList();
}

// ✓ 使用Objects.isNull()
if (Objects.isNull(user)) {
    throw new BusinessException("用户不存在");
}

// ✓ 集合判空
if (CollectionUtils.isEmpty(userList)) {
    return;
}
```

#### 1.4.3 异常处理
```java
// ✓ 捕获具体异常
try {
    // ...
} catch (IOException e) {
    log.error("文件读取失败", e);
    throw new BusinessException("文件读取失败");
} catch (SQLException e) {
    log.error("数据库操作失败", e);
    throw new BusinessException("数据库操作失败");
}

// ✗ 捕获Exception
try {
    // ...
} catch (Exception e) {
    // bad practice
}

// ✗ 空catch块
try {
    // ...
} catch (Exception e) {
    // do nothing - bad!
}
```

#### 1.4.4 资源关闭
```java
// ✓ 使用try-with-resources
try (InputStream is = new FileInputStream("file.txt");
     BufferedReader br = new BufferedReader(new InputStreamReader(is))) {
    String line;
    while ((line = br.readLine()) != null) {
        // process line
    }
}

// ✗ 手动关闭（容易忘记）
InputStream is = null;
try {
    is = new FileInputStream("file.txt");
    // ...
} finally {
    if (is != null) {
        is.close();
    }
}
```

## 2. 分层架构规范

### 2.1 项目结构

```
ai-stock-user/
├── src/main/java/com/aistock/user/
│   ├── controller/          # 控制层
│   │   ├── UserController.java
│   │   └── AuthController.java
│   ├── service/             # 服务层
│   │   ├── UserService.java
│   │   └── impl/
│   │       └── UserServiceImpl.java
│   ├── mapper/              # 数据访问层
│   │   └── UserMapper.java
│   ├── entity/              # 实体类
│   │   ├── User.java
│   │   └── UserPreference.java
│   ├── dto/                 # 数据传输对象
│   │   ├── request/
│   │   │   ├── UserRegisterDTO.java
│   │   │   └── UserLoginDTO.java
│   │   └── response/
│   │       └── UserInfoDTO.java
│   ├── vo/                  # 视图对象
│   │   └── UserVO.java
│   ├── enums/               # 枚举类
│   │   └── UserStatusEnum.java
│   ├── exception/           # 异常类
│   │   └── UserException.java
│   ├── config/              # 配置类
│   │   ├── SecurityConfig.java
│   │   └── RedisConfig.java
│   └── util/                # 工具类
│       └── JwtUtils.java
├── src/main/resources/
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-prod.yml
│   └── mapper/              # MyBatis XML
│       └── UserMapper.xml
└── src/test/java/           # 测试类
    └── com/aistock/user/
        └── service/
            └── UserServiceTest.java
```

### 2.2 分层职责

#### 2.2.1 Controller层
```java
/**
 * 职责：
 * 1. 接收请求参数
 * 2. 参数校验
 * 3. 调用Service
 * 4. 返回响应
 * 
 * 注意：
 * - 不处理业务逻辑
 * - 不直接访问数据库
 * - 统一返回Result<T>
 */
@RestController
@RequestMapping("/api/v1/users")
@Slf4j
public class UserController {
    
    @Autowired
    private UserService userService;
    
    /**
     * 用户注册
     */
    @PostMapping("/register")
    public Result<Long> register(@Valid @RequestBody UserRegisterDTO dto) {
        log.info("用户注册: {}", dto.getUsername());
        Long userId = userService.register(dto);
        return Result.success(userId);
    }
}
```

#### 2.2.2 Service层
```java
/**
 * 职责：
 * 1. 业务逻辑处理
 * 2. 事务控制
 * 3. 调用Mapper
 * 4. 调用其他Service
 * 
 * 注意：
 * - 不处理HTTP请求
 * - 不返回VO，返回DO或DTO
 */
@Service
@Slf4j
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long register(UserRegisterDTO dto) {
        // 1. 校验用户名是否存在
        if (userMapper.existsByUsername(dto.getUsername())) {
            throw new BusinessException("用户名已存在");
        }
        
        // 2. 创建用户
        User user = new User();
        BeanUtils.copyProperties(dto, user);
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setCreateTime(LocalDateTime.now());
        userMapper.insert(user);
        
        // 3. 初始化用户偏好
        UserPreference preference = new UserPreference();
        preference.setUserId(user.getId());
        preferenceMapper.insert(preference);
        
        return user.getId();
    }
}
```

#### 2.2.3 Mapper层
```java
/**
 * 职责：
 * 1. 数据库CRUD操作
 * 2. 复杂SQL查询
 * 
 * 注意：
 * - 不处理业务逻辑
 * - 使用MyBatis Plus BaseMapper
 */
@Mapper
public interface UserMapper extends BaseMapper<User> {
    
    /**
     * 检查用户名是否存在
     */
    boolean existsByUsername(@Param("username") String username);
    
    /**
     * 根据手机号查询用户
     */
    User selectByPhone(@Param("phone") String phone);
    
    /**
     * 查询VIP用户列表
     */
    List<User> selectVipUsers(@Param("vipLevel") Integer vipLevel);
}
```

### 2.3 对象转换

```java
// Controller层：DTO -> Service
UserRegisterDTO dto = new UserRegisterDTO(); // 前端传入
Long userId = userService.register(dto);

// Service层：DTO -> Entity
User user = new User();
BeanUtils.copyProperties(dto, user);
userMapper.insert(user);

// Service层：Entity -> VO (Controller层返回)
User user = userMapper.selectById(id);
UserVO vo = new UserVO();
BeanUtils.copyProperties(user, vo);
return vo;

// 批量转换（使用工具类）
List<User> userList = userMapper.selectList(null);
List<UserVO> voList = BeanCopyUtils.copyList(userList, UserVO.class);
```

## 3. 数据库规范

### 3.1 表设计规范

#### 3.1.1 命名规范
```sql
-- 表名：小写，下划线分隔，tb_前缀
✓ tb_user
✓ tb_stock_info
✓ tb_user_preference

-- 字段名：小写，下划线分隔
✓ user_id
✓ create_time
✓ stock_code

-- 索引名：idx_表名_字段名
✓ idx_user_username
✓ idx_stock_info_code

-- 唯一索引：uk_表名_字段名
✓ uk_user_phone
✓ uk_stock_code
```

#### 3.1.2 字段规范
```sql
-- 主键：id BIGINT，自增
id BIGINT PRIMARY KEY AUTO_INCREMENT

-- 时间字段：DATETIME，not null，default
create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

-- 逻辑删除：deleted TINYINT
deleted TINYINT NOT NULL DEFAULT 0 COMMENT '0-未删除，1-已删除'

-- 金额字段：DECIMAL(15,2)
amount DECIMAL(15,2) NOT NULL DEFAULT 0.00

-- 状态字段：TINYINT
status TINYINT NOT NULL DEFAULT 1 COMMENT '0-禁用，1-正常'

-- 所有字段必须有COMMENT
username VARCHAR(50) NOT NULL COMMENT '用户名'
```

#### 3.1.3 索引规范
```sql
-- 单列索引
CREATE INDEX idx_user_username ON tb_user(username);

-- 联合索引（最左前缀原则）
CREATE INDEX idx_user_phone_status ON tb_user(phone, status);

-- 唯一索引
CREATE UNIQUE INDEX uk_user_phone ON tb_user(phone);

-- 不建议对文本字段建索引
✗ CREATE INDEX idx_content ON tb_article(content); -- content是TEXT类型
```

### 3.2 SQL编写规范

#### 3.2.1 查询规范
```sql
-- ✓ 指定具体字段
SELECT id, username, phone FROM tb_user WHERE id = 1;

-- ✗ 使用SELECT *
SELECT * FROM tb_user WHERE id = 1;

-- ✓ 分页查询
SELECT id, username FROM tb_user LIMIT 0, 20;

-- ✓ 使用EXPLAIN分析
EXPLAIN SELECT id, username FROM tb_user WHERE username = 'zhangsan';

-- ✓ 避免全表扫描
SELECT id FROM tb_user WHERE create_time > '2024-01-01'; -- create_time有索引

-- ✗ 索引失效
SELECT id FROM tb_user WHERE YEAR(create_time) = 2024; -- 函数导致索引失效
```

#### 3.2.2 更新规范
```sql
-- ✓ 带WHERE条件
UPDATE tb_user SET status = 0 WHERE id = 1;

-- ✗ 不带WHERE（危险！）
UPDATE tb_user SET status = 0;

-- ✓ 批量更新
UPDATE tb_user SET status = 0 WHERE id IN (1, 2, 3);

-- ✓ 乐观锁
UPDATE tb_user SET balance = balance - 100, version = version + 1 
WHERE id = 1 AND version = 10;
```

#### 3.2.3 删除规范
```sql
-- ✓ 逻辑删除
UPDATE tb_user SET deleted = 1 WHERE id = 1;

-- ✗ 物理删除（慎用！）
DELETE FROM tb_user WHERE id = 1;

-- ✓ 带WHERE条件
DELETE FROM tb_user WHERE id = 1 AND status = 0;
```

## 4. API设计规范

### 4.1 RESTful规范

```
资源URL：
  GET    /api/v1/users          # 获取用户列表
  GET    /api/v1/users/{id}     # 获取单个用户
  POST   /api/v1/users          # 创建用户
  PUT    /api/v1/users/{id}     # 更新用户（全量）
  PATCH  /api/v1/users/{id}     # 更新用户（部分）
  DELETE /api/v1/users/{id}     # 删除用户

子资源URL：
  GET    /api/v1/users/{id}/orders        # 获取用户的订单
  POST   /api/v1/portfolios/{id}/positions # 添加持仓

复杂操作：
  POST   /api/v1/users/{id}/activate      # 激活用户
  POST   /api/v1/stocks/{code}/analyze    # 分析股票
```

### 4.2 统一响应格式

```java
/**
 * 统一响应对象
 */
@Data
public class Result<T> {
    private Integer code;     // 状态码
    private String message;   // 消息
    private T data;           // 数据
    private Long timestamp;   // 时间戳
    
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        result.setTimestamp(System.currentTimeMillis());
        return result;
    }
    
    public static <T> Result<T> error(Integer code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        result.setTimestamp(System.currentTimeMillis());
        return result;
    }
}
```

### 4.3 异常处理

```java
/**
 * 全局异常处理器
 */
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    /**
     * 业务异常
     */
    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        log.error("业务异常: {}", e.getMessage());
        return Result.error(e.getCode(), e.getMessage());
    }
    
    /**
     * 参数校验异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValidException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldError().getDefaultMessage();
        log.error("参数校验失败: {}", message);
        return Result.error(400, message);
    }
    
    /**
     * 系统异常
     */
    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.error(500, "系统异常，请稍后重试");
    }
}
```

## 5. 日志规范

### 5.1 日志级别

```java
// ERROR：错误，需要立即处理
log.error("用户登录失败，userId: {}, 原因: {}", userId, e.getMessage(), e);

// WARN：警告，需要关注
log.warn("库存不足，商品ID: {}, 当前库存: {}", productId, stock);

// INFO：重要业务流程
log.info("用户注册成功，userId: {}, username: {}", userId, username);

// DEBUG：调试信息
log.debug("查询参数: {}", queryDTO);
```

### 5.2 日志内容

```java
// ✓ 包含关键信息
log.info("用户登录成功，userId: {}, username: {}, ip: {}", userId, username, ip);

// ✗ 信息不足
log.info("登录成功");

// ✓ 异常日志包含堆栈
log.error("数据库操作失败", e);

// ✗ 只记录消息
log.error("数据库操作失败: " + e.getMessage());

// ✓ 使用占位符（性能更好）
log.info("userId: {}, username: {}", userId, username);

// ✗ 使用字符串拼接
log.info("userId: " + userId + ", username: " + username);
```

## 6. 测试规范

### 6.1 单元测试

```java
@SpringBootTest
class UserServiceTest {
    
    @Autowired
    private UserService userService;
    
    @Test
    void testRegister_Success() {
        // Given
        UserRegisterDTO dto = new UserRegisterDTO();
        dto.setUsername("test001");
        dto.setPassword("123456");
        dto.setPhone("13800138000");
        
        // When
        Long userId = userService.register(dto);
        
        // Then
        assertNotNull(userId);
        assertTrue(userId > 0);
    }
    
    @Test
    void testRegister_DuplicateUsername() {
        // Given
        UserRegisterDTO dto = new UserRegisterDTO();
        dto.setUsername("existing_user");
        
        // When & Then
        assertThrows(BusinessException.class, () -> {
            userService.register(dto);
        });
    }
}
```

### 6.2 接口测试

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void testRegister() throws Exception {
        String requestBody = """
            {
                "username": "test001",
                "password": "123456",
                "phone": "13800138000"
            }
            """;
        
        mockMvc.perform(post("/api/v1/users/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data").isNumber());
    }
}
```

## 7. Git规范

### 7.1 分支规范

```
主分支：
  - main/master: 生产环境分支
  - develop: 开发环境分支

功能分支：
  - feature/xxx: 新功能分支
  - bugfix/xxx: bug修复分支
  - hotfix/xxx: 紧急修复分支
  - release/xxx: 发布分支

示例：
  feature/user-register
  bugfix/fix-login-error
  hotfix/fix-payment-bug
  release/v1.0.0
```

### 7.2 提交规范

```
格式：<type>(<scope>): <subject>

type类型：
  - feat: 新功能
  - fix: 修复bug
  - docs: 文档更新
  - style: 代码格式调整
  - refactor: 重构
  - test: 测试相关
  - chore: 构建/工具相关

示例：
  feat(user): 添加用户注册功能
  fix(login): 修复登录失败的bug
  docs(readme): 更新README文档
  refactor(service): 重构用户服务代码
  test(user): 添加用户服务单元测试
```

---

**文档版本**: v1.0  
**最后更新**: 2025-11-14


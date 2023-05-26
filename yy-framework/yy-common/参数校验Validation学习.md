# [参数validation学习](https://dashboard.urdp.iocoder.cn/Spring-Boot/Validation/)
## Bean Validation 定义的约束注解

#### 空和非空检查
   - `@NotBlank` ：只能用于字符串不为 null ，并且字符串 #trim() 以后 length 要大于 0 。
- `@NotEmpty`：集合对象的元素不为 0 ，即集合不为空，也可以用于字符串不为 null 。
- `@NotNull`：不能为 null 。
- `@Null`：必须为 null 。
#### 数值检查
- `@DecimalMax(value)` ：被注释的元素必须是一个数字，其值必须小于等于指定的最大值。
- `@DecimalMin(value)` ：被注释的元素必须是一个数字，其值必须大于等于指定的最小值。
- `@Digits(integer, fraction)` ：被注释的元素必须是一个数字，其值必须在可接受的范围内。
- `@Positive` ：判断正数。
- `@PositiveOrZero` ：判断正数或 0 。
- `@Max(value)` ：该字段的值只能小于或等于该值。
- `@Min(value)` ：该字段的值只能大于或等于该值。
- `@Negative` ：判断负数。
- `@NegativeOrZero` ：判断负数或 0 。
- `Boolean` 值检查
- `@AssertFalse` ：被注释的元素必须为 true 。
- `@AssertTrue` ：被注释的元素必须为 false 。
#### 长度检查
- `@Size(max, min)` ：检查该字段的 size 是否在 min 和 max 之间，可以是字符串、数组、集合、Map 等。
#### 日期检查
- `@Future` ：被注释的元素必须是一个将来的日期。
- `@FutureOrPresent`：判断日期是否是将来或现在日期。
- `@Past` ：检查该字段的日期是在过去。
- `@PastOrPresent` ：判断日期是否是过去或现在日期。
#### 其它检查
- `@Email` ：被注释的元素必须是电子邮箱地址。
- `@Pattern(value)` ：被注释的元素必须符合指定的正则表达式。

## @Valid 和 @Validated
- `@Valid` 注解，是 __Bean Validation__ 所定义，可以添加在普通方法、构造方法、方法参数、方法返回、成员变量上，表示它们需要进行约束校验。

- `@Validated` 注解，是 __Spring Validation__ 锁定义，可以添加在类、方法参数、普通方法上，表示它们需要进行约束校验。同时，`@Validated` 有 value 属性，支持**分组校验**。属性如下：   
> 分组：分组是一个抽象的概念，它可以理解为一组需要被共同校验的属性集合。
```java
// Validated.java
Class<?>[] value() default {};
```
#### @Valid 和 @Validated注解区别
总的来说，绝大多数场景下，我们使用 @Validated 注解即可。  
而在有嵌套校验的场景，我们使用 @Valid 注解添加到成员属性上。

## Demo Code
- UserAddDTO
```java
public class UserAddDTO {
    /**
     * 账号
     */
    @NotEmpty(message = "登录账号不能为空")
    @Length(min = 5, max = 16, message = "账号长度为 5-16 位")
    @Pattern(regexp = "^[A-Za-z0-9]+$", message = "账号格式为数字以及字母")
    private String username;
    /**
     * 密码
     */
    @NotEmpty(message = "密码不能为空")
    @Length(min = 4, max = 16, message = "密码长度为 4-16 位")
    private String password;
    
    // ... 省略 setting/getting 方法
}
```
- UserController
```java
@RestController
@RequestMapping("/users")
@Validated
public class UserController {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @GetMapping("/get")
    public void get(@RequestParam("id") @Min(value = 1L, message = "编号必须大于 0") Integer id) {
        logger.info("[get][id: {}]", id);
    }
    // ⬆️对于 #get(id) 方法，需要校验的参数 id ，是平铺开的，所以无需添加 @Valid 注解。

    @PostMapping("/add")
    public void add(@Valid UserAddDTO addDTO) {
        logger.info("[add][addDTO: {}]", addDTO);
    }
    // 对于 #add(addDTO) 方法，需要校验的参数 addDTO ，实际相当于嵌套校验，要校验的参数的都在 addDTO 里面，所以需要添加 @Valid 注解。

}
```
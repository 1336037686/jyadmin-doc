# Api文档的设计与实现

<br/>

| 版本   | 时间       | 修改点 | 修改人  |
| :----- | :--------- | :----- | :------ |
| v1.0.0 | 2022-05-25 | 编写   | LGX_TvT |
| -      | -          | -      | -       |

<br/>

## 1、概述



API文档是一种非常重要的工具，它可以为其他开发人员提供接口的详细说明和使用方法，促进团队成员之间的协作和沟通，简化测试过程，改善代码文档的可读性，提高开发效率。在Spring Boot中，使用API文档可以更好地理解和使用应用程序的RESTful API，从而提高开发效率、促进协作和简化测试等方面都有很大的帮助。



有很多种常用的API文档工具，以下是其中几个比较常见的：

1. **Swagger**：可以通过注释和配置文件生成 API 文档，并提供了一个交互式 UI 来帮助开发人员测试和调试 API。
2. **Postman**：提供了一个 API 文档生成器，可以将 Postman 集合转换为静态 HTML 页面。
3. **Spring REST Docs**：Spring REST Docs 是一个基于 Spring Boot 的 API 文档工具，它可以根据测试用例自动生成 API 文档，并支持多种格式，包括 AsciiDoc、Markdown 等。
5. **Knife4j**：是基于 Swagger 和 Spring Boot 技术栈的Api文档，并提供了许多额外的功能，如文档聚合、在线调试、接口鉴权等。



在jyadmin中使用`Knife4j `实现Api文档的展示



> `Knife4j ` 官方文档 https://doc.xiaominfo.com/v2/documentation/  `jyadmin使用的版本`
>
> 新版Knife4j：https://doc.xiaominfo.com/ 





## 2、SpringBoot 集成 Knife4j 实现 Api文档

第一步：在maven项目的`pom.xml`中引入Knife4j的依赖包，代码如下：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.7</version>
</dependency>
```



第二步：创建配置依赖

```java
/**
 * Knife4j API文档配置类
 * @author LGX_TvT <br>
 * @version 1.0 <br>
 * Create by 2022-04-04 22:25 <br>
 * @description: JyKnife4jConfiguration <br>
 */
@Configuration
@EnableSwagger2WebMvc // 开启Swagger
public class Knife4jConfig {

    @Resource
    private JyApiDocumentProperties jyApiDocumentProperties;

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(new ApiInfoBuilder()
                        // 更新api描述
                        .description(jyApiDocumentProperties.getDescription())
                        // 服务条款url
                        .termsOfServiceUrl(jyApiDocumentProperties.getTermsOfServiceUrl())
                        // API负责人的联系信息
                        .contact(jyApiDocumentProperties.getContact())
                        // api版本
                        .version(jyApiDocumentProperties.getVersion())
                        .build())
                // 分组名称
                .groupName(jyApiDocumentProperties.getGroupName())
                .select()
                //这里指定Controller扫描包路径
                .apis(RequestHandlerSelectors.basePackage(jyApiDocumentProperties.getBasePackage()))
                .paths(PathSelectors.any())
                .build();
    }
}
```



第三步：添加基础描述注解，`TagController.java`包含一个简单的RESTful接口,代码示例如下：

```java
@Slf4j
@Api(value = "博客标签", tags = {"博客：博客标签接口"})  // 基础描述注解
@RequestMapping("/api/tag")
@RestController
public class TagController {

    @Resource
    private TagService tagService;

    @Idempotent
    @ApiOperation(value = "新增标签", notes = "") // 基础描述注解
    @PreAuthorize("@jy.check('tag:create')")
    @PostMapping("/create")
    public Result<Object> doCreate(@RequestBody @Valid TagCreateVO vo) {
        return ResultUtil.toResult(tagService.save(BeanUtil.copyProperties(vo, Tag.class)));
    }
}
```



第四步：访问`http://localhost:端口号/doc.html`，效果如下：

![image-20230613105929135](Api%E6%96%87%E6%A1%A3%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0.assets/image-20230613105929135-16866251702671.png)



## 3、项目使用

以下是Knife4j中常用的注解及其功能的表格展示，这些注解可以帮助我们更加清晰地描述API接口的信息，使得文档更加易读和易理解。

| 注解名称           | 功能描述                                              |
| ------------------ | ----------------------------------------------------- |
| @Api               | 用于类上，表示该类是Swagger资源                       |
| @ApiOperation      | 用于方法上，说明该方法的作用                          |
| @ApiImplicitParams | 用于方法上，表示一组参数说明                          |
| @ApiImplicitParam  | 用于@ApiImplicitParams注解中，表示单个参数说明        |
| @ApiModel          | 用于类上，表示对类进行说明，用于参数用实体类接收      |
| @ApiModelProperty  | 用于方法或字段上，表示对model属性的说明或数据操作更改 |
| @ApiIgnore         | 用于类、方法、参数、字段上，表示这些元素将被忽略      |
| @ApiParam          | 用于方法上，参数说明                                  |
| @ApiResponse       | 用在@ApiResponses中，一般用于表达一个错误的响应信息   |
| @ApiResponses      | 用于方法上，表示一组响应                              |
| @ApiSort           | 用于排序优先级                                        |



>  注：至于更复杂的使用方式，或者开发者想要自定义请参照官方文档：  https://doc.xiaominfo.com/v2/documentation/ 


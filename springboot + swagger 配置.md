# springboot + swagger 配置

**Swagger简介**

swagger包括三部分: Swagger Editor(基于浏览器的编辑器)，Swagger UI(可以让我们通过浏览器来查看并操作Rest API，Swagger Codegen。

**Swagger接口相关注解说明**

1.@Api：可设置对控制器的描述

2.@ApiOperation：: 可设置对接口的描述

3 .@ApiIgnore: Swagger 文档不会显示拥有该注解的接口。

4 @ApiImplicitParams: 用于描述接口的非对象参数集。

5 @ApiImplicitParam: 用于描述接口的非对象参数，一般与 @ApiImplicitParams 组合使用。

6 @ApiModel:可设置接口相关实体的描述

7 @ApiModelProperty: 可设置实体属性的相关描述。
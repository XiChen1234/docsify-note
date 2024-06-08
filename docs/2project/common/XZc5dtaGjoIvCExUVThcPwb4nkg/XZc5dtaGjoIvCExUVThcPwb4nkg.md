# 通用响应类的设计与使用

# 为什么要做通用响应类设计？

因为在前后端分离的大环境下，后端无法像以前那样只用针对于网页端，将数据渲染成完善的 html 网页之后直接返回给客户端进行展示了，在多种多样的客户端环境中，后端选择直接将数据根据某种格式（前后端交互通常使用 JSON 格式）打包好发送给前端，将页面具体如何渲染的权利交给前端进行，以此降低项目耦合度；同时，使用前后端分离的架构分工明确，前端专心处理用户逻辑，后端专心处理数据格式，以此实现项目分工合作进行开发，提高开发效率

但是，后端的数据结构往往是多种多样的，比如某个用户对象，再比如一个用户信息列表，亦或是一个简单的字符串……非常繁杂。因此，在处理前后端交互的数据的时候，为保证前后端通信顺畅，需要约定相互交流的格式。于是通用响应类接管了所有响应并将数据封装到响应中

# 通用响应类设计的具体实现

我进行通用响应类的设计过程中，将通用响应类设计为了以下三种情形：

- 成功响应（SUCCESS）：能够给客户端提供数据
- 警告响应（WARNING）：一般为客户端方面的错误，包含语法错误或因客户端原因无法完成请求，比如在参数校验过程中发现的参数错误
- 异常响应（ERROR）：服务器内部错误，在处理请求的过程中发生了错误，需要执行异常处理相关操作

基于上面三种场景对通用响应类的规范进行约定：成功响应携带状态码、数据，或者状态码、信息；警告和异常响应携带状态码、错误信息。

同时，为了扩展响应码和异常种类，我编写了响应码枚举类，在其中可以随意添加自定义的响应码（这部分在统一异常处理的文章中再进行规范扩展）

## 响应码枚举类

该类列举了项目当中需要用到的所有响应码，包括常规的成功（200）、警告（400）和异常（500），以及在项目中可以进行自定义的其他异常。

成功为正常状态下的响应；警告为不影响服务端运行的客户端，同时为未进行自定义的异常提供兜底；错误为影响服务端运行的异常，以及为其他开发过程中未考虑的所有异常进行兜底和处理

```java
/**
 * @Description 响应码枚举类
 * 枚举项目中所用到的所有响应码
 */
@Getter
public enum ResponseCode {
    SUCCESS(200, "SUCCESS"),
    WARNING(400, "WARNING"),
    ERROR(500, "ERROR"),

    // 在这之后可以添加自定义响应码，一般为自定义异常响应码
    LOGIN_EMPTY(1000, "登录表单提交为空"),
    LOGIN_ERROR(1001, "登录表单验证失败");

    private final Integer code;
    private final String message;

    ResponseCode(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
}
```

在响应码枚举中可以增加新的响应码，但在这里暂且按下不表，异常响应码的扩展放在统一异常处理的文章中再说

## 通用响应码结构

该类能够保证响应的基本结构为 JSON 格式，并且包含：status、message、data 三个自选大模块，在类内部，构造方法可以根据传参不同创建通用响应类。

但是由于为了适配多种多样的业务数据，data 为泛型，因此需要将构造方法设置为 private，并提供与之配套的创造方法来内部返回创建新的响应类，通过不同的方法创建不同的响应类，来避免歧义。

最后，该类还提供了自定义异常的创建方法（但在本篇中未提及）。这些和上面一样在统一异常处理的文章中进行阐述。

```java
/**
 * @Description 通用响应类结构
 */
@Getter
@JsonInclude(JsonInclude.Include._NON_NULL_) // 空数据不包含
public class CommonResponse<T> {
    private final Integer code;
    private String message;
    private T data;

    // 私有构造方法
    private CommonResponse(Integer code) {
        this.code = code;
    }

    private CommonResponse(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    private CommonResponse(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    /**
     * 创建成功响应方法
     *
     * @return 成功响应
     */
    public static <T> CommonResponse<T> creatForSuccess() {
        return new CommonResponse<>(ResponseCode._SUCCESS_.getCode());
    }

    /**
     * 创建成功响应方法
     *
     * @param message 成功自定义信息
     * @return 成功响应
     */
    public static <T> CommonResponse<T> creatForSuccessMessage(String message) {
        return new CommonResponse<>(ResponseCode._SUCCESS_.getCode(), message);
    }

    /**
     * 创建成功响应方法_
     *
     * @param data 响应数据
     * @return 成功响应
     */
    public static <T> CommonResponse<T> creatForSuccessData(T data) {
        return new CommonResponse<>(ResponseCode._SUCCESS_.getCode(), ResponseCode._SUCCESS_.getMessage(), data);
    }

    /**
     * 创建成功响应方法_
     *
     * @param message 成功自定义信息
     * @param data    响应数据
     * @return 成功响应
     */
    public static <T> CommonResponse<T> creatForSuccessMessageData(String message, T data) {
        return new CommonResponse<>(ResponseCode._SUCCESS_.getCode(), message, data);
    }

    /**
     * 创建警告响应方法
     *
     * @return 异常响应
     */
    public static <T> CommonResponse<T> creatForWarning() {
        return new CommonResponse<>(ResponseCode._WARNING_.getCode());
    }

    /**
     * 创建警告响应方法_
     *
     * @param message 警告自定义信息
     * @return 异常响应
     */
    public static <T> CommonResponse<T> creatForWarningMessage(String message) {
        return new CommonResponse<>(ResponseCode._WARNING_.getCode(), message);
    }

    /**
     * 创建错误响应方法
     *
     * @return 错误响应
     */
    _public static <T> CommonResponse<T> creatForError() {
        return new CommonResponse<>(ResponseCode._ERROR_.getCode());
    }

    /**
     * 创建错误响应方法
     *
     * @param message 错误自定义信息
     * @return 错误响应
     */
    public static <T> CommonResponse<T> creatForErrorMessage(String message) {
        return new CommonResponse<>(ResponseCode._ERROR_.getCode(), message);
    }
}
```

# 通用响应类的使用

在项目基本 MVC 三层结构中，通常在 Service 层中处理数据的处理和封装工作，完成后将完整的数据传递到 Controller 层中，并由 Controller 层发往前端。此时就需要通用响应类将数据进行封装，具体使用示例如下：

```java
/**_
 * 三种通用响应类型_
 *_
 * _**@param **_type 类型_
 *             success：成功200_
 *             warning：警告400_
 *             error：错误500_
 * _**@return **_通用响应类_
 */_
@GetMapping("/common/{type}")
public CommonResponse<String> response(@PathVariable String type) {
    if (Objects._equals_(type, "success")) {
        return CommonResponse._creatForSuccessData_("my data is here");
    }

    if (Objects._equals_(type, "warning")) {
        return CommonResponse._creatForWarning_();
    }

    if (Objects._equals_(type, "error")) {
        return CommonResponse._creatForErrorMessage_("服务器错误");
    }

    return CommonResponse._creatForWarningMessage_("参数判断错误");
}
```

各种 if 条件判断模拟 Controller 层对具体的业务模块流程的控制，在得到指定流程的结果后，将其封装进行返回。

- 访问：[http://localhost:8099/common/success](http://localhost:8099/common/success)，得到 `{"code":200,"message":"SUCCESS","data":"my data is here"}`（data 可以有更复杂的格式）
- 访问：[http://localhost:8099/common/warning](http://localhost:8099/common/success)，得到 `{"code":400}`
- 访问：[http://localhost:8099/common/error](http://localhost:8099/common/success)，得到 `{"code":500,"message":"服务器错误"}`
- 访问：[http://localhost:8099/common/%&saj12a](http://localhost:8099/common/success)，得到 `{"code":400,"message":"参数判断错误"}`

随后将 json 字符串反序列化后即可在前端进行对应的处理

至此，通用响应类的设计已经结束了。但是该模块仍有部分概念存在缺失，如自定义异常的状态码和处理方式的不足，更多内容在统一异常处理的文章中进行阐述，敬请期待

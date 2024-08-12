# 使用

1. 下载

   ~~~go
   go install github.com/swaggo/swag/cmd/swag@v1.8.12
   
   go get -u github.com/swaggo/gin-swagger
   go get -u github.com/swaggo/files
   ~~~

2. 生成 api 文档
   `swag init`

3. 浏览器访问
   `http://ip:host/swagger/index.html`

# 配置

## 示例

~~~go
package main

import (
	_ "Deom_Swagger/docs"
	"github.com/gin-gonic/gin"
	swaggerFiles "github.com/swaggo/files"
	gs "github.com/swaggo/gin-swagger"
)

type Response struct {
	Code int    `json:"code"` //响应码
	Msg  string `json:"msg"`  //描述
	Data any    `json:"data"` //具体的数据
}

// UserList 用户列表
// @Tags 标签_用户管理
// @Summary 用户列表
// @Description 返回一个用户列表，可根据查询参数指定
// @Param limit query string false "返回多少条"
// @Param age query string true "返回多少条"
// @Router /api/users [get]
// @Produce json
// @Success 200 {object} Response
func UserList(ctx *gin.Context) {
	ctx.JSON(400, Response{Code: 200, Msg: "成功", Data: gin.H{"Name": "age", "Age": 18}})
}

// @title zheshishnem
// @description zheshishnem
// @host 127.0.0.1:8086
// @BasePath /
func main() {
	router := gin.Default()
	router.GET("/api/users", UserList)
	router.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))
	err := router.Run("127.0.0.1:8086")
	if err != nil {
		return
	}
}

~~~

## 配置参数

~~~go

注解	描述
@Summary	摘要
@Produce	API 可以产生的 MIME 类型的列表，MIME 类型你可以简单的理解为响应类型，例如：json、xml、html 等等
@Param		参数格式，从左到右分别为：参数名、参数类型、数据类型、是否必填、注释
@Success	响应成功，从左到右分别为：状态码、参数类型、数据类型、注释
@Failure	响应失败，从左到右分别为：状态码、参数类型、数据类型、注释
@Router		路由，从左到右分别为：路由地址，[HTTP方法]

~~~



| 支持的参数类型 | 支持的数据类型                      |
| :------------- | :---------------------------------- |
| query          | string (string)                     |
| path           | integer (int, uint, uint32, uint64) |
| header         | number (float32)                    |
| body           | boolean (bool)                      |
| formData       | user defined struct                 |


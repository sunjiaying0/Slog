# restful

REST（Representational State Transfer，表现层状态转化）是近几年使用较广泛的分布式结点间同步通信的实现方式。REST原则描述网络中client-server的一种交互形式，即用URL定位资源，用HTTP方法描述操作的交互形式。如果CS之间交互的网络接口满足REST风格，则称为RESTful API。以下是 理解RESTful架构 总结的REST原则：
* 网络上的资源通过URI统一标示。
* 客户端和服务器之间传递，这种资源的某种表现层。表现层可以是json，文本，二进制或者图片等。
* 客户端通过HTTP的四个动词，对服务端资源进行操作，实现表现层状态转化。

## go restful

go-restful is a package for building REST-style Web Services using Google Go <br>

go-restful定义了Container WebService和Route三个重要数据结构 

* Route 表示一条路由，包含 URL/HTTP method/输入输出类型/回调处理函数RouteFunction
* WebService 表示一个服务，由多个Route组成，他们共享同一个Root Path
* Container 表示一个服务器，由多个WebService和一个 http.ServerMux 组成，使用RouteSelector进行分发；
    container是根据标准库http的路由器ServeMux写的，并且它通过ServeMux的路由表实现了Handler接口，

最简单的使用实例，向WebService注册路由，将WebService添加到Container中，由Container负责分发。

```go
type User struct {
	Id, Name string
}

type UserResource struct {
	// normally one would use DAO (data access object)
	users map[string]User
}

func (u UserResource) Register(container *restful.Container) {
	ws := new(restful.WebService)
	ws.
		Path("/users").
		Consumes(restful.MIME_XML, restful.MIME_JSON).
		Produces(restful.MIME_JSON, restful.MIME_XML) // you can specify this per route as well

	ws.Route(ws.GET("/{user-id}").To(u.findUser))
	ws.Route(ws.POST("").To(u.updateUser))
	ws.Route(ws.PUT("/{user-id}").To(u.createUser))
	ws.Route(ws.DELETE("/{user-id}").To(u.removeUser))

	container.Add(ws)
}

// GET http://localhost:8080/users/1
//
func (u UserResource) findUser(request *restful.Request, response *restful.Response) {
	id := request.PathParameter("user-id")
	fmt.Println("finduser")
	fmt.Println(u)
	usr , ok := u.users[id]
	if !ok {
		response.AddHeader("Content-Type", "text/plain")
		response.WriteErrorString(http.StatusNotFound, "User could not be found.")
	} else {
		response.WriteEntity(usr)
	}
}

// POST http://localhost:8080/users
// <User><Id>1</Id><Name>Melissa Raspberry</Name></User>
//
func (u *UserResource) updateUser(request *restful.Request, response *restful.Response) {
	usr := new(User)
	err := request.ReadEntity(&usr)
	if err == nil {
		u.users[usr.Id] = *usr
		response.WriteEntity(usr)
	} else {
		response.AddHeader("Content-Type", "text/plain")
		response.WriteErrorString(http.StatusInternalServerError, err.Error())
	}
}

// PUT http://localhost:8080/users/1
// <User><Id>1</Id><Name>Melissa</Name></User>
//
func (u *UserResource) createUser(request *restful.Request, response *restful.Response) {
	usr := User{Id: request.PathParameter("user-id")}
	err := request.ReadEntity(&usr)
	if err == nil {
		u.users[usr.Id] = usr
		response.WriteHeaderAndEntity(http.StatusCreated, usr)
	} else {
		response.AddHeader("Content-Type", "text/plain")
		response.WriteErrorString(http.StatusInternalServerError, err.Error())
	}
}

// DELETE http://localhost:8080/users/1
//
func (u *UserResource) removeUser(request *restful.Request, response *restful.Response) {
	id := request.PathParameter("user-id")
	delete(u.users, id)
}

func main() {
	// 向WebService注册路由，将WebService添加Container中，由Container负责分发
	wsContainer := restful.NewContainer()
	wsContainer.Router(restful.CurlyRouter{})
	fmt.Println(map[string]User{})
	u := UserResource{map[string]User{}}
	fmt.Println(u)
	u.Register(wsContainer)

	log.Printf("start listening on localhost:8080")
	server := &http.Server{Addr: ":8080", Handler: wsContainer}
	log.Fatal(server.ListenAndServe())
}


```
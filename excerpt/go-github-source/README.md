# go-github 源码阅读散记

- 继承

```go
type BasicAuthTransport struct {
	Username string
	Password string
  OTP      string 

  // 继承
	Transport http.RoundTripper
}
```

- 字符串赋值

```go
// 普通字符串
u = fmt.Sprintf("users/%v", user)

// Error 字符串
fmt.Errorf("BaseURL must have a trailing slash, but %q does not", c.BaseURL)
```

- 依赖注入

```go
func NewClient(httpClient *http.Client) *Client {
	if httpClient == nil {
		httpClient = &http.Client{}
	}
	baseURL, _ := url.Parse(defaultBaseURL)
	uploadURL, _ := url.Parse(uploadBaseURL)

  c := &Client{client: httpClient, BaseURL: baseURL, UserAgent: userAgent, UploadURL: uploadURL}
  // 依赖注入
	c.common.client = c
	c.Activity = (*ActivityService)(&c.common)
	c.Admin = (*AdminService)(&c.common)
	c.Apps = (*AppsService)(&c.common)
	c.Authorizations = (*AuthorizationsService)(&c.common)
	c.Checks = (*ChecksService)(&c.common)
	c.Gists = (*GistsService)(&c.common)
	c.Git = (*GitService)(&c.common)
	c.Gitignores = (*GitignoresService)(&c.common)
	c.Interactions = (*InteractionsService)(&c.common)
	c.Issues = (*IssuesService)(&c.common)
	c.Licenses = (*LicensesService)(&c.common)
	c.Marketplace = &MarketplaceService{client: c}
	c.Migrations = (*MigrationService)(&c.common)
	c.Organizations = (*OrganizationsService)(&c.common)
	c.Projects = (*ProjectsService)(&c.common)
	c.PullRequests = (*PullRequestsService)(&c.common)
	c.Reactions = (*ReactionsService)(&c.common)
	c.Repositories = (*RepositoriesService)(&c.common)
	c.Search = (*SearchService)(&c.common)
  c.Teams = (*TeamsService)(&c.common)
  // 依赖注入
	c.Users = (*UsersService)(&c.common)
	return c
```

- 控制反转

```go
func (s *UsersService) Get(ctx context.Context, user string) (*User, *Response, error) {
	var u string
	if user != "" {
		u = fmt.Sprintf("users/%v", user)
	} else {
		u = "user"
  }
  // 控制反转，使用注入的 client 新建一个 NewRequest
	req, err := s.client.NewRequest("GET", u, nil)
	if err != nil {
		return nil, nil, err
	}

	uResp := new(User)
	resp, err := s.client.Do(ctx, req, uResp)
	if err != nil {
		return nil, resp, err
	}

	return uResp, resp, nil
}
```

- 定义常量

```go
const (
	defaultBaseURL = "https://api.github.com/"
	uploadBaseURL  = "https://uploads.github.com/"
)
```

- 指定分页参数

```go
// url tag 似乎可以被 URL 解析
type ListOptions struct {
	// For paginated result sets, page of results to retrieve.
	Page int `url:"page,omitempty"`

	// For paginated result sets, the number of results to include per page.
	PerPage int `url:"per_page,omitempty"`
}
```

- 定义常量

```go
const (
	// Diff format.
	Diff RawType = 1 + iota
	// Patch format.
	Patch // 2 + iota
)
```

- 解析 URL

```go
u, err := url.Parse(s)
```

- 反射

```go
type Parameter struct {
	C int `url:"c"`
	D int `url:"d"`
}

p := Parameter{
	C: 1,
	D: 2,
}

val := reflect.ValueOf(v) // reflect.Value

// reflect.Value
type Value struct {
	// typ holds the type of the value represented by a Value.
	typ *rtype

	// Pointer-valued data or, if flagIndir is set, pointer to data.
	// Valid when either flagIndir is set or typ.pointers() is true.
	ptr unsafe.Pointer

	// flag holds metadata about the value.
	// The lowest bits are flag bits:
	//	- flagStickyRO: obtained via unexported not embedded field, so read-only
	//	- flagEmbedRO: obtained via unexported embedded field, so read-only
	//	- flagIndir: val holds a pointer to the data
	//	- flagAddr: v.CanAddr is true (implies flagIndir)
	//	- flagMethod: v is a method value.
	// The next five bits give the Kind of the value.
	// This repeats typ.Kind() except for method values.
	// The remaining 23+ bits give a method number for method values.
	// If flag.kind() != Func, code can assume that flagMethod is unset.
	// If ifaceIndir(typ), code can assume that flagIndir is set.
	flag

	// A method value represents a curried method invocation
	// like r.Read for some receiver r. The typ+val+flag bits describe
	// the receiver r, but the flag's Kind bits say Func (methods are
	// functions), and the top bits of the flag give the method number
	// in r's type's method table.
}

// 如果 val 的类型为 reflect.Ptr 则可以通过 Elem() 方法获取指针指向的值
val = val.Elem()

// Kind() 方法可以获取 reflect 对象的类型
kind := val.Kind() // reflect.Struct

// Type() 方法可以获取 reflect 对象的原始类型，是一个可操作的对象，对应 reflect.Type
typ := val.Type() // Parameter

// NumField 获取结构体 struct 的字段数量
typ.NumField()

// 获取指定下标的字段，返回 reflect.StructField 类型
sf := type.Field(i)

// A StructField describes a single field in a struct.
type StructField struct {
	// Name is the field name.
	Name string
	// PkgPath is the package path that qualifies a lower case (unexported)
	// field name. It is empty for upper case (exported) field names.
	// See https://golang.org/ref/spec#Uniqueness_of_identifiers
	PkgPath string

	Type      Type      // field type
	Tag       StructTag // field tag string
	Offset    uintptr   // offset within struct, in bytes
	Index     []int     // index sequence for Type.FieldByIndex
	Anonymous bool      // is an embedded field
}

fmt.Println(sf.Name) // C
fmt.Println(sf.Tag) // url:"c"
fmt.Println(sf.Type) // int

// 返回的是 reflect.Value
val.Field(i)

// StructField.Tag.Get 用于获取 struct 的 tag 标签
tag := sf.Tag.Get("url") // 从 C int `url:"c"` 中获取到的值为 c

// tag 为 "-" 时，默认忽略该 tag
if tag == "-" {
	continue
}

// 通过反射获取一个自定义 struct 的类型
var timeType = reflect.TypeOf(time.Time{})

// 判断一个自定义类型是否为空
if v.Type() == timeType {
	return v.Interface().(time.Time).IsZero()
}

```
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
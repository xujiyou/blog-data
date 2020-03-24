# Etcd API (七) - Auth

```protobuf
service Auth {
  // AuthEnable enables authentication.
  rpc AuthEnable(AuthEnableRequest) returns (AuthEnableResponse) {
      option (google.api.http) = {
        post: "/v3/auth/enable"
        body: "*"
    };
  }

  // AuthDisable disables authentication.
  rpc AuthDisable(AuthDisableRequest) returns (AuthDisableResponse) {
      option (google.api.http) = {
        post: "/v3/auth/disable"
        body: "*"
    };
  }

  // Authenticate processes an authenticate request.
  rpc Authenticate(AuthenticateRequest) returns (AuthenticateResponse) {
      option (google.api.http) = {
        post: "/v3/auth/authenticate"
        body: "*"
    };
  }

  // UserAdd adds a new user. User name cannot be empty.
  rpc UserAdd(AuthUserAddRequest) returns (AuthUserAddResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/add"
        body: "*"
    };
  }

  // UserGet gets detailed user information.
  rpc UserGet(AuthUserGetRequest) returns (AuthUserGetResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/get"
        body: "*"
    };
  }

  // UserList gets a list of all users.
  rpc UserList(AuthUserListRequest) returns (AuthUserListResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/list"
        body: "*"
    };
  }

  // UserDelete deletes a specified user.
  rpc UserDelete(AuthUserDeleteRequest) returns (AuthUserDeleteResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/delete"
        body: "*"
    };
  }

  // UserChangePassword changes the password of a specified user.
  rpc UserChangePassword(AuthUserChangePasswordRequest) returns (AuthUserChangePasswordResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/changepw"
        body: "*"
    };
  }

  // UserGrant grants a role to a specified user.
  rpc UserGrantRole(AuthUserGrantRoleRequest) returns (AuthUserGrantRoleResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/grant"
        body: "*"
    };
  }

  // UserRevokeRole revokes a role of specified user.
  rpc UserRevokeRole(AuthUserRevokeRoleRequest) returns (AuthUserRevokeRoleResponse) {
      option (google.api.http) = {
        post: "/v3/auth/user/revoke"
        body: "*"
    };
  }

  // RoleAdd adds a new role. Role name cannot be empty.
  rpc RoleAdd(AuthRoleAddRequest) returns (AuthRoleAddResponse) {
      option (google.api.http) = {
        post: "/v3/auth/role/add"
        body: "*"
    };
  }

  // RoleGet gets detailed role information.
  rpc RoleGet(AuthRoleGetRequest) returns (AuthRoleGetResponse) {
      option (google.api.http) = {
        post: "/v3/auth/role/get"
        body: "*"
    };
  }

  // RoleList gets lists of all roles.
  rpc RoleList(AuthRoleListRequest) returns (AuthRoleListResponse) {
      option (google.api.http) = {
        post: "/v3/auth/role/list"
        body: "*"
    };
  }

  // RoleDelete deletes a specified role.
  rpc RoleDelete(AuthRoleDeleteRequest) returns (AuthRoleDeleteResponse) {
      option (google.api.http) = {
        post: "/v3/auth/role/delete"
        body: "*"
    };
  }

  // RoleGrantPermission grants a permission of a specified key or range to a specified role.
  rpc RoleGrantPermission(AuthRoleGrantPermissionRequest) returns (AuthRoleGrantPermissionResponse) {
      option (google.api.http) = {
        post: "/v3/auth/role/grant"
        body: "*"
    };
  }

  // RoleRevokePermission revokes a key or range permission of a specified role.
  rpc RoleRevokePermission(AuthRoleRevokePermissionRequest) returns (AuthRoleRevokePermissionResponse) {
      option (google.api.http) = {
        post: "/v3/auth/role/revoke"
        body: "*"
    };
  }
}
```

一共 16 个 rpc 方法

```go
type AuthServer interface {
   AuthEnable(context.Context, *AuthEnableRequest) (*AuthEnableResponse, error)
   AuthDisable(context.Context, *AuthDisableRequest) (*AuthDisableResponse, error)
   Authenticate(context.Context, *AuthenticateRequest) (*AuthenticateResponse, error)
   UserAdd(context.Context, *AuthUserAddRequest) (*AuthUserAddResponse, error)
   UserGet(context.Context, *AuthUserGetRequest) (*AuthUserGetResponse, error)
   UserList(context.Context, *AuthUserListRequest) (*AuthUserListResponse, error)
   UserDelete(context.Context, *AuthUserDeleteRequest) (*AuthUserDeleteResponse, error)
   UserChangePassword(context.Context, *AuthUserChangePasswordRequest) (*AuthUserChangePasswordResponse, error)
   UserGrantRole(context.Context, *AuthUserGrantRoleRequest) (*AuthUserGrantRoleResponse, error)
   UserRevokeRole(context.Context, *AuthUserRevokeRoleRequest) (*AuthUserRevokeRoleResponse, error)
   RoleAdd(context.Context, *AuthRoleAddRequest) (*AuthRoleAddResponse, error)
   RoleGet(context.Context, *AuthRoleGetRequest) (*AuthRoleGetResponse, error)
   RoleList(context.Context, *AuthRoleListRequest) (*AuthRoleListResponse, error)
   RoleDelete(context.Context, *AuthRoleDeleteRequest) (*AuthRoleDeleteResponse, error)
   RoleGrantPermission(context.Context, *AuthRoleGrantPermissionRequest) (*AuthRoleGrantPermissionResponse, error)
   RoleRevokePermission(context.Context, *AuthRoleRevokePermissionRequest) (*AuthRoleRevokePermissionResponse, error)
}
```





## AuthEnable

```go
type AuthEnableResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
}
```

认证开启前需要手动添加 root 账号：

```bash
$ etcdctl user add root
$ sudo etcdctl auth enable
```



## AuthDisable

```bash
$ sudo etcdctl auth disable --user=root
```


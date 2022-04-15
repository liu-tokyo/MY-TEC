# Keycloak基本注意事项

## 环境

- 已安装 OpenJDK (ver 11.x)
- 我使用 Keycloak 的目标是向 REST API 添加身份验证（因此内容更接近）

## 安装和启动

- 安装
  Keycloak 是一个 Java 应用程序，因此如果包含 Java，它就可以工作。
  只需从[此处](https://www.keycloak.org/downloads)下载 zip，解压到适当的位置并运行它。

- 启动

  ```shell
  cd keycloak-12.0.4/
  
  bash bin/standalone.sh
  ```

使用standalone.sh -b 0.0.0.0 允许来自本地主机以外的访问。

## 动作确认

- 初始设置


  ```shell
  http://localhost:8080/auth
  HTTP：//本地主机：8080
  ```

  当您访问它时，将显示以下屏幕，因此请创建一个管理员帐户。

  由于这是本地测试，请将 ID 和 PW 设置为 admin 和 admin（不要复制它们）。

- 单击管理控制台

  重定向到管理屏幕的首页，可以在这里做各种各样的事情。

  Keycloak 有 Realm 的概念，即顶级分组管理单元。

## 尝试连接 API

### 获取基本设置信息
查看各种设置和endpoint端点（省略输出）。

```
curl -s \
http://localhost:8080/auth/realms/master/.well-known/openid-configuration | python -m json.tool
```

提取用于获取令牌的Endpoint端点。

```
curl -s \
http://localhost:8080/auth/realms/master/.well-known/openid-configuration | jq -r ".token_endpoint"

http://localhost:8080/auth/realms/master/protocol/openid-connect/token
```

如果不能使用 `jq` 命令，请使用 `brew install jq`。

### 获取请求中使用的客户端名称和密码

#### 客户

![スクリーンショット_2021-03-14_12_07_28.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F55188%2F71ffc020-d272-2955-388e-e90e6f7d39b6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2c653457ae3627fa411faacbd02a8b6c)

#### secret

![スクリーンショット_2021-03-14_12_07_45.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F55188%2Fe549aa87-a403-feb3-38e9-383834c821cd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=896b5868e1ce47c4eb359a745ace86ea)

### 获取令牌

尝试使用默认存在的“account”帐户。

```
curl -s \
 -d "client_id=account" \
 -d "client_secret=19beb1ef-d25b-45e7-bb66-6790562e0f04" \
 -d "username=admin" \
 -d "password=admin" \
 -d "grant_type=password" \
 -d "scope=openid" \
"http://localhost:8080/auth/realms/master/protocol/openid-connect/token" | python -m json.tool
```

> 您可以使用上述命令获取令牌，因为 Client 设置为 [Direct Access Grants Enabled]。 通过设置 scope = openid，id_token 也会返回。

返回很多东西：

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJYMGk0Mk1CU2dFb1FGeGpZZEhDOWNzTmlxcS1WdFIwdW53bEVoampCdFpNIn0.eyJleHAiOjE2MTU2OTA2MDksImlhdCI6MTYxNTY5MDU0OSwianRpIjoiMmI3ZGEwMTAtNzkxYy00MzI1LWE4ZjgtOTI0ZDdmODNhODhmIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL21hc3RlciIsInN1YiI6IjA0OTI1NjIwLTBlOWYtNDRkMC1iNDRhLTA4MGUxZmIwZWRmMCIsInR5cCI6IkJlYXJlciIsImF6cCI6ImFjY291bnQiLCJzZXNzaW9uX3N0YXRlIjoiNGJlZjQzNDQtNmQ4MC00MDA0LWFiYmYtYWNhZjNhYWU5NGE2IiwiYWNyIjoiMSIsInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGVtYWlsIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJhZG1pbiJ9.r96bNksy85Kh-E9G0c4WOHpEuoy7f7KUFDrhV3iORCXz5wPaGKktLxkLfYpnd1xrijRs-amWvic2ZvzCii_pOTozv8kRm1QmGL54h4lOY2Q8PZCAgbDcyajmOoDedweOnGqTzOSGaBJftoMsd0bcoSqxiPtF5N6bcd1_aw-BRaP6HDiGutiKe9KAHI8g_1w-gq4UmVkh-OSa-B8FVXUw6S-gsyMxO6UAa0A_W13H4rXCc9LzpMMtTh3WR1VMoQ3VX8GpzVl9jA9uAI67yzI0RhUeGgBNaHXXgOPVUOj0abNsAXfHh0kVbIaAKWqCML7e2fafCnqVOL7OGUa931JR4g",
    "expires_in": 60,
    "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJYMGk0Mk1CU2dFb1FGeGpZZEhDOWNzTmlxcS1WdFIwdW53bEVoampCdFpNIn0.eyJleHAiOjE2MTU2OTA2MDksImlhdCI6MTYxNTY5MDU0OSwiYXV0aF90aW1lIjowLCJqdGkiOiJhMjU1NGU1OS0wMjgyLTQ1MTItOTdmNC1lN2QyZjQ5YmVmMGEiLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvbWFzdGVyIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjA0OTI1NjIwLTBlOWYtNDRkMC1iNDRhLTA4MGUxZmIwZWRmMCIsInR5cCI6IklEIiwiYXpwIjoiYWNjb3VudCIsInNlc3Npb25fc3RhdGUiOiI0YmVmNDM0NC02ZDgwLTQwMDQtYWJiZi1hY2FmM2FhZTk0YTYiLCJhdF9oYXNoIjoiRHVyeEpYS3UzX2ltNHRUY2FKbTE3QSIsImFjciI6IjEiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6ImFkbWluIn0.DvohU3Cw8yDLGFHf20PyBV6rxkXilfoe76SMdNm-HxVqamQyJdBPFGPYGk4dUqJVjcVCjit6dgd_Q_hL3y5nGac44UIfZsFaGJ7VFJ_r9YbjcaT3ppDwLj3_yBxmxBsThwsHo1n6WpQVPXALhpleVTa6yMgS13IovOG6QL-dILnweNBNg4LYWd8tAkgbB96S40G0jwlWNwL3Kt4tR2mjZMipjznjhC23W2FgubeNjxwTI8WSvYwGOawlpBemeXu-ma-Y_cLu-VHU7UL8mb1U0GdQbYXt04CNWpo7nSF-H2JuYAnuu_9ivkJ32tlVFitHyLKjFwBOuptizPdtuUUeOQ",
    "not-before-policy": 0,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI0N2Y5NmFjMS02ZTBhLTQ5NGItYjdlYS02OTk1NDE0MGIwNzgifQ.eyJleHAiOjE2MTU2OTIzNDksImlhdCI6MTYxNTY5MDU0OSwianRpIjoiOTYzMzFhNGYtMWY5OC00YzUzLWFjOTgtYzcxNDk1YzU0ZGM3IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL21hc3RlciIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4MC9hdXRoL3JlYWxtcy9tYXN0ZXIiLCJzdWIiOiIwNDkyNTYyMC0wZTlmLTQ0ZDAtYjQ0YS0wODBlMWZiMGVkZjAiLCJ0eXAiOiJSZWZyZXNoIiwiYXpwIjoiYWNjb3VudCIsInNlc3Npb25fc3RhdGUiOiI0YmVmNDM0NC02ZDgwLTQwMDQtYWJiZi1hY2FmM2FhZTk0YTYiLCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGVtYWlsIn0.5oOznYptdgOwjK24vYZptDEyu_O4HYLYFp4Vlt7frcA",
    "scope": "openid profile email",
    "session_state": "4bef4344-6d80-4004-abbf-acaf3aae94a6",
    "token_type": "Bearer"
}
```

## 令牌验证

- JWT 可以独立验证，但它也有一个 Endpoint 用于验证：

  ```shell
  curl -s -X POST \
  -d "client_id=account" \
  -d "client_secret=19beb1ef-d25b-45e7-bb66-6790562e0f04" \
  -d "token=eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJYMGk0Mk1CU2dFb1FGeGpZZEhDOWNzTmlxcS1WdFIwdW53bEVoampCdFpNIn0.eyJleHAiOjE2MTU2OTA2MDksImlhdCI6MTYxNTY5MDU0OSwianRpIjoiMmI3ZGEwMTAtNzkxYy00MzI1LWE4ZjgtOTI0ZDdmODNhODhmIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL21hc3RlciIsInN1YiI6IjA0OTI1NjIwLTBlOWYtNDRkMC1iNDRhLTA4MGUxZmIwZWRmMCIsInR5cCI6IkJlYXJlciIsImF6cCI6ImFjY291bnQiLCJzZXNzaW9uX3N0YXRlIjoiNGJlZjQzNDQtNmQ4MC00MDA0LWFiYmYtYWNhZjNhYWU5NGE2IiwiYWNyIjoiMSIsInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGVtYWlsIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJhZG1pbiJ9.r96bNksy85Kh-E9G0c4WOHpEuoy7f7KUFDrhV3iORCXz5wPaGKktLxkLfYpnd1xrijRs-amWvic2ZvzCii_pOTozv8kRm1QmGL54h4lOY2Q8PZCAgbDcyajmOoDedweOnGqTzOSGaBJftoMsd0bcoSqxiPtF5N6bcd1_aw-BRaP6HDiGutiKe9KAHI8g_1w-gq4UmVkh-OSa-B8FVXUw6S-gsyMxO6UAa0A_W13H4rXCc9LzpMMtTh3WR1VMoQ3VX8GpzVl9jA9uAI67yzI0RhUeGgBNaHXXgOPVUOj0abNsAXfHh0kVbIaAKWqCML7e2fafCnqVOL7OGUa931JR4g" \
  "http://localhost:8080/auth/realms/master/protocol/openid-connect/token/introspect" | python -m json.tool
  ```

- 有效的情况：

  ```yaml
  {
      "acr": "1",
      "active": true,
      "azp": "account",
      "client_id": "account",
      "email_verified": false,
      "exp": 1615691183,
      "iat": 1615691123,
      "iss": "http://localhost:8080/auth/realms/master",
      "jti": "197e3010-445a-4bdd-b506-4122121964a3",
      "preferred_username": "admin",
      "resource_access": {
          "account": {
              "roles": [
                  "manage-account",
                  "manage-account-links",
                  "view-profile"
              ]
          }
      },
      "scope": "openid profile email",
      "session_state": "b3bf3cd2-bcb8-436c-b681-28b3cfa9427e",
      "sub": "04925620-0e9f-44d0-b44a-080e1fb0edf0",
      "typ": "Bearer",
      "username": "admin"
  }
  ```

- 无效的情况

  ```yaml
  {
      "active": false
  }
  ```

根据active的true、false来进行判断。

## Userinfo

获取用户信息：

> 由于超时，访问令牌与上述不同。

```shell
curl -s -X GET \
-H "Content-Type: application/x-www-form-urlencoded" \
-H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJYMGk0Mk1CU2dFb1FGeGpZZEhDOWNzTmlxcS1WdFIwdW53bEVoampCdFpNIn0.eyJleHAiOjE2MTU3NDY1MDIsImlhdCI6MTYxNTc0NjQ0MiwianRpIjoiMmIyNWE0ZjctN2NhZS00MThlLWE0ZDItOTY0OTBlODUxMWJmIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL21hc3RlciIsInN1YiI6IjA0OTI1NjIwLTBlOWYtNDRkMC1iNDRhLTA4MGUxZmIwZWRmMCIsInR5cCI6IkJlYXJlciIsImF6cCI6ImFjY291bnQiLCJzZXNzaW9uX3N0YXRlIjoiZDJhMjQ0NTItMTU4Ni00M2IzLTlkM2QtZTVkMDUwYzI5NzBmIiwiYWNyIjoiMSIsInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGVtYWlsIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJhZG1pbiJ9.k0tGO2eJf7fJJaPtq6AYGdYy0h1aAVwkKhwC7m08_JUXWv_ZZqH0XNflgt1aYJLKFXB-oxoPJhjLYObyoH84f10DDhI2AKKB2JBtm_qjyDATsOs4haOoaEYi9dS8X-Le4C1jZ23OVCWdxH_xdl_bKdjVVPMe2w7zfqNiXo44ZxkKDY5uHqO1g4GBW7RvRCy6vCcCm622ijO-YfaMrtTqc4e3QkrI7l-PX1hDeqNFGid5zId6YwVnnY62WUznj26uOd3j9Zc7f4i-gXoBkoN3nGbVIzu1Ycg8ebYOkkMTu7ENI1ZetCzL9Q-XuzHo_j-SCeAs_anni8CU9RJjmWZsiA" \
"http://localhost:8080/auth/realms/master/protocol/openid-connect/userinfo" | python -m json.tool
```

暂时会返回以下内容，我想知道如何自定义、检查。

```yaml
{
    "email_verified": false,
    "preferred_username": "admin",
    "sub": "04925620-0e9f-44d0-b44a-080e1fb0edf0"
}
```



- 参照

  https://qiita.com/zaburo/items/6df185803cb019cd257c

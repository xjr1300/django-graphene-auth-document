# API

<https://django-graphene-auth.readthedocs.io/en/latest/api/>

- [API](#api)
  - [クエリ](#クエリ)
    - [UserQuery](#userquery)
      - [例](#例)
    - [MeQuery](#mequery)
      - [例](#例-1)
  - [ミューテーション](#ミューテーション)
    - [標準的なレスポンス](#標準的なレスポンス)
    - [公開されたミューテーション - Public](#公開されたミューテーション---public)
      - [ObtainJSONWebToken](#obtainjsonwebtoken)
      - [PasswordSet](#passwordset)
      - [PasswordReset](#passwordreset)
      - [RefreshToken](#refreshtoken)
      - [Register](#register)
      - [ResendActivationEmail](#resendactivationemail)
      - [RevokeToken](#revoketoken)
      - [SendPasswordResetEmail](#sendpasswordresetemail)
      - [VerifyAccount](#verifyaccount)
      - [VerifySecondaryEmail](#verifysecondaryemail)
      - [VerifyToken](#verifytoken)
    - [保護されたミューテーション - Protected](#保護されたミューテーション---protected)
      - [ArchiveAccount](#archiveaccount)
      - [DeleteAccount](#deleteaccount)
      - [PasswordChange](#passwordchange)
      - [RemoveSecondaryEmail](#removesecondaryemail)
      - [SendSecondaryEmailActivation](#sendsecondaryemailactivation)
      - [SwapEmails](#swapemails)
      - [UpdateAccount](#updateaccount)

## クエリ

GraphQL Authは、いくつかの便利なフィルタが付属したユーザーを問い合わせする`UserQuery`を提供します。
また、GraphQL Authは、現在の認証されたユーザーのデータを取得する`MeQuery`も提供しています。

### UserQuery

```python
from graphql_auth.schema import UserQuery
```

`UserQuery`を探求する最も簡単な方法は、[graphQL](https://docs.graphene-python.org/projects/django/en/latest/tutorial-plain/#creating-graphql-and-graphiql-views)を使用することです。

#### 例

クエリ1

```graphql
query {
  users {
    edge {
      node {
        username
        archived
        verified
        email
        secondEmail
      }
    }
  }
}
```

クエリ1のレスポンス

```json
{
  "data": {
    "users": {
      "edges": [
        {
          "node": {
            "username": "user1",
            "archived": false,
            "verified": false,
            "email": "user1@email.com",
            "secondaryEmail": null
          }
        },
        {
          "node": {
            "username": "user2",
            "archived": false,
            "verified": true,
            "email": "user2@email.com",
            "secondaryEmail": null
          }
        },
        {
          "node": {
            "username": "user3",
            "archived": true,
            "verified": true,
            "email": "user3@email.com",
            "secondaryEmail": null
          }
        },
        {
          "node": {
            "username": "user4",
            "archived": false,
            "verified": true,
            "email": "user4@email.com",
            "secondaryEmail": "user4_secondary@email.com"
          }
        }
      ]
    }
  }
}
```

クエリ2

```graphql
query {
  users (last: 1) {
    edges {
      node {
        id
        username
        email
        isActive
        archived
        verified
        secondaryEmail
      }
    }
  }
}
```

クエリ2のレスポンス

```json
{
  "data": {
    "users": {
      "edges": [
        {
          "node": {
            "id": "VXNlck5vZGU6NQ==",
            "username": "new_user",
            "email": "new_user@email.com",
            "isActive": true,
            "archived": false,
            "verified": false,
            "secondaryEmail": null
          }
        }
      ]
    }
  }
}
```

クエリ3

```graphql
query {
  user (id: "VXNlck5vZGU6NQ==") {
    username
    verified
  }
}
```

クエリ3のレスポンス

```json
{
  "data": {
    "user": {
      "username": "new_user",
      "verified": true
    }
  }
}
```

### MeQuery

```python
from graphql_auth.schema import MeQuery
```

このクエリは認証されたユーザーを要求するため、[insomnia APIクライアント](https://insomnia.rest/)を使用することで探求できます。

#### 例

クエリ

```graphql
query {
  me {
    username
    verified
  }
}
```

クエリのレスポンス

```json
{
  "data": {
    "user": {
      "username": "new_user",
      "verified": true
    }
  }
}
```

## ミューテーション

すべてのミューテーションは、次のようにインポートされます。

```python
# GraphQLの場合
from graphql_auth import mutations

# ミューテーションで
class SomeMutation(graphene.ObjectType):
    register = mutations.Register.Field()
    # ...
```

```python
# Relayの場合
from graphql_auth import relay

# ミューテーションで
class SomeMutation(graphene.ObjectTyep):
    register = relay.Register.Field()
    # ...
```

### 標準的なレスポンス

すべてのミューテーションは、`errors`と`success`を含む標準的なレスポンスを変えぢます。

GraphQLのリクエスト例

```graphql
mutation {
  register(
    email: "new_user@email.com"
    username: "new_user"
    password1: "123456"
    password2: "123456"
  ) {
    success
    errors
    token
    refreshToken
  }
}
```

レスポンス

```json
{
  "data": {
    "register": {
      "success": false,
      "errors": {
        "password2": [
          {
            "message": "This password is too short. It must contain at least 8 characters.",
            "code": "password_too_short"
          },
          {
            "message": "This password is too common.",
            "code": "password_too_common"
          },
          {
            "message": "This password is entirely numeric.",
            "code": "password_entirely_numeric"
          }
        ]
      },
      "token": null
      "refreshToken": null
    }
  }
}
```

relayのレスポンス例

```graphql
mutation {
  register(
    input: {
      email: "new_user@email.com",
      username: "new_user",
      password1: "123456",
      password2: "123456",
    }
  ) {
    success,
    errors,
    token,
    refreshToken
  }
}
```

### 公開されたミューテーション - Public

パブリックなミューテーションは、ユーザーがログインする必要はありません。
`GRAPHQL_JWT["JWT_ALLOW_ANY_CLASSES"]`にすべてのパブリックなミューテーションを追加しなければなりません。

---

#### ObtainJSONWebToken

ユーザーに与えるためにJSON Webトークンを取得します。

さまざまなフィールド、それと2つ目のEメールが設定されている場合にそれを使用してログインを実行できるようにします。
フィールドは`settings`で定義されます。
デフォルトでユーザーアカウントが検証されていないユーザーがログインできます。
これは`settings`で変更できます。
もしユーザーがアーカイブされた場合、それはアーカイブをやめて、出力で`unarchiving＝True`を返します。

GraphQLのリクエスト例

```graphql
mutation {
  tokenAuth(
    # username or email
    email: "skywalker@email.com"
    password: "123456super"
  ) {
    success,
    errors,
    token,
    refreshToken,
    unarchiving,
    user {
      id,
      username
    }
  }
}
```

relayのレスポンス例

```graphql
mutation {
  tokenAuth(
    input: {
      email: "skywalker@email.com"
      password: "123456super"
    }
  ) {
    success,
    errors,
    token,
    refreshToken,
    user {
      id,
      username
    }
  }
}
```

成功したときのレスポンス

```json
{
  "data": {
    "tokenAuth": {
      "success": true,
      "errors": null,
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImV4cCI6MTU3OTQ1ODI2Niwib3JpZ0lhdCI6MTU3OTQ1Nzk2Nn0.BKz4ohxQCGtJWnyd5tIbYFD2kpGYDiAVzWTDO2ZyUYY",
      "refreshToken": "5f5fad67cd043437952ddde2750be20201f1017b",
      "unarchiving": false,
      "user": {
        "id": "VXNlck5vZGU6MQ==",
        "username": "skywalker"
      }
    }
  }
}
```

クレデンシャルが誤っているときのレスポンス

```json
{
  "data": {
    "tokenAuth": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Please, enter valid credentials.",
            "code": "invalid_credentials"
          }
        ]
      },
      "token": null,
      "refreshToken": null,
      "unarchiving": false,
      "user": null
    }
  }
}
```

#### PasswordSet

パスワードなし登録のために、ユーザーパスワードを設定します。
Eメールによって送信されたトークンを受け取ってください。
もし、トークンと新しいパスワードが妥当である場合、ユーザーのパスワードが設定され、トークンの更新が使用された場合、それらすべてが無効になります。
また、ユーザーがユーザーアカウントが検証されていない場合、検証されます。

GraphQLのリクエスト例

```graphql
mutation {
  passwordSet(
    token: "1eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImFjdGlvbiI6InBhc3N3b3JkX3Jlc2V0In0:1itExL:op0roJi-ZbO9cszNEQMs5mX3c6s",
    newPassword1: "supersecretpassword",
    newPassword2: "supersecretpassword"
  ) {
    success,
    errors
  }
}
```

relayのレスポンス例

```graphql
mutation {
  passwordSet(
    input: {
      token: "1eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImFjdGlvbiI6InBhc3N3b3JkX3Jlc2V0In0:1itExL:op0roJi-ZbO9cszNEQMs5mX3c6s",
      newPassword1: "supersecretpassword",
      newPassword2: "supersecretpassword"
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "passwordSet": {
      "success": true,
      "errors": null
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "passwordSet": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      }
    }
  }
}
```

パスワードが一致しないときのレスポンス例

```json
{
  "data": {
    "passwordSet": {
      "success": false,
      "errors": {
        "newPassword2": [
          {
            "message": "The two password fields didn’t match.",
            "code": "password_mismatch"
          }
        ]
      }
    }
  }
}
```

パスワードの検証に失敗したときのレスポンス例

```json
{
  "data": {
    "passwordSet": {
      "success": false,
      "errors": {
        "newPassword2": [
          {
            "message": "This password is too short. It must contain at least 8 characters.",
            "code": "password_too_short"
          },
          {
            "message": "This password is too common.",
            "code": "password_too_common"
          },
          {
            "message": "This password is entirely numeric.",
            "code": "password_entirely_numeric"
          }
        ]
      }
    }
  }
}
```

パスワードがすでに設定されているときのレスポンス例

```json
{
  "data": {
    "passwordSet": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Password already set for account.",
            "code": "password_already_set"
          }
        ]
      }
    }
  }
}
```

#### PasswordReset

古いパスワードなしでユーザーのパスワードを変更します。
Eメールによって送信されたトークンを受け取ってください。
トークンと新しいパスワードが妥当な場合、ユーザーのパスワードを更新して、トークンが更新された場合、それらすべてを無効にします。
また、ユーザーがユーザーアカウントが検証されていない場合、検証します。

GraphQLのリクエスト例

```graphql
mutation {
  passwordReset(
    token: "1eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImFjdGlvbiI6InBhc3N3b3JkX3Jlc2V0In0:1itExL:op0roJi-ZbO9cszNEQMs5mX3c6s",
    newPassword1: "supersecretpassword",
    newPassword2: "supersecretpassword"
  ) {
    success,
    errors
  }
}
```

relayのレスポンス例

```graphql
mutation {
  passwordReset(
    input: {
      token: "1eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImFjdGlvbiI6InBhc3N3b3JkX3Jlc2V0In0:1itExL:op0roJi-ZbO9cszNEQMs5mX3c6s",
      newPassword1: "supersecretpassword",
      newPassword2: "supersecretpassword"
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "passwordReset": {
      "success": true,
      "errors": null
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "passwordReset": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      }
    }
  }
}
```

パスワードが一致しないときのレスポンス例

```json
{
  "data": {
    "passwordReset": {
      "success": false,
      "errors": {
        "newPassword2": [
          {
            "message": "The two password fields didn’t match.",
            "code": "password_mismatch"
          }
        ]
      }
    }
  }
}
```

パスワードの検証に失敗したときのレスポンス例

```json
{
  "data": {
    "passwordReset": {
      "success": false,
      "errors": {
        "newPassword2": [
          {
            "message": "This password is too short. It must contain at least 8 characters.",
            "code": "password_too_short"
          },
          {
            "message": "This password is too common.",
            "code": "password_too_common"
          },
          {
            "message": "This password is entirely numeric.",
            "code": "password_entirely_numeric"
          }
        ]
      }
    }
  }
}
```

#### RefreshToken

`graphql_jwt`の実装と同様に、例外エラーメッセージを"Error decoding signature"から"Invalid token"に変更しています。

GraphQLのリクエスト例

```graphql
mutation {
  refreshToken(
    refreshToken: "d9b58dce41cf14549030873e3fab3be864f76ce44"
  ) {
    success,
    errors,
    payload,
    refreshExpiresIn,
    token,
    refreshToken
  }
}
```

relayのレスポンス例

```graphql
mutation {
  refreshToken(
    input: {
      refreshToken: "d9b58dce41cf14549030873e3fab3be864f76ce44"
    }
  ) {
    success,
    errors,
    payload,
    refreshExpiresIn,
    token,
    refreshToken
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "refreshToken": {
      "success": true,
      "errors": null,
      "payload": {
        "username": "skywalker",
        "exp": 1601646082,
        "origIat": 1601645782
      },
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImV4cCI6MTYwMTY0NjA4Miwib3JpZ0lhdCI6MTYwMTY0NTc4Mn0.H6gLeky7lX834kBI5RFT8ziNNfGOL3XXg1dRwvpQuRI",
      "refreshToken": "a64f732b4e00432f2ff1b47537a11458be13fc82",
      "refreshExpiresIn": 1602250582
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "refreshToken": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      }
    }
  }
}
```

#### Register

`settings`で定義されたフィールドでユーザーを登録します。
ユーザーモデルのEメールフィールドが登録フィールドの一部な場合（デフォルト）、そのEメールまたは2つ目のEメールをもつユーザーが存在しないか確認します。
もし、存在した場合、Eメールフィールドは一意として定義されていませんが（デフォルトのDjangoユーザーモデルのデフォルト）、そのユーザーを登録しません。

ユーザーを作成したとき、そのユーザーに関連する`UserStatus`も作成され、それはユーザーがアーカイブされた、検証された、または2つ目のEメールを持った場合を追跡できるようにします。

アカウントが検証されたことを通知するEメールを送信します。
ユーザーアカウントが検証されていないユーザーのログインを許可している場合、トークンが返されます。

GraphQLのリクエスト例

```graphql
mutation {
  register(
    email:"skywalker@email.com",
    username:"skywalker",
    password1: "qlr4nq3f3",
    password2:"qlr4nq3f3"
  ) {
    success,
    errors,
    token,
    refreshToken
  }
}
```

relayのレスポンス例

```graphql
mutation {
  register(
    input: {
      email:"skywalker@email.com",
      username:"skywalker",
      password1: "qlr4nq3f3",
      password2:"qlr4nq3f3"
    }
  ) {
    success,
    errors,
    token,
    refreshToken
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": true,
      "errors": null,
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImpvZWpvZSIsImV4cCI6MTU4MDE0MjE0MCwib3JpZ0lhdCI6MTU4MDE0MTg0MH0.BGUSGKUUd7IuHnWKy8V6MU3slJ-DHsyAdAjGrGb_9fw",
      "refreshToken": "d9b58dce41cf14549030873e3fab3be864f76ce44"
    }
  }
}
```

ユーザー名の一意制約に違反したときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": false,
      "errors": {
        "username": [
          {
            "message": "A user with that username already exists.",
            "code": "unique"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

パスワードが一致しないときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": false,
      "errors": {
        "password2": [
          {
            "message": "The two password fields didn’t match.",
            "code": "password_mismatch"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

パスワードの検証に失敗したときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": false,
      "errors": {
        "password2": [
          {
            "message": "This password is too short. It must contain at least 8 characters.",
            "code": "password_too_short"
          },
          {
            "message": "This password is too common.",
            "code": "password_too_common"
          },
          {
            "message": "This password is entirely numeric.",
            "code": "password_entirely_numeric"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

Eメールが不正なときの例

```json
{
  "data": {
    "register": {
      "success": false,
      "errors": {
        "email": [
          {
            "message": "Enter a valid email address.",
            "code": "invalid"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

#### ResendActivationEmail

アクティベーションEメールを送信します。
理論上、最初のアクティベーションEメールは、ユーザーが登録されたときに送信されるため、resend（再送信）と呼ばれます。
もし、要求したEメールを持つユーザーが存在しない場合、成功レスポンスが返されます。

GraphQLのリクエスト例

```graphql
mutation {
  resendActivationEmail(
    email:"skywalker@email.com",
  ) {
    success,
    errors

  }
}
```

relayのリクエスト例

```graphql
mutation {
  resendActivationEmail(
    input: {
      email:"skywalker@email.com",
    }
  ) {
    success,
    errors

  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": true,
      "errors": null
    }
  }
}
```

すでに検証されているときのレスポンス例

```json
{
  "data": {
    "resendActivationEmail": {
      "success": false,
      "errors": {
        "email": [
          [
            {
              "message": "Account already verified.",
              "code": "already_verified"
            }
          ]
        ]
      }
    }
  }
}
```

Eメールが不正なときのレスポンス例

```json
{
  "data": {
    "resendActivationEmail": {
      "success": false,
      "errors": {
        "email": [
          {
            "message": "Enter a valid email address.",
            "code": "invalid"
          }
        ]
      }
    }
  }
}
```

Eメールの送信に失敗したときのレスポンス例

```json
{
  "data": {
    "resendActivationEmail": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
            {
              "message": "Failed to send email.",
              "code": "email_fail"
            }
        ]
      }
    }
  }
}
```

#### RevokeToken

`graphql_jwt`の実装と同様に、例外エラーメッセージを"Error decoding signature"から"Invalid token"に変更しています。

GraphQLのリクエスト例

```graphql
mutation {
  revokeToken(
    refreshToken: "a64f732b4e00432f2ff1b47537a11458be13fc82"
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  revokeToken(
    input: {
      refreshToken: "a64f732b4e00432f2ff1b47537a11458be13fc82"
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "revokeToken": {
      "success": true,
      "errors": null
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "revokeToken": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      }
    }
  }
}
```

#### SendPasswordResetEmail

パスワードリセットEメールを送信します。

ユーザーアカウントが検証されていないユーザーに対しては、代わりにアクティベーションEメールを送信します。
1つ目と2つ目のEメールの両方とも受け付けます。
もし、Eメールを要求したユーザーが存在しない場合、成功レスポンスを返します。

GraphQLのリクエスト例

```graphql
mutation {
  sendPasswordResetEmail(
    email: "skywalker@email.com"
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  sendPasswordResetEmail(
    input: {
      email: "skywalker@email.com"
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": true,
      "errors": null
    }
  }
}
```

Eメールが不正なときのレスポンス例

```json
{
  "data": {
    "sendPasswordResetEmail": {
      "success": false,
      "errors": {
        "email": [
          {
            "message": "Enter a valid email address.",
            "code": "invalid"
          }
        ]
      }
    }
  }
}
```

Eメールの送信に失敗したときのレスポンス例

```json
{
  "data": {
    "sendPasswordResetEmail": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Failed to send email.",
            "code": "email_fail"
          }
        ]
      }
    }
  }
}

```

ユーザーの検証に失敗したときのレスポンス例

```json
{
  "data": {
    "sendPasswordResetEmail": {
      "success": false,
      "errors": {
        "email": [
          {
            "message": "Verify your account. A new verification email was sent.",
            "code": "not_verified"
          }
        ]
      }
    }
  }
}
```

#### VerifyAccount

ユーザーアカウントを検証します。
Eメールによって送信されたトークンを受け取ってください。
もしトークンが妥当な場合、`user.status.verified`フィールドを`True`にすることで、検証されたユーザーとしてマークします。

GraphQLのリクエスト例

```graphql
mutation {
  verifyAccount(
    token:"eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImFjdGlvbiI6ImFjdGl2YXRpb24ifQ:1itC5A:vJhRJwBcrNxvmEKxHrZa6Yoqw5Q",
  ) {
    success, errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  verifyAccount(
    input: {
      token:"eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImFjdGlvbiI6ImFjdGl2YXRpb24ifQ:1itC5A:vJhRJwBcrNxvmEKxHrZa6Yoqw5Q",
    }
  ) {
    success, errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "verifyAccount": {
      "success": true,
      "errors": null
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "verifyAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      }
    }
  }
}
```

すでに検証されていたときのレスポンス例

```json
{
  "data": {
    "verifyAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Account already verified.",
            "code": "already_verified"
          }
        ]
      }
    }
  }
}
```

#### VerifySecondaryEmail

ユーザーの2つ目のEメールを検証します。
Eメールによって送信されたトークンを受け取ってください。
このミューテーションを使用するとき、ユーザーはすでに検証されています。

トークンが妥当な場合、2つ目のEメールが`user.status.secondary_email`フィールドに追加されます。

2つ目のEメールが検証されるまで、トークン以外はどこにも保存されないため、2つ目のEメールを新しいアカウントを作成するために使用できることに注意してください。
検証された後、そのEメールは利用できなくなります。

GraphQLのリクエスト例

```graphql
mutation {
  verifySecondaryEmail(
    token: "eyJ1c2VybmFtZSI6Im5ld191c2VyMSIsImFjdGlvbiI6ImFjdGl2YXRpb25fc2Vjb25kYXJ5X2VtYWlsIiwic2Vjb25kYXJ5X2VtYWlsIjoibXlfc2Vjb25kYXJ5X2VtYWlsQGVtYWlsLmNvbSJ9:1ivhfJ:CYZswRKV3avWA8cb41KqZ1-zdVo"
    ) {
    success
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  verifySecondaryEmail(
    input: {
      token: "eyJ1c2VybmFtZSI6Im5ld191c2VyMSIsImFjdGlvbiI6ImFjdGl2YXRpb25fc2Vjb25kYXJ5X2VtYWlsIiwic2Vjb25kYXJ5X2VtYWlsIjoibXlfc2Vjb25kYXJ5X2VtYWlsQGVtYWlsLmNvbSJ9:1ivhfJ:CYZswRKV3avWA8cb41KqZ1-zdVo"
    }
  ) {
    success
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "verifySecondaryEmail": {
      "success": true,
      "errors": null
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "verifySecondaryEmail": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      }
    }
  }
}
```

トークンの有効期限が切れているときのレスポンス例

```json
{
  "data": {
    "verifySecondaryEmail": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Expired token.",
            "code": "expired_token"
          }
        ]
      }
    }
  }
}
```

#### VerifyToken

`graphql_jwt`の実装と同様に、例外エラーメッセージを"Error decoding signature"から"Invalid token"に変更しています。

GraphQLのリクエスト例

```graphql
mutation {
  verifyToken(
    token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImV4cCI6MTU3OTQ1ODY3Miwib3JpZ0lhdCI6MTU3OTQ1ODM3Mn0.rrB4sMA-v7asrr8Z2ru69U1x-d98DuEJVBnG2F1C1S0"
  ) {
    success,
    errors,
    payload
  }
}
```

relayのリクエスト例

```graphql
mutation {
  verifyToken(
    input: {
      token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6InNreXdhbGtlciIsImV4cCI6MTU3OTQ1ODY3Miwib3JpZ0lhdCI6MTU3OTQ1ODM3Mn0.rrB4sMA-v7asrr8Z2ru69U1x-d98DuEJVBnG2F1C1S0"
    }
  ) {
    success
    errors
    payload
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "verifyToken": {
      "success": true,
      "errors": null,
      "payload": {
        "username": "skywalker",
        "exp": 1579458672,
        "origIat": 1579458372
      }
    }
  }
}
```

トークンが不正なときのレスポンス例

```json
{
  "data": {
    "verifyToken": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Invalid token.",
            "code": "invalid_token"
          }
        ]
      },
      "payload": null
    }
  }
}
```

トークンの有効期限が切れているときのレスポンス例

```json
{
  "data": {
    "verifyToken": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Expired token.",
            "code": "expired_token"
          }
        ]
      }
    }
  }
}
```

### 保護されたミューテーション - Protected

保護されたミューテーションは、HTTPの`Authorization`ヘッダーを要求します。
HTTPの`Authorization`ヘッダーなしで、または不正なトークンでリクエストを送信した場合、次が発生します。

- `graphql_jwt.backends.JSONWebTokenBackend`を使用している場合、例外が発生します。
- `graphql_auth.backends.GraphQLAuthBackend`を使用している場合、`success=True`そして`errors`を持つ標準的なレスポンスを返します。

[インストールガイド](https://django-graphene-auth.readthedocs.io/en/latest/installation/)で説明しています。

> 例外を発生させるのではなく、エラーを示すレスポンスを受け取る方がクライアントの実装を単純にできるため、`graphql_auth.backends.GraphQLAuthBackend`を使用する。

#### ArchiveAccount

アカウントをアーカイブして、リフレッシュトークンを無効にします。
ユーザーは、検証され、パスワードを確認されなくてはなりません。

GraphQLのリクエスト例

```graphql
mutation {
  archiveAccount(
    password: "supersecretpassword",
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  archiveAccount(
    input: {
      password: "supersecretpassword",
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "register": {
      "success": true,
      "errors": null
    }
  }
}
```

認証に失敗したときのレスポンス例

```json
{
  "data": {
    "archiveAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Unauthenticated.",
            "code": "unauthenticated"
          }
        ]
      }
    }
  }
}
```

パスワードが不正なときのレスポンス例

```json
{
  "data": {
    "archiveAccount": {
      "success": false,
      "errors": {
        "password": [
          {
            "message": "Invalid password.",
            "code": "invalid_password"
          }
        ]
      }
    }
  }
}
```

ユーザーがユーザーアカウントが検証されていないときのレスポンス例

```json
{
  "data": {
    "archiveAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Please verify your account.",
            "code": "not_verified"
          }
        ]
      }
    }
  }
}
```

#### DeleteAccount

永続的にアカウントを削除するか、`user.is_active=False`にします。
この振る舞いは`settings`で定義されています。
とにかく、ユーザーのリフレッシュトークンは無効になります。
ユーザーは検証され、パスワードを確認されなくてはなりません。

GraphQLのリクエスト例

```graphql
mutation {
  deleteAccount(
    password: "supersecretpassword",
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  deleteAccount(
    input: {
      password: "supersecretpassword",
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "deleteAccount": {
      "success": true,
      "errors": null
    }
  }
}
```

認証に失敗したときのレスポンス例

```json
{
  "data": {
    "deleteAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Unauthenticated.",
            "code": "unauthenticated"
          }
        ]
      }
    }
  }
}
```

パスワードが不正なときのレスポンス例

```json
{
  "data": {
    "deleteAccount": {
      "success": false,
      "errors": {
        "password": [
          {
            "message": "Invalid password.",
            "code": "invalid_password"
          }
        ]
      }
    }
  }
}
```

ユーザーがユーザーアカウントが検証されていないときのレスポンス例

```json
{
  "data": {
    "deleteAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Please verify your account.",
            "code": "not_verified"
          }
        ]
      }
    }
  }
}
```

#### PasswordChange

ユーザーが古いパスワードを知っているときに、アカウントのパスワードを変更します。
新しいトークンとリフレッシュトークンが送信されます。
ユーザーは検証されていなくてはなりません。

GraphQLのリクエスト例

```graphql
mutation {
  passwordChange(
    oldPassword: "supersecretpassword",
    newPassword1: "123456super",
    newPassword2: "123456super"
  ) {
    success,
    errors,
    token,
    refreshToken
  }
}
```

relayのリクエスト例

```graphql
mutation {
  passwordChange(
    input: {
        oldPassword: "supersecretpassword",
        newPassword1: "123456super",
        newPassword2: "123456super"
    }
  ) {
    success,
    errors,
    token,
    refreshToken
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "passwordChange": {
      "success": true,
      "errors": null,
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImpvZWpvZSIsImV4cCI6MTU4MDE0MjE0MCwib3JpZ0lhdCI6MTU4MDE0MTg0MH0.BGUSGKUUd7IuHnWKy8V6MU3slJ-DHsyAdAjGrGb_9fw",
      "refreshToken": "67eb63ba9d279876d3e9ae4d39c311e845e728fc"
    }
  }
}
```

認証に失敗したときのレスポンス例

```json
{
  "data": {
    "passwordChange": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Unauthenticated.",
            "code": "unauthenticated"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

ユーザーアカウントが検証されていないときのレスポンス例

```json
{
  "data": {
    "passwordChange": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Please verify your account.",
            "code": "not_verified"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

パスワードの検証に失敗したときのレスポンス例

```json
{
  "data": {
    "passwordChange": {
      "success": false,
      "errors": {
        "newPassword2": [
          {
            "message": "This password is too short. It must contain at least 8 characters.",
            "code": "password_too_short"
          },
          {
            "message": "This password is too common.",
            "code": "password_too_common"
          },
          {
            "message": "This password is entirely numeric.",
            "code": "password_entirely_numeric"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

パスワードが一致しないときのレスポンス例

```json
{
  "data": {
    "passwordChange": {
      "success": false,
      "errors": {
        "newPassword2": [
          {
            "message": "The two password fields didn’t match.",
            "code": "password_mismatch"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

パスワードが不正なときのレスポンス例

```json
{
  "data": {
    "passwordChange": {
      "success": false,
      "errors": {
        "oldPassword": [
          {
            "message": "Invalid password.",
            "code": "invalid_password"
          }
        ]
      },
      "token": null,
      "refreshToken": null
    }
  }
}
```

#### RemoveSecondaryEmail

ユーザーの2つ目のEメールを削除します。
パスワードの確認を要求します。

GraphQLのリクエスト例

```graphql
mutation {
  removeSecondaryEmail(
    password: "supersecretpassword"
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  removeSecondaryEmail(
    input: {
      password: "supersecretpassword"
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "removeSecondaryEmail": {
      "success": true,
      "errors": null
    }
  }
}
```

パスワードが不正なときのレスポンス例

```json
{
  "data": {
    "removeSecondaryEmail": {
      "success": false,
      "errors": {
        "password": [
          {
            "message": "Invalid password.",
            "code": "invalid_password"
          }
        ]
      }
    }
  }
}
```

#### SendSecondaryEmailActivation

2つ目のEメールにアクティベーションを送信します。
ユーザーは、検証され、パスワードを確認されなくてはなりません。

GraphQLのリクエスト例

```graphql
mutation {
  sendSecondaryEmailActivation(
    email: "my_secondary_email@email.com"
    password: "supersecretpassword",
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  sendSecondaryEmailActivation(
    input: {
      email: "my_secondary_email@email.com"
      password: "supersecretpassword",
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "sendSecondaryEmailActivation": {
      "success": true,
      "errors": null
    }
  }
}
```

認証に失敗したときのレスポンス例

```json
{
  "data": {
    "sendSecondaryEmailActivation": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Unauthenticated.",
            "code": "unauthenticated"
          }
        ]
      }
    }
  }
}
```

Eメールが不正なときのレスポンス例

```json
{
  "data": {
    "sendSecondaryEmailActivation": {
      "success": false,
      "errors": {
        "email": [
          {
            "message": "Enter a valid email address.",
            "code": "invalid"
          }
        ]
      }
    }
  }
}
```

パスワードが不正なときのレスポンス例

```json
{
  "data": {
    "sendSecondaryEmailActivation": {
      "success": false,
      "errors": {
        "password": [
          {
            "message": "Invalid password.",
            "code": "invalid_password"
          }
        ]
      }
    }
  }
}
```

ユーザーがユーザーアカウントが検証されていないときのレスポンス例

```json
{
  "data": {
    "sendSecondaryEmailActivation": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Please verify your account."
            "code": "not_verified"
          }
        ]
      }
    }
  }
}
```

#### SwapEmails

1つ目と2つ目のEメールを交換します。
パスワードの確認を要求します。

GraphQLのリクエスト例

```graphql
mutation {
  swapEmails(
    password: "supersecretpassword"
  ) {
    success,
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  swapEmails(
    input: {
      password: "supersecretpassword"
    }
  ) {
    success,
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "swapEmails": {
      "success": true,
      "errors": null
    }
  }
}
```

パスワードが不正なときのレスポンス例

```json
{
  "data": {
    "swapEmails": {
      "success": false,
      "errors": {
        "password": [
          {
            "message": "Invalid password.",
            "code": "invalid_password"
          }
        ]
      }
    }
  }
}
```

#### UpdateAccount

`settings`で定義されたユーザーモデルのフィールドを更新します。
ユーザーは検証されている必要があります。

GraphQLのリクエスト例

```graphql
mutation {
  updateAccount(
    firstName: "Luke"
  ) {
    success
    errors
  }
}
```

relayのリクエスト例

```graphql
mutation {
  updateAccount(
    input: {
      firstName: "Luke"
    }
  ) {
    success
    errors
  }
}
```

成功したときのレスポンス例

```json
{
  "data": {
    "updateAccount": {
      "success": true,
      "errors": null
    }
  }
}
```

ユーザーが認証されていないときのレスポンス例

```json
{
  "data": {
    "updateAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Unauthenticated.",
            "code": "unauthenticated"
          }
        ]
      }
    }
  }
}
```

ユーザーアカウントが検証されていないときのレスポンス例

```json
{
  "data": {
    "updateAccount": {
      "success": false,
      "errors": {
        "nonFieldErrors": [
          {
            "message": "Please verify your account.",
            "code": "not_verified"
          }
        ]
      }
    }
  }
}
```

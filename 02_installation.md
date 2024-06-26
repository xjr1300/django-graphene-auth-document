# インストール

<https://django-graphene-auth.readthedocs.io/en/latest/installation/>

- [インストール](#インストール)
  - [要求事項](#要求事項)
  - [インストール](#インストール-1)
  - [準備](#準備)
    - [1. スキーマ](#1-スキーマ)
    - [2. 任意のクラスを許可](#2-任意のクラスを許可)
    - [3. 認証バックエンド用にユーザーハンドラを取得](#3-認証バックエンド用にユーザーハンドラを取得)
    - [4. 認証バックエンド - オプション](#4-認証バックエンド---オプション)
    - [5. リフレッシュトークン - オプション](#5-リフレッシュトークン---オプション)
    - [6. Eメールテンプレート](#6-eメールテンプレート)
    - [7. Eメールバックエンド](#7-eメールバックエンド)

## 要求事項

- Python: 3.10 - 3.12
- Django: 3.2 - 4.2 - 5.0

## インストール

```sh
pip install django-graphene-auth
```

> これは、自動的に`graphene`、`graphene-django`、`django-graphql-jwt`、`django-filter`そして`django`をインストールします。

`INSTALLED_APPS`に`graphql_auth`を追加します。

```python
INSTALLED_APPS = [
    # ...
    "graphql_auth",
]
```

マイグレートしてください。

```sh
python manage.py migrate
```

## 準備

次は実行できるようにするために要求される最小限のステップです。

### 1. スキーマ

スキーマに次を追加してください。

```python
# GraphQLの場合
import graphene

from graphql_auth.schema import UserQuery, MeQuery
from graphql_auth import mutations

class AuthMutation(graphene.ObjectType):
    register = mutations.Register.Field()
    verify_account = mutations.VerifyAccount.Field()
    resend_activation_email = mutations.ResendActivationEmail.Field()
    send_password_reset_email = mutations.SendPasswordResetEmail.Field()
    password_reset = mutations.PasswordReset.Field()
    password_set = mutations.PasswordSet.Field() # パスワードなしのユーザー登録
    password_change = mutations.PasswordChange.Field()
    update_account = mutations.UpdateAccount.Field()
    archive_account = mutations.ArchiveAccount.Field()
    delete_account = mutations.DeleteAccount.Field()
    send_secondary_email_activation =  mutations.SendSecondaryEmailActivation.Field()
    verify_secondary_email = mutations.VerifySecondaryEmail.Field()
    swap_emails = mutations.SwapEmails.Field()
    remove_secondary_email = mutations.RemoveSecondaryEmail.Field()

    # django-graphql-jwtの継承
    token_auth = mutations.ObtainJSONWebToken.Field()
    verify_token = mutations.VerifyToken.Field()
    refresh_token = mutations.RefreshToken.Field()
    revoke_token = mutations.RevokeToken.Field()


class Query(UserQuery, MeQuery, graphene.ObjectType):
    pass


class Mutation(AuthMutation, graphene.ObjectType):
    pass


schema = graphene.Schema(query=Query, mutation=Mutation)
```

```python
# Relayの場合
import graphene

from graphql_auth.schema import UserQuery, MeQuery
from graphql_auth import relay

class AuthRelayMutation(graphene.ObjectType):
    register = relay.Register.Field()
    verify_account = relay.VerifyAccount.Field()
    resend_activation_email = relay.ResendActivationEmail.Field()
    send_password_reset_email = relay.SendPasswordResetEmail.Field()
    password_reset = relay.PasswordReset.Field()
    password_set = relay.PasswordSet.Field() # パスワードなしのユーザー登録
    password_change = relay.PasswordChange.Field()
    update_account = relay.UpdateAccount.Field()
    archive_account = relay.ArchiveAccount.Field()
    delete_account = relay.DeleteAccount.Field()
    send_secondary_email_activation =  relay.SendSecondaryEmailActivation.Field()
    verify_secondary_email = relay.VerifySecondaryEmail.Field()
    swap_emails = relay.SwapEmails.Field()
    remove_secondary_email = mutations.RemoveSecondaryEmail.Field()

    # django-graphql-jwtの継承
    token_auth = relay.ObtainJSONWebToken.Field()
    verify_token = relay.VerifyToken.Field()
    refresh_token = relay.RefreshToken.Field()
    revoke_token = relay.RevokeToken.Field()


class Query(UserQuery, MeQuery, graphene.ObjectType):
    pass


class Mutation(AuthRelayMutation, graphene.ObjectType):
    pass


schema = graphene.Schema(query=Query, mutation=Mutation)
```

### 2. 任意のクラスを許可

`GRAPHQL_JWT["JWT_ALLOW_ANY_CLASSES"]`に次を追加します。

```python
# GraphQLの場合
GRAPHQL_JWT = {
    #...
    "JWT_ALLOW_ANY_CLASSES": [
        "graphql_auth.mutations.Register",
        "graphql_auth.mutations.VerifyAccount",
        "graphql_auth.mutations.ResendActivationEmail",
        "graphql_auth.mutations.SendPasswordResetEmail",
        "graphql_auth.mutations.PasswordReset",
        "graphql_auth.mutations.ObtainJSONWebToken",
        "graphql_auth.mutations.VerifyToken",
        "graphql_auth.mutations.RefreshToken",
        "graphql_auth.mutations.RevokeToken",
        "graphql_auth.mutations.VerifySecondaryEmail",
    ],
}
```

```python
# Relayの場合
GRAPHQL_JWT = {
    #...
    "JWT_ALLOW_ANY_CLASSES": [
        "graphql_auth.relay.Register",
        "graphql_auth.relay.VerifyAccount",
        "graphql_auth.relay.ResendActivationEmail",
        "graphql_auth.relay.SendPasswordResetEmail",
        "graphql_auth.relay.PasswordReset",
        "graphql_auth.relay.ObtainJSONWebToken",
        "graphql_auth.relay.VerifyToken",
        "graphql_auth.relay.RefreshToken",
        "graphql_auth.relay.RevokeToken",
        "graphql_auth.relay.VerifySecondaryEmail",
    ],
}
```

### 3. 認証バックエンド用にユーザーハンドラを取得

クエリ内で`select_related("status")`を使用することで、ユーザーの状態をDjangoの`request.user`に結合するために`GRAPHQL_JWT["JWT_GET_USER_BY_NATURAL_KEY_HANDLER"]`設定を追加します。

```python
GRAPHQL_JWT = {
    # ...
    "JWT_GET_USER_BY_NATURAL_KEY_HANDLER": "graphql_auth.utils.get_user_by_natural_key",
    # ...
}
```

### 4. 認証バックエンド - オプション

`AUTHENTICATION_BACKENDS`に次を追加してください。

```python
AUTHENTICATION_BACKENDS = [
    # 次を削除
    # "graphql_jwt.backends.JSONWebTokenBackend",

    # 次を追加
    "graphql_auth.backends.GraphQLAuthBackend",

    # ...
]
```

> **🖊 graphql_jwt.backendとの違いは何でしょうか？
> たった1つしか違わない同じバックエンドを実装しました。
>
> - `JWT_ALLOW_ANY_CLASSES`にないクラスに良くないトークンでリクエストを送信した場合に例外を起こしません。
>
> なぜこの振る舞いにしたかったのでしょうか？
>
> 実際のエラーが発生する代わりに、それを処理して意味のあるものを返すことができます。例えば・・・
>
> ```python
> cls(success=False, errors="Unauthenticated.")
> ```
>
> この状況に対処するために、次のいずれかを行います。
>
> - 単純に、`graphql_jwt`の[@login_required](https://django-graphql-jwt.domake.io/en/latest/decorators.html#login-required)デコレーターを使用する。
> - [このパッケージのlogin_requiredデコレーターを使用する](https://github.com/ptbang/django-graphene-auth/blob/face76be02928947e32358c5207b397d2457f99b/graphql_auth/decorators.py#L7)。ただし、これは、出力に"success"と"errors"フィールドが含まれていることを期待していることに注意してください。
> - 独自の`login_required`デコレーターを作成する。

### 5. リフレッシュトークン - オプション

リフレッシュトークンはオプションで、このパッケージは[Django GraphQL JWT](https://github.com/flavors/django-graphql-jwt)のデフォルトのトークンと機能します。

[公式ドキュメント](https://django-graphql-jwt.domake.io/en/latest/refresh_token.html#long-running-refresh-tokens)に従うか、単純に`settings.py`についを追加してください。

```python
INSTALLED_APPS = [
    # ...
    'graphql_jwt.refresh_token.apps.RefreshTokenConfig'
]

GRAPHQL_JWT = {
    # ...
    'JWT_VERIFY_EXPIRATION': True,
    'JWT_LONG_RUNNING_REFRESH_TOKEN': True,
}
```

そして、マイグレートすることを覚えておいてください。

```sh
python manage.py migrate
```

### 6. Eメールテンプレート

> Eメールテンプレートのオーバーライドは[ここ](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)で説明しています。

このパッケージは、あるデフォルトのEメールテンプレートを同梱しています。
それを使う予定であれば、テンプレートの設定に次があるか確認してくださいｓ．

```python
TEMPLATES = [
    {
        # ...
        "APP_DIR": True,
    },
]
```

### 7. Eメールバックエンド

デフォルトの設定は、アクティベーションEメールを送信するため、[設定](https://django-graphene-auth.readthedocs.io/en/latest/settings/)でそれを`False`に設定できますが、パスワードをリセットするためにEメールバックエンドがいまだ必要です。

開発で最も早く準備する方法は、[コンソールEメールバックエンド](https://docs.djangoproject.com/en/5.0/topics/email/#console-backend)を準備することで、`settings.py`に次を単純に追加します。

```python
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

これで、すべてのEメールは、実際のEメールではなく、標準出力に送信されます。

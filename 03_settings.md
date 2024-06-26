# 設定

<https://django-graphene-auth.readthedocs.io/en/latest/settings/>

- [設定](#設定)
  - [例](#例)
  - [論理フラグ](#論理フラグ)
    - [ALLOW\_LOGIN\_NOT\_VERIFIED](#allow_login_not_verified)
    - [ALLOW\_LOGIN\_WITH\_SECONDARY\_EMAIL](#allow_login_with_secondary_email)
    - [ALLOW\_PASSWORDLESS\_REGISTRATION](#allow_passwordless_registration)
    - [ALLOW\_DELETE\_ACCOUNT](#allow_delete_account)
    - [SEND\_ACTIVATION\_EMAIL](#send_activation_email)
    - [SEND\_PASSWORD\_SET\_EMAIL](#send_password_set_email)
  - [動的フィールド](#動的フィールド)
    - [基本](#基本)
    - [LOGIN\_ALLOWED\_FIELDS](#login_allowed_fields)
    - [REGISTER\_MUTATION\_FIELDS](#register_mutation_fields)
    - [REGISTER\_MUTATION\_FIELDS\_OPTIONAL](#register_mutation_fields_optional)
    - [UPDATE\_MUTATION\_FIELDS](#update_mutation_fields)
    - [CUSTOM\_ERROR\_TYPE](#custom_error_type)
  - [クエリ](#クエリ)
    - [USER\_NODE\_FILTER\_FIELDS](#user_node_filter_fields)
    - [USER\_NODE\_EXCLUDE\_FIELDS](#user_node_exclude_fields)
  - [トークンの有効期限](#トークンの有効期限)
    - [EXPIRATION\_ACTIVATION\_TOKEN](#expiration_activation_token)
    - [EXPIRATION\_PASSWORD\_RESET\_TOKEN](#expiration_password_reset_token)
    - [EXPIRATION\_SECONDARY\_EMAIL\_ACTIVATION\_TOKEN](#expiration_secondary_email_activation_token)
    - [EXPIRATION\_PASSWORD\_SET\_TOKEN](#expiration_password_set_token)
  - [Eメール](#eメール)
    - [EMAIL\_FROM](#email_from)
    - [ACTIVATION\_PATH\_ON\_EMAIL](#activation_path_on_email)
    - [PASSWORD\_RESET\_PATH\_ON\_EMAIL](#password_reset_path_on_email)
    - [PASSWORD\_SET\_PATH\_ON\_EMAIL](#password_set_path_on_email)
    - [ACTIVATION\_SECONDARY\_EMAIL\_PATH\_ON\_EMAIL](#activation_secondary_email_path_on_email)
    - [EMAIL\_ASYNC\_TASK](#email_async_task)
  - [Eメール件名テンプレート](#eメール件名テンプレート)
    - [EMAIL\_SUBJECT\_ACTIVATION](#email_subject_activation)
    - [EMAIL\_SUBJECT\_ACTIVATION\_RESEND](#email_subject_activation_resend)
    - [EMAIL\_SUBJECT\_SECONDARY\_EMAIL\_ACTIVATION](#email_subject_secondary_email_activation)
    - [EMAIL\_SUBJECT\_PASSWORD\_RESET](#email_subject_password_reset)
    - [EMAIL\_SUBJECT\_PASSWORD\_SET](#email_subject_password_set)
  - [Eメールテンプレート](#eメールテンプレート)
    - [EMAIL\_TEMPLATE\_ACTIVATION](#email_template_activation)
    - [EMAIL\_TEMPLATE\_ACTIVATION\_RESEND](#email_template_activation_resend)
    - [EMAIL\_TEMPLATE\_SECONDARY\_EMAIL\_ACTIVATION](#email_template_secondary_email_activation)
    - [EMAIL\_TEMPLATE\_PASSWORD\_RESET](#email_template_password_reset)
    - [EMAIL\_TEMPLATE\_PASSWORD\_SET](#email_template_password_set)
    - [EMAIL\_TEMPLATE\_VARIABLES](#email_template_variables)

## 例

構成は、`GRAPHQL_AUTH`という名前の単独のDjango設定で行われます。

```python
# settings.py
GRAPHQL_JWT = {
    # JWTトークンからユーザーインスタンスの取得を試みるとき、ユーザーとユーザー状態を結合します。
    "JWT_GET_USER_BY_NATURAL_KEY_HANDLER": "graphql_auth.utils.get_user_by_natural_key",
}

GRAPHQL_AUTH = {
    "LOGIN_ALLOWED_FIELDS": ["email", "username"],
}
```

---

## 論理フラグ

### ALLOW_LOGIN_NOT_VERIFIED

検証されることなしで、ユーザーがログインできるかを決定します。
もし`True`の場合、登録は出力で`token`と`refresh token`を返します。
デフォルトは`True`です。

> ユーザーの検証に成功した後にログインできるようにするために`False`にする。

### ALLOW_LOGIN_WITH_SECONDARY_EMAIL

もしユーザーが2つ目のEメールアドレスを設定している場合、そのユーザーはログインにそれを使用できます。
デフォルトは`True`です。

### ALLOW_PASSWORDLESS_REGISTRATION

パスワード無しでユーザーを登録できるようにします。
Djangoの`set_unusable_password()`がデフォルトのパスワード設定に使用されます。

- ユーザーは、ユーザーのパスワードを設定するまで、ログインできません。

デフォルトは`False`です。

### ALLOW_DELETE_ACCOUNT

アカウントを削除する代わりに、`user.is_active`を`False`にします。
もし、`True`を設定した場合、実際にアカウントが削除されます。
デフォルトは`False`です。

### SEND_ACTIVATION_EMAIL

`False`に設定した場合、Eメールは送信されません。
ユーザーは、`verified=False`状態のままになることに注意してください。
デフォルトは`True`です。

### SEND_PASSWORD_SET_EMAIL

`True`に設定した場合、`ALLOW_PASSWORDLESS_REGISTRATION`に依存して、登録後ユーザーはユーザーのパスワードが設定されたことを通知されます。

---

## 動的フィールド

選択できるフィールドです。

### 基本

すべてのフィールドは、ユーザーモデルフィールドの名前と一致する必要があります。

- 文字列リストフィールドまたは辞書マッピングフィールドと[grapheneの基本的なスカラー](https://docs.graphene-python.org/en/latest/types/scalars/#base-scalars)です。

次は例です。

```python
update_fields_list = ["first_name", "last_name"]

# 次と同じです。

update_field_dict = {
    "first_name": "String",
    "last_name": "String",
}

# 登録したい任意のIntがあるかもしれません。

REGISTER_MUTATION_FIELDS = {
  "email": "String",
  "username": "String",
  "luck_number": "Int",
}
```

### LOGIN_ALLOWED_FIELDS

デフォルトは、`["email", "username"]`です。

### REGISTER_MUTATION_FIELDS

登録で必須となるフィールドで、`password1`と`password2`と一緒です。
デフォルトは、`["email", "username"]`です。

### REGISTER_MUTATION_FIELDS_OPTIONAL

登録でオプションとなるフィールドです。
デフォルトは、`[]`です。

### UPDATE_MUTATION_FIELDS

アカウンの更新でオプションとなるフィールドです。
デフォルトは、`["first_name", "last_name"]`です。

### CUSTOM_ERROR_TYPE

Graphene型を提供することによって、カスタマイズされたミューテーションエラーを出力します。
デフォルトは、`graphql_auth.types.ExpectedErrorType`です。
次は例です。

```python
class CustomErrorType(graphene.Scalar):
    @staticmethod
    def serialize(errors):
        return {"my_custom_error_format"}
```

---

## クエリ

### USER_NODE_FILTER_FIELDS

詳細は[graphene django](https://docs.graphene-python.org/projects/django/en/latest/filtering/)と[django filter](https://django-filter.readthedocs.io/en/master/guide/usage.html#the-filter)で学んでください。

デフォルトは次です。

```python
{
    "email": ["exact",],
    "username": ["exact", "icontains", "istartswith"],
    "is_active": ["exact"],
    "status__archived": ["exact"],
    "status__verified": ["exact"],
    "status__secondary_email": ["exact"],
}
```

### USER_NODE_EXCLUDE_FIELDS

デフォルトは、`["password", "is_superuser"]`です。

---

## トークンの有効期限

### EXPIRATION_ACTIVATION_TOKEN

アクティベーショントークンの有効期限です。
デフォルトは、`timedelta(days=7)`です。

### EXPIRATION_PASSWORD_RESET_TOKEN

パスワードリセットトークンの有効期限です。
デフォルトは、`timedelta(hours=1)`です。

### EXPIRATION_SECONDARY_EMAIL_ACTIVATION_TOKEN

2つ目のEメールアクティベーショントークンの有効期限です。
デフォルトは、`timedelta(hours=1)`です。

### EXPIRATION_PASSWORD_SET_TOKEN

パスワード設定トークンの有効期限です。
デフォルトは、`timedelta(days=7)`です。

---

## Eメール

### EMAIL_FROM

設定からデフォルト値を得ますが、特定のEメールを提供できます。
デフォルトは、`getattr(django_settings, "DEFAULT_FROM_EMAIL", "test@email.com")`です。

### ACTIVATION_PATH_ON_EMAIL

アクティベーションEメールで使用されるパス[変数](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)です。
デフォルトは、`"activate"`です。

### PASSWORD_RESET_PATH_ON_EMAIL

パスワードリセットEメールで使用されるパス[変数](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)です。
デフォルトは、`"password-reset"`です。

### PASSWORD_SET_PATH_ON_EMAIL

パスワード設定Eメールで使用されるパス[変数](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)です。
デフォルトは、`"password-set"`です。

### ACTIVATION_SECONDARY_EMAIL_PATH_ON_EMAIL

2つ目のEメールアクティベーションEメールで使用されるパス[変数](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)です。
デフォルトは、`"activate"`です。

### EMAIL_ASYNC_TASK

すべてのEメールを送信する関数のラッパー関数への文字列のパスです。
この関数は、Eメールを送信する関数と引数のタプルからなる、2つの引数を受け付けなければ成りません。

これは擬似的な非同期サポートで、非同期コードを実装できるようにするための単なるフックであることに注意してください。

celeryを使用した基本的な使用方法を次に示します。

```python
# path/to/async_task.py
from celery import Celery

app = Celery("some_name", broker="pyamqp://guest@localhost//")

@app.task
def async_email(func, args):
    """
    graphql_authパッケージ用のEメール送信タスク
    """
    return func(*args)

def graphql_auth_async_email(args):
    async_email.delay(args)
```

上記の例のために、次の設定をが必要です。

```python
GRAPHQL_AUTH = {
    "EMAIL_ASYNC_TASK": "path.to.async_task.graphql_auth_async_email",
}
```

デフォルトは、`False`です。

---

## Eメール件名テンプレート

[ここ](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)に示されたEメールテンプレートを上書きできますが、テンプレートの名前も変更できます。

### EMAIL_SUBJECT_ACTIVATION

デフォルトは、`"email/activation_subject.txt"`です。

### EMAIL_SUBJECT_ACTIVATION_RESEND

デフォルトは、`"email/activation_subject.txt"`です。

### EMAIL_SUBJECT_SECONDARY_EMAIL_ACTIVATION

デフォルトは、`"email/activation_subject.txt"`です。

### EMAIL_SUBJECT_PASSWORD_RESET

デフォルトは、`"email/email/password_reset_subject.txt"`です。

### EMAIL_SUBJECT_PASSWORD_SET

デフォルトは、`"email/email/password_set_subject.txt"`です。

---

## Eメールテンプレート

[ここ](https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/)に示されたEメールテンプレートをオーバーライドできますが、テンプレート名も変更できます。

### EMAIL_TEMPLATE_ACTIVATION

デフォルトは、`"email/activation_email.html"`です。

### EMAIL_TEMPLATE_ACTIVATION_RESEND

デフォルトは、`"email/activation_email.html"`です。

### EMAIL_TEMPLATE_SECONDARY_EMAIL_ACTIVATION

デフォルトは、`"email/activation_email.html"`です。

### EMAIL_TEMPLATE_PASSWORD_RESET

デフォルトは、`"email/password_reset_email.html"`です。

### EMAIL_TEMPLATE_PASSWORD_SET

デフォルトは、`"email/password_set_email.html"`です。

### EMAIL_TEMPLATE_VARIABLES

デフォルトは、`{}`です。
テンプレートに注入されるテンプレート変数のキーと値のペアの辞書です。
例は次のとおりです。

```python
GRAPHQL_AUTH = {
    "EMAIL_TEMPLATE_VARIABLES": {
        "frontend_domain": "the-frontend.com",
    }
}
```

これで、テンプレートに次の通り注入できます。

```text
<p>{{ protocol }}://{{ frontend_domain }}/{{ path }}/{{ token}}</p>
```

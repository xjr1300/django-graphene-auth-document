# テスト

<https://django-graphene-auth.readthedocs.io/en/latest/tests/>

- [テスト](#テスト)

> **🖊 バージョン1.1.0の新機能です。

テスト用に`CommonTestCase`クラスを利用でき、それはオリジナルの`GraphQLTestCase`を実装して、GraphQLとRelayのクエリをテストするための追加機能を含んでいます。
この方法のおかげて、一度テストを記述したら、GraphQLとRelayのリクエスト両方でそれを使用することができます。

例えば、`CommonTestCase`クラスを実装した`LoginCommonTestCase`を作成して、このクラスに`_test_login`という名前のメソッドを追加します。

次に、下のようにGraphQLクエリのための`LoginTestCase`クラスと、Relayクエリのための`LoginRelayTestCase`クラスを作成します。

```python
# test_login.py
from copy import copy

from django.conf import settings
from graphql_auth.common_testcase import CommonTestCase


class LoginCommonTestCase(CommonTestCase):
    def _test_login(self):
        # query = self.get_query("username", self.verified_user.username) # type: ignore
        query = self.get_query(
            "username",
            self.verified_user.username,
            "password",
        ) # type: ignore
        with self.assertNumQueries(3):
            response = self.query(query)
        result = self.get_response_result(response)
        self.assertResponseNoErrors(response)
        self.assertTrue(result['success'])
        self.assertIsNone(result['errors'])
        self.assertTrue(result["token"])
        self.assertTrue(result["payload"])
        self.assertTrue(result["refreshToken"])
        self.assertTrue(result["refreshExpiresIn"])
        self.assertFalse(result['unarchiving'])


# GraphQLクエリ用
class LoginTestCase(LoginCommonTestCase):
    RESPONSE_RESULT_KEY = "tokenAuth"

    def get_query(self, field, username, password):
        return """
            mutation {
              tokenAuth(%s: "%s", password: %s") {
                success
                errors
                token
                payload
                refreshToken
                refreshExpiresIn
                unarchiving
                user {
                  username
                  id
                }
              }
            }""" % (
              field, username, password,
            )


# Relayクエリ用
class LoginRelayTestCase(LoginCommonTestCase):
    RESPONSE_RESULT_KEY = "relayTokenAuth"

    def get_query(self, field, username, password):
        return """
            mutation {
                relayTokenAuth(
                  input: { %s: "%s", password: "%s" }
                ) {
                  success
                  errors
                  token
                  payload
                  refreshToken
                  refreshExpiresIn
                  unarchiving
                  user {
                    username
                    id
                  }
                }""" % (
                  field, username, password,
                )
```

> `RESPONSE_RESULT_KEY`は、`CommonTestCase`クラスの`get_response_result`、`get_response_errors`メソッドで、レスポンスを辞書、または辞書のリストに変換する際に使用されている。
> <https://github.com/ptbang/django-graphene-auth/blob/master/graphql_auth/common_testcase.py#L64>

これらのログインクラスは両方とも、`CommonTestCase`クラスのメタクラス定義のおかげで、自動的に接頭辞の`_`のない`test_login`メソッドを含むようになります。

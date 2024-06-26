# Django Graphene Auth

<https://django-graphene-auth.readthedocs.io/en/latest/>

- [Django Graphene Auth](#django-graphene-auth)
  - [パッケージについて](#パッケージについて)
  - [機能](#機能)
  - [例](#例)

GraphQLを使用した最も最新のバージョンのDjango、Django Graphene、Django GraphQL JWTをサポートした[Django](https://github.com/django/django)のユーザー登録と認証パッケージです。

## パッケージについて

このプロジェクトは、*Pedro Bern*によって作成された[Django GraphQL Auth](https://github.com/PedroBern/django-graphql-auth)からフォークしたリポジトリを基礎としています。

このプロジェクトを作成すること決定した理由は、オリジナルが新しいバージョンのdjango、graphene-djangoそしてdjango-graphql-jwtをサポートしていないからです。
さらに、オリジナルは近い将来にされない開発されないようだからです。

## 機能

- ✅ 素晴らしいドキュメント
- ✅ [Relay](https://github.com/facebook/relay%3E)と完全互換
- ✅ デフォルトまたはカスタムユーザーモデルと機能
- ✅ [Django GraphQL JWT](https://github.com/flavors/django-graphql-jwt)を使用したJWT認証
- ✅ [Django Filter](https://github.com/carltongibson/django-filter)と[Graphene Django](https://github.com/graphql-python/graphene-django)を使用したクエリのフィルター
- ✅ Eメール検証を使用したユーザー登録
- ✅ Eメール検証も使用した2つ目のEメールの追加
- ✅ アクティベーションEメールの再送
- ✅ ユーザーの取得と更新
- ✅ ユーザーのアーカイブ
- ✅ ユーザーの永続的な削除、またはユーザーの非アクティブ化
- ✅ アーカイブしたユーザーをログイン時に再度アクティブ化
- ✅ ユーザー状態の追跡（アーカイブ、検証済み、2つ目のEメール）
- ✅ パスワードの変更
- ✅ Eメールを介したパスワードのリセット
- ✅ アカウントをアーカイブ、削除、パスワード変更、リセットしたときに、ユーザートークンを破棄
- ✅ すべてのミューテーションが`success`と`errors`を返却
- ✅ カスタマイズ可能なデフォルトのEメールテンプレート
- ✅ ロックインなしのカスタマイズ性
- ✅ パスワードなしのユーザー登録

## 例

ユーザーアカウントの処理が、とても簡単になります。

```graphql
mutation {
  register(
    email: "new_user@email.com",
    username: "new_user",
    password1: "123456super",
    password2: "123456super",
  ) {
    success
    errors
    token
    refreshToken
  }
}
```

新しいユーザーのステータスを確認します。

```python
u = UserModel.objects.last()
u.status.verified
# False
```

ユーザー登録の間、検証リンクが載ったEメールが送信されます。

```graphql
mutation {
  verifyAccount(
    token: "<TOKEN ON EMAIL LINK>",
  ) {
    success
    errors
  }
}
```

現在、ユーザーが検証されました。

```python
u.status.verified
# True
```

[インストールガイド](https://django-graphene-auth.readthedocs.io/en/latest/installation/)を確認してください。
または、もしよければ、[api](https://django-graphene-auth.readthedocs.io/en/latest/api/)を確認してください。

# Eメールテンプレートの上書き

<https://django-graphene-auth.readthedocs.io/en/latest/overriding-email-templates/>

- [Eメールテンプレートの上書き](#eメールテンプレートの上書き)
  - [settingsの更新](#settingsの更新)
  - [ファイルとフォルダ構造](#ファイルとフォルダ構造)
  - [Eメール変数](#eメール変数)
  - [テンプレートの記述](#テンプレートの記述)

デフォルトのEメールテンプレートは単なる例で、おそらくそれをカスタマイズしたいでしょう。

## settingsの更新

```python
# settings.py
TEMPLATES = [
    {
        # ...
        "DIRS": [os.path.join(BASE_DIR, "templates")],
        # ...
    }
]
```

## ファイルとフォルダ構造

次のフォルダとファイルの構造を作成してください。

```text
project_name/
├── project_name/
├── templates/
│    └── email/
│         ├── activation_email.html
│         ├── activation_subject.txt
│         ├── password_reset_email.html
│         └── password_reset_subject.txt
├── db.sqlite3
└── manage.py
```

これは最小限です。
[Eメールテンプレート設定](https://django-graphene-auth.readthedocs.io/en/latest/settings/)を確認して、次のカスタムテンプレートを作成できます。

- アカウントのアクティベーション
- アクティベーションEメールの再送信
- パスワードリセットEメール
- 2つ目のEメールのアクティベーション

## Eメール変数

件名とEメールテンプレートの両方は、次の変数を受け取ります。

- user
- token -> アカウントのアクティベーション、パスワードのリセット、2つ目のメールアドレスのアクティベーション
- port
- site_name -> [Django Sites Framework](https://docs.djangoproject.com/en/5.0/ref/contrib/sites/)（オプション）
- domain -> [Django Sites Framework](https://docs.djangoproject.com/en/5.0/ref/contrib/sites/)（オプション）
- protocol
- path -> [settings](https://django-graphene-auth.readthedocs.io/en/latest/settings/)で定義（何らかのフロントエンドのパス）
- request
- timestamp
- `EMAIL_TEMPLATE_VARIABLES`設定を使用して定義されたカスタム変数 -> [settings](https://django-graphene-auth.readthedocs.io/en/latest/settings/)で定義

## テンプレートの記述

次のようにテンプレートを記述できます。

```html
<!-- activation_email.html -->

<h3>{{ site_name }}</h3>

<p>こんにちは、{{ user.username }} さん!</p>

<p>次のURLをWebブラウザで表示して、あなたのアカウントを有効にしてください。

<p>{{ protocol }}://{{ domain }}/{{ path }}/{{ token }}</p>
```

# rails tutorialまとめ

## cloud9でrails server

```bash
rails server -b $IP -p $PORT
```

## rails gで作成したものを元に戻す

```bash
rails destroy controller hoges
```

## お作法

### sharedディレクトリ
複数のviewで共通して使うパーシャルは
```appp/views/shared```におく


## rails test拡張

REDやGREENに色を付ける

```ruby
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'
+ require "minitest/reporters"
+ Minitest::Reporters.use!

class ActiveSupport::TestCase
  # Setup all fixtures in test/fixtures/*.yml for all tests
  # in alphabetical order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end

```

## testでのinitialize
テストで初期化時に設定する処理

```ruby
def setup

end
```

## rails test手順

https://railstutorial.jp/chapters/static_pages?version=5.0#sec-sample_app_setup

### テストパターンを追加する
```test/controllers/hoge_controller_test.rb```に追記

```ruby
  test "should get about" do
    get static_pages_about_url
    assert_response :success
  end

```

### ルーティングを追加
```config/routes.rb```に追記する

```ruby
 get  'static_pages/about'
```

### コントローラにアクションを追加する
```app/controllers/hoge.rb```にアクションを追加する

```ruby
def about
end
```

### テンプレートファイルを追加する
```app/views/hoge/hoge.html.erb```を追加する

**一旦これでGREEN**

## タイトルテスト手順

### テストパターンを追加する
```test/controllers/hoge_controller_test.rb```に追記する

```ruby
 test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Home | Ruby on Rails Tutorial Sample App"
  end
```


## ROOT URLヘルパー
テストなどでroot urlのテストなどしたい場合は
```root_url```を使う


## テスト自動化

Guarによるテスト自動化


### guardインストール

gemfileに以下を追記

```Gemfile
group :test do
  gem 'rails-controller-testing', '1.0.2'
  gem 'minitest-reporters',       '1.1.14'
  gem 'guard',                    '2.13.0'
  gem 'guard-minitest',           '2.4.4'
end
```

```bash
bundle exec guard init
```

### Guardfileカスタマイズ

Guardfileを以下に置き換える

```ruby
# Guardのマッチング規則を定義
guard :minitest, spring: "bin/rails test", all_on_start: false do
  watch(%r{^test/(.*)/?(.*)_test\.rb$})
  watch('test/test_helper.rb') { 'test' }
  watch('config/routes.rb')    { integration_tests }
  watch(%r{^app/models/(.*?)\.rb$}) do |matches|
    "test/models/#{matches[1]}_test.rb"
  end
  watch(%r{^app/controllers/(.*?)_controller\.rb$}) do |matches|
    resource_tests(matches[1])
  end
  watch(%r{^app/views/([^/]*?)/.*\.html\.erb$}) do |matches|
    ["test/controllers/#{matches[1]}_controller_test.rb"] +
    integration_tests(matches[1])
  end
  watch(%r{^app/helpers/(.*?)_helper\.rb$}) do |matches|
    integration_tests(matches[1])
  end
  watch('app/views/layouts/application.html.erb') do
    'test/integration/site_layout_test.rb'
  end
  watch('app/helpers/sessions_helper.rb') do
    integration_tests << 'test/helpers/sessions_helper_test.rb'
  end
  watch('app/controllers/sessions_controller.rb') do
    ['test/controllers/sessions_controller_test.rb',
     'test/integration/users_login_test.rb']
  end
  watch('app/controllers/account_activations_controller.rb') do
    'test/integration/users_signup_test.rb'
  end
  watch(%r{app/views/users/*}) do
    resource_tests('users') +
    ['test/integration/microposts_interface_test.rb']
  end
end

# 与えられたリソースに対応する統合テストを返す
def integration_tests(resource = :all)
  if resource == :all
    Dir["test/integration/*"]  else
    Dir["test/integration/#{resource}_*.rb"]
  end
end

# 与えられたリソースに対応するコントローラのテストを返す
def controller_test(resource)
  "test/controllers/#{resource}_controller_test.rb"
end

# 与えられたリソースに対応するすべてのテストを返す
def resource_tests(resource)
  integration_tests(resource) << controller_test(resource)
end

```

### gitignoreファイル追記

```gitignore
# Ignore Spring files.
/spring/*.pid
```

### guard実行

```bash
bundle exec guard
```


## application全体のhelper
```app/helpers/application_helper.rb```

## 配列あれこれ

```ruby
0..9
(0..9).to_a # 配列に変換
 
%w[foo bar baz quux]       # %wを使って文字列の配列に変換
```

## パーシャル
HTMLをパーツ単位で分割する

```app/views/layouts/_shim.html.erb```にファイルを作成し
以下で読み込ます

```html
<%= render 'layouts/shim' %>
```


## 名前付きルーティング

```ruby
Rails.application.routes.draw do
  root 'static_pages#home'
  get  '/help',    to: 'static_pages#help'
  get  '/about',   to: 'static_pages#about'
  get  '/contact', to: 'static_pages#contact'
end
```

## 名前付きルーティング時のテスト

getのpathを変更する

```ruby
require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest

  test "should get home" do
    get root_path
    assert_response :success
    assert_select "title", "Ruby on Rails Tutorial Sample App"
  end

  test "should get help" do
    get help_path
    assert_response :success
    assert_select "title", "Help | Ruby on Rails Tutorial Sample App"
  end

  test "should get about" do
    get about_path
    assert_response :success
    assert_select "title", "About | Ruby on Rails Tutorial Sample App"
  end

  test "should get contact" do
    get contact_path
    assert_response :success
    assert_select "title", "Contact | Ruby on Rails Tutorial Sample App"
  end
end
```


## 結合テスト

### テストテンプレート作成

```bash
rails generate integration_test site_layout

```

### レイアウトのリンクに対するテスト

```test/integration/site_layout_test.rb```

```ruby
require 'test_helper'

class SiteLayoutTest < ActionDispatch::IntegrationTest

  test "layout links" do
    get root_path
    assert_template 'static_pages/home' #適切なテンプレートが使われているか。非推奨？！
    assert_select "a[href=?]", root_path, count: 2
    assert_select "a[href=?]", help_path
    assert_select "a[href=?]", about_path
    assert_select "a[href=?]", contact_path
  end
end
```


## rails console sandboxモード
終了時にデータベースの変更などはロールバックされる

```bash
rails console --sandbox
```

## ORM関連

### update_attributes

属性のハッシュを受け取り、成功時には更新と保存を続けて同時に行います (保存に成功した場合はtrueを返します)。ただし、検証に1つでも失敗すると、 update_attributesの呼び出しは失敗します

```ruby
 user.update_attributes(name: "The Dude", email: "dude@abides.org")

```


### update_attribute
特定の属性のみを更新

```ruby
user.update_attribute(:name, "El Duderino")
```


## Model バリデーションテスト

### テストを定義する

```test/models/user_test.rb```

```ruby
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup #インスタンス変数を作る
    @user = User.new(name: "Example User", email: "user@example.com")
  end

  test "should be valid" do
    assert @user.valid?
  end
end

```

### 存在性を検証
値が存在することをテストする

```test/models/user_test.rb```に追記

```ruby
  test "name should be present" do
    @user.name = "     "
    assert_not @user.valid?
  end
```

```app/models/user.rb```

validates メソッドにpresence: trueを与えて必須チェック

```ruby
class User < ApplicationRecord
  validates :name, presence: true
end
```

### エラー時の検証

**errors**オブジェクトを使えばどの検証が失敗したかわかる

```ruby
>> user.errors.full_messages
=> ["Name can't be blank"]
```

### 長さを検証

```test/models/user_test.rb```に追記

```ruby
  test "name should not be too long" do
    @user.name = "a" * 51
    assert_not @user.valid?
  end

  test "email should not be too long" do
    @user.email = "a" * 244 + "@example.com"
    assert_not @user.valid?
  end
```

```app/models/user.rb```を修正

validates メソッドに長さの検証を追加する

```ruby
class User < ApplicationRecord
   validates :name,  presence: true, length: { maximum: 50 }
  validates :email, presence: true, length: { maximum: 255 }end
```

### メールフォーマット検査

#### 有効なメールフォーマットをテスト
```test/models/user_test.rb```に追記

```ruby
  test "email validation should accept valid addresses" do
    valid_addresses = %w[user@example.com USER@foo.COM A_US-ER@foo.bar.org
                         first.last@foo.jp alice+bob@baz.cn]
    valid_addresses.each do |valid_address|
      @user.email = valid_address
      assert @user.valid?, "#{valid_address.inspect} should be valid"
    end
  end
```

assertメソッドの第2引数にエラーメッセージを追加していることに注目してください。これによって、どのメールアドレスでテストが失敗したのかを特定できるようになります。


#### 無効なメールフォーマットをテスト

```test/models/user_test.rb```に追記

```ruby
   test "email validation should reject invalid addresses" do
    invalid_addresses = %w[user@example,com user_at_foo.org user.name@example.
                           foo@bar_baz.com foo@bar+baz.com]
    invalid_addresses.each do |invalid_address|
      @user.email = invalid_address
      assert_not @user.valid?, "#{invalid_address.inspect} should be invalid"
    end
  end
```

メールフォーマットを正規表現で検証する

```app/models/user.rb```に追記

```ruby
 VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX }
```

**format**オプションは引数に正規表現を取る

### 一意性検証

```test/models/user_test.rb```に追記

```ruby
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    duplicate_user.email = @user.email.upcase
    @user.save
    assert_not duplicate_user.valid?
  end
```
dupは、同じ属性を持つデータを複製するためのメソッド

大文字小文字は区別しないので **upcase** で大文字に変換する

```app/models/user.rb```に追記

```ruby
   format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
                    
```
case_sensitiveは大文字小文字を区別するか
falseは区別しない

#### カラムに一意性追加
現状ではソースコードレベルでの一意性検査しかできていないため、DB側にも一位性制約を追加する

```bash
rails g migration add_index_to_users_email
```
マイグレーションファイルを作成

上記ファイルに追記する

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration[5.0]
  def change
+    add_index :users, :email, unique: true
  end
end
```

#### メールアドレスを小文字に変換
DBによっては常に大文字小文字を区別するインデックス を使っているとは限らないので、メールアドレスを保存前に小文字に変換する


```app/models/user.rb```に追記

```ruby
   before_save { self.email = email.downcase }                    
```
Userモデルの中では右式のselfを省略できるので、今回は次のように書きました (ちなみにこのselfは現在のユーザーを指します)。


## セキュアなパスワード

セキュアなパスワードの実装は、 **has_secure_password** というRailsのメソッドを呼び出すだけでほとんど終わってしまいます

 **has_secure_password** を使うにはモデル内にpassword_digestという属性が必要

### パスワード保存用のカラムを追加

```bash
rails generate migration add_password_digest_to_users password_digest:string

```

### ハッシュ関数であるbcryptインストール
has_secure_passwordを使ってパスワードをハッシュ化するためには、最先端のハッシュ関数であるbcryptが必要になります

```Gemfile
source 'https://rubygems.org'

gem 'rails',          '5.1.2'
gem 'bcrypt',         '3.1.11'
```

### has_secure_password追加

```app/models/user.rb
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
+  has_secure_password
end
```

has_secure_passwordには、仮想的なpassword属性とpassword_confirmation属性に対してバリデーションをする機能も(強制的に)追加されているためテストがこける

テストをパスさせるために、パスワードとパスワード確認の値を追加します。

```test/models_user_test.rb
  def setup
    @user = User.new(name: "Example User", email: "user@example.com",
                     password: "foobar", password_confirmation: "foobar")
  end

```

### パスワード最小文字数テスト

```test/models/user_test.rb
  test "password should be present (nonblank)" do
    @user.password = @user.password_confirmation = " " * 6
    assert_not @user.valid?
  end

  test "password should have a minimum length" do
    @user.password = @user.password_confirmation = "a" * 5
    assert_not @user.valid?
  end
```

```app/models/user.rb
validates :password, presence: true, length: { minimum: 6 }

```

## ユーザ追加

rails console上でユーザを追加する場合

```bash
$ rails console
>> User.create(name: "Michael Hartl", email: "mhartl@example.com",
?>             password: "foobar", password_confirmation: "foobar")
```



## resources :users

routes.rbファイルに
**resources :hoge** をかくと
RESTfulなアクションが有効になる

## debuggerメソッド

任意のコントロラー内に
**debugger**
を入れるとそのタイミングでデバッグができる

rails serverを実行しているターミナル


## createのセキュリティ

paramsハッシュ全体を初期化するという行為はセキュリティ上、極めて危険だからです。これは、ユーザーが送信したデータをまるごとUser.newに渡していることになります。ここで、Userモデルにadmin属性というものがあるとしましょう。この属性は、Webサイトの管理者であるかどうかを示します (この属性を実装するのは10.4.1になってからです)。admin=’1’という値をparams[:user]の一部に紛れ込ませて渡してしまえば、この属性をtrueにすることができます。これはcurlなどのコマンドを使えば簡単に実現できます。paramsハッシュがまるごとUser.newに渡されてしまうと、どのユーザーでもadmin=’1’をWebリクエストに紛れ込ませるだけでWebサイトの管理者権限を奪い取ることができてしまいます。

**Strong Parameters**を設定する

```ruby
params.require(:user).permit(:name, :email, :password, :password_confirmation)
```

このコードの戻り値は、paramsハッシュのバージョンと、許可された属性です (:user属性がない場合はエラーになります)。

これらのパラメータを使いやすくするために、user_paramsという外部メソッドを使うのが慣習になっています。このメソッドは適切に初期化したハッシュを返し、params[:user]の代わりとして使われます。

```
@user = User.new(user_params)
```

## バリデーショエラー

バリデーションエラーは
**@user.errors**
で取得できる


### エラー表示用

```ruby
<% if @user.errors.any? %>
 <ul>
    <% @user.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
<% end %>
```
**any?** は要素が1つでもある場合はtrue、ない場合はfalseを返します。

## 入力フォーム失敗時のテスト
ユーザー登録ボタンを押したときに (ユーザー情報が無効であるために) ユーザーが作成されないことを確認します
### ファイル作成

結合テスト用ファイル作成

```bash
rails generate integration_test users_signup
```

### postテスト
フォーム送信をテストするためには、 users_pathに対してPOSTリクエストを送信する必要があります

```ruby
assert_no_difference 'User.count' do
  post users_path, params: { user: { name:  "",
                                     email: "user@invalid",
                                     password:              "foo",
                                     password_confirmation: "bar" } }
end
```
assert_no_differenceメソッドのブロック内でpostを使い、メソッドの引数には’User.count’を与えています。これはassert_no_differenceのブロックを実行する前後で引数の値 (User.count) が変わらないことをテストしています。すなわちこのテストは、ユーザ数を覚えた後にデータを投稿してみて、ユーザ数が変わらないかどうかを検証するテストになります。したがって、次のコードと等価になります。

### ファイル全体

```ruby
require 'test_helper'

class UsersSignupTest < ActionDispatch::IntegrationTest

  test "invalid signup information" do
    get signup_path
    assert_no_difference 'User.count' do
      post users_path, params: { user: { name:  "",
                                         email: "user@invalid",
                                         password:              "foo",
                                         password_confirmation: "bar" } }
    end
    assert_template 'users/new'
  end
end
```

### エラーメッセージテスト

バリデーションエラー表示領域が表示されているか

```ruby
    assert_select 'div#error_explanation'
    assert_select 'div.field_with_errors'
```

## 入力フォーム成功時のテスト
```ruby
  test "valid signup information" do
    get signup_path
    assert_difference 'User.count', 1 do
      post users_path, params: { user: { name:  "Example User",
                                         email: "user@example.com",
                                         password:              "password",
                                         password_confirmation: "password" } }
    end
    follow_redirect!
    assert_template 'users/show'
  end
```

## フラッシュメッセージ

```ruby
flash[:success] = "Welcome to the Sample App!"
```

``` html
      <% flash.each do |message_type, message| %>
        <div class="alert alert-<%= message_type %>"><%= message %></div>
      <% end %>
```

### 一度だけのflashメッセージ
.nowをつけるとその後のリクエストからは消滅

```ruby
flash.now[:danger] = 'Invalid email/password combination'
```

## DBの中身リセット

```bash
rails db:migrate:reset
```


## 本番環境でのSSL

configに「本番環境ではSSLを使うようにする」という設定

```config/environments/production.rb```

``` ruby
Rails.application.configure do
  .
  .
  .
  # Force all access to the app over SSL, use Strict-Transport-Security,
  # and use secure cookies.
  config.force_ssl = true
  .
  .
  .
end
```

## ユーザパスワード検証

```ruby
user.authenticate(params[:session][:password])
```

## ログイン失敗テスト
- ログイン用のパスを開く
- 新しいセッションのフォームが正しく表示されたことを確認する
- わざと無効なparamsハッシュを使ってセッション用パスにPOSTする
- 新しいセッションのフォームが再度表示され、フラッシュメッセージが追加されることを確認する
- 別のページ (Homeページなど) にいったん移動する
- 移動先のページでフラッシュメッセージが表示されていないことを確認する

```ruby
require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest

  test "login with invalid information" do
    get login_path
    assert_template 'sessions/new'
    post login_path, params: { session: { email: "", password: "" } }
    assert_template 'sessions/new'
    assert_not flash.empty?
    get root_path
    assert flash.empty?
  end
end
```

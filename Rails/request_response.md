## リクエストからレスポンスまでの流れについて(GET編)
とあるSNSアプリがあります。ユーザーの一覧ページがブラウザに表示されるまでの流れを説明してください。なお、以下の言葉を使うことを条件とします。  
- /users (← URLです)  
- GET  
- コントローラ  
- routes.rb  
- モデル  
- ビュー  
- データベース  
---

WEBブラウザから`/users` というURLに対して`GET`のリクエストをWEBサーバ（Railsアプリケーション)に送る。  
具体的にはlink_to(aタグ生成)にusers_path(href属性のリンク先である/usersを出力)を指定することでHTML側で`<a href="/users">`が出力され`/users`に対して`GET`リクエストが送られる。

```
views

  = link_to '一覧', users_path, class: 'nav-link'

↓

HTML

　　<a class="nav-link" href="/users">一覧</a>  

```

WEBサーバ（Railsアプリケーション）はユーザーの一覧を取得する為に`routes.rb`に定義されているルーティングをもとに利用されるコントローラ#アクションを決定する。  
今回の場合だと`routes.rb`に`resources :users`を記述し、Usersコントローラーのindexアクションに割り振られるようハンドリングしている。  

```
routes.rb

Rails.application.routes.draw do
  resources :users
end

↓

rails routes  

users GET    /users(.:format)    users#index

``` 

Usersコントローラーはindexアクションに記述されているUserモデルに命令を出してUserモデルはデータベースから該当するデータをコントローラーに返す。   
(下記の指示であればユーザーの情報全てを抽出)  

```
  def index
    @users = User.all
  end
```

コントローラーは得られた結果をビューに渡して、ビューはHTMLを生成する。  
（下記ようにeachで回して一覧表示させる）

```
    <% @users.each do |user| %>
      <tr>
        <td><%= user.nickname %></td>
        <td><%= user.profile %></td>
      </tr>
    <% end %>
```

WEBブラウザはHTTPレスポンスの`Content-Type: text/html; charset=utf-8`に従い、ユーザーにわかりやすくHTMLをレンダリングする。    

![GET](https://gyazo.com/490e40ac92cdf63e2555c27a48255401.png)  

## リクエストからレスポンスまでの流れについて(POST編)  
とあるタスク管理アプリがあります。タスクを新規作成するときの流れを説明してください。以下のような要件とします。  

/tasks/new (← URLです)  
/tasks (← URLです)  
GET  
POST  
コントローラ  
routes.rb  
モデル  
ビュー  
データベース  
form_with  
action  
method  
name=tasks[name]  
params 

---  

### 新規作成ページを表示する    

WEBブラウザから`/tasks/new`というURLに対して`GET`のリクエストをWEBサーバ（Railsアプリケーション)に送る。  
link_to(aタグ生成)にnew_task_path(href属性のリンク先である/tasks/newを出力)を指定することでHTML側で`<a href="/tasks/new">`が出力され`/tasks/new`に対して`GET`リクエストが送られる。

```
views

  = link_to '新規登録', new_task_path, class: 'btn btn-primary'

↓

HTML

　　<a class="btn btn-primary" href="/tasks/new">新規登録</a>  

```

WEBサーバ（Railsアプリケーション）は`routes.rb`に定義されているルーティングをもとに利用されるコントローラ#アクションを決定する。  
今回の場合だと`routes.rb`に`resources :tasks`を記述し、Tasksコントローラーのnewアクションに割り振られるようハンドリングをしている。

```
routes.rb

Rails.application.routes.draw do
  resources :tasks
end  

↓

rails routes 

GET    /tasks/new(.:format)      tasks#new
``` 

TasksコントローラーのnewアクションでTaskモデルをインスタンス化してformで送られる値をデータベースに保存できるよう@taskに代入する。 

```
  def new
    @task = Task.new
  end
```

コントローラーは結果をビューに渡して、ビューはHTMLを生成する。  

WEBブラウザはリクエストが返ってきたらHTTPレスポンスの`Content-Type: text/html; charset=utf-8`に従い、ユーザーにわかりやすくHTMLをレンダリングする。 

![GET](https://gyazo.com/bec027e91b2099d4ac42e7b4d25d3c75.png)

### タスク名(name)を入力する

新規作成ページのformにはRailsで情報を送信するためのヘルパーメソッドである`form_with`を使用する。  
`form_with`を使うことにより入力フォームに必要なHTMLを簡単に作成することができる。

```
new.html.slim

= form_with model: @task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .form-group 
    = f.label :description
    = f.text_field :description, rows: 5, class: 'form-control', id: 'task_description'
  = f.submit nil, class: 'btn btn-primary'
```
↓  

```
HTML

<form action="/tasks" accept-charset="UTF-8" method="post">
  <input name="utf8" type="hidden" value="✓">
  <input type="hidden" name="authenticity_token" value="CbmpofpB8AlO5MAxV2QE5dpbMvAr4LbWudVMktrE0PkVNjDFJror6ZAPco2moQBXs+92/MC/IaDfGUWwpX9c6g==">
  <div class="form-group">
    <label for="task_name">名称</label>
    <input class="form-control" id="task_name" type="text" name="task[name]">
  </div>
  <input type="submit" name="commit" value="作成する" class="btn btn-primary" data-disable-with="作成する">
</form>
```
`form_with`は、HTML側に`method="post"`を自動で設定してくれる。  
また`persisted?`メソッドでデータベースに値があるかを確認しfalseなら新規登録(create)、trueなら更新(update)といった判断も自動的にしてくれる。

### 作成ボタンを押す  

作成ボタンを押下するとWEBブラウザから`/tasks`というURLに対して`POST`のリクエストがWEBサーバ（Railsアプリケーション)に送られる。  
form_withが`persisted?`メソッドを使って新規登録であることを判断し、tasks#createアクションのURL`/tasks`を指定することでHTML側で`<form action="/tasks">`が出力され`/tasks`に対して`POST`リクエストが送られる。また`params`はinputタグの`<name="task[name]">`に入力された値がHTTPリクエストの`Form Data`に格納されてリクエストが送られる。

WEBサーバ（Railsアプリケーション）は`routes.rb`に定義されているTasksコントローラーのcreateアクションに割り振られるようにハンドリングする。

```
rails routes 

POST   /tasks(.:format)          tasks#create
``` 

![POST](https://gyazo.com/4846eee216bd81d027609fa6b6ff447a.png)　　
![POST](https://gyazo.com/a8c777b3d844c07442556cf2a9e683c4.png)    

`params`はWEBサーバ（Railsアプリケーション)に送られるとRails側でハッシュ形式に変換される。  
```
[2] pry(#<TasksController>)> params[:task]
=> <ActionController::Parameters {"name"=>"a"} permitted: false>
```

createアクションに渡された`params`をデータベースに保存するには`strong_parameters`を使用する。

理由：コントローラーが受け取った`params`を自動的にモデルへ渡せるような仕様だった場合、攻撃者に悪用される可能性がある為。  
createアクションに`params`を記述した場合は、セキュリティ担保の為、例外が走る。  

`strong_parameters`はparamsの特定のキーに紐付く値だけを抽出する`require`メソッドとカラムのパラメーターのみをデータベースへ保存の許可をする`permit`メソッドを使用する。

`require`メソッドは、`name="task[name]"`でいうと`task`をさし、`permit`メソッドは`[name]`を指している。  

また、外部から不正に呼び出されることのないように`private`宣言下に記述する。

```
  空のタスクモデルのインスタンスにストロングパラメーターを引数としてセットする。

  def create
    @task = current_user.tasks.new(task_params)
    if @task.save
      redirect_to @task, notice: "タスク「#{@task.name}」を登録しました。"
    else
      render :new
    end  
  end

  strong_parameters↓

　private

  def task_params
    params.require(:task).permit(:name)
  end

```
 
モデルで設定しているvalidatesやデータベースのマイグレーション時に設定したnull: falseに反して登録をした場合、`render :new`によって入力された内容は保持された状態で新規登録画面が描画される。問題がなければ`@task.save`によってデータベースに保存される。

### タスクの詳細画面に遷移する  

データが保存されたら、createアクションの`redirect_to`で指定したページに飛ぶ。上記のように`@task`を指定した場合は自動的に詳細画面に遷移するようになっている。

![GET](https://gyazo.com/bec21194c074fd695ff98c677a964838.png)

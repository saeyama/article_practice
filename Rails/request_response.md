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

WEBブラウザ上で、`link_to`で指定されている`tasks_path`の `一覧`ボタンを押下するとWEBサーバーにHTTPリクエストが送られる。  
一覧ページを表示させるにはHTTPリクエストの`Request URL`に`routes.rb`で設定した一覧ページのエンドポイント`/users `を指定し、`Request Method`には`GET`を指定する。  

```
routes.rb

Rails.application.routes.draw do
  resources :users
end

↓

rails routes 

GET    /users/new(.:format)      users#new
``` 

Railsサーバーはリクエストをroutesに送り、routesはHTTPリクエスト（URLとHTTPメソッド）に合致しているUsersコントローラーのindexアクションに割り振られる。

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

WEBブラウザ上で、`link_to`で指定されている`new_task_path`の `新規登録`ボタンを押下するとWEBサーバーにHTTPリクエストが送られる。
新規作成ページを表示するにはHTTPリクエストの`Request URL`に`routes.rb`で設定した新規作成のエンドポイント`/tasks/new`のURLを指定し、`Request Method`には`GET`を指定する。  

```
routes.rb

Rails.application.routes.draw do
  resources :tasks
end  

↓

rails routes 

GET    /tasks/new(.:format)      tasks#new
``` 

Railsサーバーはリクエストをroutesに送り、routesはHTTPリクエスト（URLとHTTPメソッド）に合致したtasksコントローラーのnewアクションに振り分けられる。  

tasksコントローラーのnewアクションでTaskモデルをインスタンス化してフォームで送られる値をデータベースに保存できるよう@taskに代入する。 

```
  def new
    @task = Task.new
  end
```

コントローラーは結果をビューに渡して、ビューはHTMLを生成する。  

WEBブラウザはリクエストが返ってきたらHTTPレスポンスの`Content-Type: text/html; charset=utf-8`に従い、ユーザーにわかりやすくHTMLをレンダリングする。 

![GET](https://gyazo.com/bec027e91b2099d4ac42e7b4d25d3c75.png)

### タスク名(name)を入力する

新規作成ページのフォームにはrailsで情報を送信するためのヘルパーメソッドである`form_with`を使用する。  
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
HTMLに変換

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
また`persisted?`メソッドでデータベースに値があるかを確認しfalseなら新規登録(create)、trueなら更新(update)といった判断ができるようになっているのでaction属性も自動的に判別し設定してくれる。

### 作成ボタンを押す  

新規作成画面のフォームからタスク名(name)を入力し`form_with`のsubmitボタンを押下するとHTTPリクエストが送られる。  

HTTPリクエストの`Request URL`に`routes.rb`で設定したcreateアクションへ飛ばす為のエンドポイント`/tasks`を指定し、`Request Method`には`POST`を指定する。  

```
rails routes 

POST   /tasks(.:format)          tasks#create
``` 

![POST](https://gyazo.com/4846eee216bd81d027609fa6b6ff447a.png)　　
![POST](https://gyazo.com/a8c777b3d844c07442556cf2a9e683c4.png)  

Railsサーバーはリクエストをroutesに送り、routesはHTTPリクエスト（URLとHTTPメソッド）に合致したtasksコントローラーのcreateアクションに割り振られる。

```
  def create
    @task = current_user.tasks.new(task_params)
    if @task.save
      redirect_to @task, notice: "タスク「#{@task.name}」を登録しました。"
    else
      render :new
    end  
  end

　private

  def task_params
    params.require(:task).permit(:name)
  end

```
フォームで入力されたパラメーター(値)は`form_with`によってcreateアクションへ渡された後、paramsメソッドを使用してデータベースに保存される。  
paramsはcreateメソッドには書かず、permitメソッドを使って`ストロングパラメーター`を設定する。  
(理由：paramsで取得したパラメーターに悪意を持ったユーザーが保存したくないパラメーター属性を含めてしまうと、その情報まで保存されてしまう危険性があるため)

上記の`task_params`で設定している、`params.require(:task).permit(:name)`はformの`name="task[name]"`をさすので値がちゃんと送られるようになっている。 
もしモデルで設定しているvalidatesやデータベースのマイグレーション時に設定したnull: falseに反して登録をした場合、`render :new`によって入力した内容は保持されたまま新規登録画面が描画される。問題がなければ`@task.save`によってデータベースに保存される。


### タスクの詳細画面に遷移する  

データが保存されたら、createアクションの`redirect_to`で指定したページに飛ぶ。上記のように`@task`を指定した場合は自動的に詳細画面に遷移するようになっている。

![GET](https://gyazo.com/bec21194c074fd695ff98c677a964838.png)

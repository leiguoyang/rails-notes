# Rails notes

## Migration
Migration就是对数据库进行更改。有两个作用。

- 对数据库的schema进行修改。如增加新的table或在已有的table上增加新的column。
- 对数据库的data进行修改，如增加新的record.

`bin/rails db:migrate`就是进行migration.

`bin/rails db:rollback`就是撤销上一次的migration.

## Router
Router有两个目的。

- 指定对应的action来处理request。
- 创建route helper供controller和view来使用, 如`new_user_path`, `new_user_url`

### Route的格式

如有一个叫`Welcome`的controller，里面包含一个叫`first_message`的action，那么产生的path helper是`welcome_first_message_path`。即格式为`controllerName_actionName_path`

Prefix | Verb | URI Pattern | Controller#Action
--- | --- | --- | ---
welcome_index | GET | /welcome/index(.:format) | welcome#index
welcome_first_message | GET | /welcome/first_message(.:format) | welcome#first_message

## Commands
- `bin/rails server` 运行rails程序
- `bin/rails server -p $PORT -b $IP` 在一个特定的port或IP运行
- `rails routes` 列出所有产生的routes

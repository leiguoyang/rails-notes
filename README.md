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

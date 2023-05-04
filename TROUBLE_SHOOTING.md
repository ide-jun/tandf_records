# トラブルシューティング一覧
## `Couldn’t drop database 'db/development.sqlite3' rails aborted!`の対処法
### 原因
Rails自体のバグ。Windows環境でのみ発生する。理由は自分がアクセスしているファイルを削除できない、という仕様のため。
- `rails db:drop`
- `rails db:reset`
- `rails db:migrate:reset`
でこの現象が起き得る。
### 解決方法
手動でデータベースを削除する。削除するコマンドは、カレントディレクトリがdbに位置している場合は
```
cd db
del development.sqlite3
```
---
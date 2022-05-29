# RSpecでのテスト
- 次のコマンドでRSpecを実行した。
```
$ bundle exec rspec
```
- すると、次のようなエラーが出た。
```
 1) Admin::Users GET /new returns http success
     Failure/Error: expect(response).to have_http_status(:success)
       expected the response to have a success status code (2xx) but it was 302
     # ./spec/requests/admin/users_spec.rb:7:in `block (3 levels) in <main>'
```
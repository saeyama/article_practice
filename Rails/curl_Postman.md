### curlコマンドで特定の記事を更新してください  
```
curl -X POST -F 'article[title]=hogehoge' -F 'article[content]=fugafuga' -F '_method=patch' http://localhost:3000/articles/3
```

### curlコマンドで特定の記事を削除してください  
```
curl -X POST -F '_method=delete' http://localhost:3000/articles/4 
```  

※下記のように`PATCH`・`DELETE`で指定することもできるがRailsには`POST`・`GET`以外のRequest Methodがないので、上記で対応。

```
curl -X PATCH -F 'article[title]=hogehoge' -F 'article[content]=fugafuga' http://localhost:3000/articles/2  

curl -X DELETE  http://localhost:3000/articles/2
```

### postmanで特定の記事を更新してください  
![curl_PATCH](https://gyazo.com/8372d98ab03201679253d8dabee96a23.png)  

### postmanで特定の記事を削除してください  
![curl_DELETE](https://gyazo.com/6f8071964ea5de723dd675719aa21177.png)
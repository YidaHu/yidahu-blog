# Angular5 API发起HTTP几种请求方式

# 前言

大多数前端应用都需要通过 HTTP 协议与后端服务器通讯。现代浏览器支持使用两种不同的 API 发起 HTTP 请求：`XMLHttpRequest` 接口和 `fetch()` API。

这里记录使用SpringMVC后台的RESTful API增删改查几种形式接口，分别以POST、DELETE、PUT、GET等方式进行HTTP请求。

<!--MORE-->

## 添加数据（POST方式）

```typescript
const body = {name: this.name, num: this.num};
addData(body) {
    return this.http.post('http://localhost:8081/api/root/add', body, {
      headers: new HttpHeaders().set('Content-Type', 'application/json;charset=UTF-8'),
      withCredentials: true,
    });
  }
```

## 删除数据（DELETE方式）

```typescript
deleteData() {
    return this.http.delete("http://localhost:8081/api/root/delete", {
      headers: new HttpHeaders().set('Content-Type', 'application/json;charset=utf-8'),
      withCredentials: true,
      params: {id: id}
    });
  }
```

## 修改数据（PUT方式）

```typescript
const body = {
        id: this.id,
        name: this.name,
        num: this.num,
      };
updateData(body) {
    return this.http.put('http://localhost:8081/api/root/update', body, {
      headers: new HttpHeaders().set('Content-Type', 'application/json;charset=UTF-8'),
      withCredentials: true,
    });
  }
```

## 查询数据（GET方式）

```typescript
getData() {
    return this.http.get('http://localhost:8081/api/root/list', {
      headers: new HttpHeaders().set('Content-Type', 'application/json;charset=utf-8'),
      withCredentials: true,
      params: {'page': 1,'size': 10}
    });
  }
```


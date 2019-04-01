## 短路运算

只有当第一个运算数的值无法确定逻辑运算的结果时，会按照顺序对剩下运算数进行求值。

### 一真即真

`一真即真` 指的是 `||` 或运算符，当一个值为真并停止对后面的判断。

#### 简化条件语句

在开发是时候，偶尔会遇到只有一行代码的条件语句：

```javascript
if(!name) {
    initName()
}
```

不知道同学会不会和我一样会觉得这样写不好看，这里可以利用 `||` 来简化代码：

```javascript
name || initName()
```

### 一假即假

`一假即假` 指的是 `&&` 或运算符，当一个值为假时则会停止判断，为真时则会一直判断下去。

#### 简化条件语句

在开发是时候，偶尔会遇到只有一行代码的条件语句：

```javascript
// 假设后端返回的用户数据为 data
const data = {
    name: 'xiaoer',
    age: 18,
    company: null
}

if (data.company && data.company.name) {
    initCompany();
}
```

`&&` 和 `||` 一样也可以用来简化条件语句：

```javascript
data.company && data.company.name && initCompany();
```

## 逗号运算符

### for 循环

在使用 for 循环的时候，可能不止需要迭代一个参数，可以利用逗号表达式：

##### 脚本

```javascript
let i = 0, j = 0, times = 5;

for (let i = 0, j = 0; i < times; i++, j++, j++){
    console.log(i, j);
}
```

### 简化语句

在写一些简短的函数时常常写成下面这样：

```javascript
const ins = (x) => {
    x++;
    return x;
}

[{count: 1}].map((x) => {
    x.count++;
    return x;
})
```

如果需要进行的操作很多倒是需要写得详细方便他人阅读，而且开发并不是一个人。如果是这种一点点操作的时候，可以利用逗号运算符来简化：

```javascript
const ins = (x) => (x++, x)

[{count: 1}].map((x) => (x.count++, x));
```

### 交换数据

在不增加变量的情况下如何调换a和b的值。

当然也可以用 es6 的 `spread` 新语法做到：

```
let a = 1, b = 2;
[a, b]=[b, a]
```

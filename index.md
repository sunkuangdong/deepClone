## 手写深拷贝

这是一个经典的面试题，关于深拷贝我们需要考虑很多

	1. 首先我们要考虑基本类型
 	2. 其次考虑复杂类型的处理判断
 	3. 考虑环特殊性

### 代码分析

#### JSON序列化

针对于简单的深拷贝，在不考虑各种类型的情况下可以使用JSON序列化进行操作。

代码如下：

```markdown
let a = {
            b: 1,
            c: [1, 2, 3],
            d: {
                d1: 'd1',
                d2: 'd2'
            }
};
let b = JSON.parse(JSON.stringify(a));
```

对于使用JSON序列化实现深拷贝来说，他是有很多缺点的：

 1. 不支持函数类型

    ```
    let a = {
    	f: function() {}
    };
    let b = JSON.parse(JSON.stringify(a));
    console.log(b); // {}
    ```

2. JSON不支持的格式，都不支持

   ```
   let a = {
   	u: undefined,
   	reg: /hi/, // 正则
   }
   let b = JSON.parse(JSON.stringify(a));
   console.log(b); // {reg:{}}
   ```

3. 不支持环引用

   ```
   let a = {
   	name: 'a'
   };
   a.self = a; // 让a引用自己就会形成环
   let b = JSON.parse(JSON.stringify(a));
   console.log(b); // 报错 Uncaught TypeError: Converting circular structure to JSON
   ```

### 手写一个深拷贝

我们知道了JSON序列化并不完美，所以要想实现一个深拷贝还得自己深入研究。

我们来先按照上面思考的步骤，实现一个深拷贝

这里提供一个单元测试用的插件

将下面代码复制到package.json文件中

```
{
  "name": "js.demo-1",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "test": "mocha test/**/*.js",
    "class": "mocha class/**/*.js"
  },
  "devDependencies": {
    "chai": "^4.2.0",
    "mocha": "^6.2.0",
    "sinon": "^7.4.1",
    "sinon-chai": "^3.3.0"
  }
}
```

然后执行：

```
npm install
```

1. 先进行对基本类型的深拷贝

   ```
   function deepClone(source) {
   	return source;
   }
   ```

2. 然后我们继续实现复杂的类型，我们先慢慢的去解决复杂类型

   * 首先我们对一个普通的对象进行深拷贝

   ```
   function deepClone(source) {
   	// 判断是否是对象
   	if(source instanceof Object){
   		let dist = new Object()
   		for(let key in source) {
   			dist[key] = deepClone(source[key])
   		}
   		return dist;
   	}
   	return source;
   }
   ```

   * 我们继续深入对数组对象、函数进行深拷贝

   ```
   function deepClone(source) {
   	// 判断是否是对象
   	if(source instanceof Object){
   		let dist;
   		// 判断是否是Array数组
   		if(source instanceof Array){
   			dist = new Array()
   		}else if(source instanceof Function){
   			dist = function(){
   				return source.apply(this, arguments)
   			}
   		} else{
   			dist = new Object()
   		}
   		for(let key in source) {
   			dist[key] = deepClone(source[key])
   		}
   		return dist;
   	}
   	return source;
   }
   ```

   * 接下来我们来思考如何拷贝环

     * 什么样的东西是环？？？如下：

       ```
       const n = {
                       name: "孙褚"
                 };
       n.self = n;
       ```

       像这种形成了一个环形的引用，就叫做环。

       我们如何对环进行判断呢？

     * 思路是： 

     * 第一步思考： 

       1. 如果他是一个对象类型的时候，他是否会其他类型？
       2. 是就继续之前类型的拷贝
       3. 不是我们来对环进行处理

     * 第二步思考：

       1. 怎么处理环？首先，我的环n与我拷贝的返回值（假设是n1）是完全相同的，也就是说：环 === source。
       2. 基于我们现在的代码来看，如果不是Array或者Function的类型时，他一定会创建一个对象：dist = new Object()
       3. 

   ```
   function deepClone(source) {
   	if(source instanceof Object){
   		if(){
   			
   		}else {
               let dist;
               // 判断是否是Array数组
               if(source instanceof Array){
                   dist = new Array()
               }else if(source instanceof Function){
                   dist = function(){
                       return source.apply(this, arguments)
                   }
               } else{
                   dist = new Object()
               }
               for(let key in source) {
                   dist[key] = deepClone(source[key])
               }
               return dist;
           }
   		return source;
   	}
   }
   ```



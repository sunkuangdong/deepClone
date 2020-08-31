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

上面这个步骤是用来之后进行单元测试工作的



接下来我们来实现一个深拷贝：

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

       1. 我们要对一个环进行深拷贝，并判断如果拷贝完成了，返回拷贝的值
       2. 这样我们就需要一个内存，来对将要拷贝的 和 拷贝的进行判断
       3. 如果缓存中存在我传递过来的参数，我就返回出我拷贝的
       4. 不存在我就继续拷贝，返回一个undefined

   ```
   // 创建一个内存
   let cache = [];
   function deepClone(source) {
   	if(source instanceof Object){
   		// 看他是否被拷贝过,如果拷贝过就直接返回拷贝的对象,停止拷贝
           // 如果没有拷贝过就继续拷贝
   		let cacheDist = findCache(source)
   		if(cacheDist){
   			return cacheDist;
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
               // 将传入的 和 拷贝的都放进内存中
               // 刚开始是没拷贝过, 所以第一次cache是[]
               // 拷贝完成cache是[source, dist], dist是完成拷贝的值
               // 最后依然会返回dist
               cache.push([source, dist])
               for(let key in source) {
                   dist[key] = deepClone(source[key])
               }
               return dist;
           }
   		return source;
   	}
   }
   
   function findCache(source) {
   	// 刚开始是个空数组
       // 如果是空数组就会返回undefined, 说明cache中没有拷贝数据dist
   	for(let i = 0;i < cache.length;i++){
   		// 然后对环进行拷贝dist = new Object();
   		// 拷贝完成之后, 返回拷贝完成的数据
   		if(cache[i][0] === source){
   			return cache[i][1] // {name: "孙褚"}
   		}
   	}
   	return undefined;
   }
   const n = { name: "孙褚" };
   n.self = n;
   const n1 = deepClone(n);
   ```

我们思考完环之后还有什么类型需要去具体考虑呢?

Json序列化不支持正则，也不支持时间，他会将时间转换成字符串

我们来拷贝这两个类型：

```
let cache = [];

function deepClone(source) {
	if(source instanceof Object){
		let cacheDist = findCache(source)
		if(cacheDist){
			return cacheDist;
		}else {
            let dist;
            if(source instanceof Array){
                dist = new Array()
            }else if(source instanceof Function){
                dist = function(){
                    return source.apply(this, arguments)
                }
            }else if(source instanceof RegExp){
            	// 拷贝正则
            	dist = new RegExp(source.source, source.flags)
            }else if(source instanceof Date){
            	// 拷贝时间
            	dist = new Date(source)
            } else{
                dist = new Object()
            }
            cache.push([source, dist])
            for(let key in source) {
                dist[key] = deepClone(source[key])
            }
            return dist;
        }
		return source;
	}
}

function findCache(source) {
	for(let i = 0;i < cache.length;i++){
		if(cache[i][0] === source){
			return cache[i][1] // {name: "孙褚"}
		}
	}
	return undefined;
}

const n = new RegExp("hi\\d+", "gi");
const n1 = deepClone(n);
```

正则是接受两个参数的，时间是接受一个参数的



我们接下来来考虑最后一个问题：

我们要不要跳过原型属性？？？就是说，我要拷贝它的原型么？

其实我认为是不需要的，因为很浪费资源，我们现在的深拷贝是拷贝了原型的，那么我如何跳过对原型属性的拷贝呢？

上代码：

```
let cache = [];

function deepClone(source) {
	if(source instanceof Object){
		let cacheDist = findCache(source)
		if(cacheDist){
			return cacheDist;
		}else {
            let dist;
            if(source instanceof Array){
                dist = new Array()
            }else if(source instanceof Function){
                dist = function(){
                    return source.apply(this, arguments)
                }
            }else if(source instanceof RegExp){
            	// 拷贝正则
            	dist = new RegExp(source.source, source.flags)
            }else if(source instanceof Date){
            	// 拷贝时间
            	dist = new Date(source)
            } else{
                dist = new Object()
            }
            cache.push([source, dist])
            for(let key in source) {
            	// 如果key是source本身属性，不是原型上的，我们就进行拷贝
            	if (source.hasOwnProperty(key)) {
                	dist[key] = deepClone(source[key])
            	}
            }
            return dist;
        }
		return source;
	}
}

function findCache(source) {
	for(let i = 0;i < cache.length;i++){
		if(cache[i][0] === source){
			return cache[i][1] // {name: "孙褚"}
		}
	}
	return undefined;
}
```

这样，我们的深拷贝就是比较完整的了，如果是要考虑爆站问题，数据结构就要改变，这里就暂时不考虑了

总结：

1. JSON序列化实现深拷贝缺点
2. 手写一个深拷贝要考虑那些类型
3. 环形如何思考与处理
4. 正则 和 时间类型参数应对
5. 原型属性不进行拷贝的应对办法
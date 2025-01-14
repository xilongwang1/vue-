[toc]

## axios

```js
import axios, { AxiosRequestConfig, AxiosResponse, AxiosError } from "axios";
import { ResponseBody } from "@/api/typing";
import { localStorage } from "@/utils/local-storage";
import { STORAGE_TOKEN_KEY } from "@/store/mutation-type";
import { notification } from "ant-design-vue";

// 这里是用于设定请求后端时，所用的 Token KEY
// 可以根据自己的需要修改，常见的如 Access-Token，Authorization
// 需要注意的是，请尽量保证使用中横线`-` 来作为分隔符，
// 避免被 nginx 等负载均衡器丢弃了自定义的请求头
export const REQUEST_TOKEN_KEY = "Access-Token";

// 创建 axios 实例
const request = axios.create({
	// API 请求的默认前缀
	baseURL: process.env.VUE_APP_API_BASE_URL,
	timeout: 6000, // 请求超时时间
});

// 异常拦截处理器
const errorHandler = (error: AxiosError): Promise<any> => {
	if (error.response) {
		const { data = {}, status, statusText } = error.response;
		// 403 无权限
		if (status === 403) {
			notification.error({
				message: "Forbidden",
				description: (data && data.message) || statusText,
			});
		}
		// 401 未登录/未授权
		if (status === 401 && data.result && data.result.isLogin) {
			notification.error({
				message: "Unauthorized",
				description: "Authorization verification failed",
			});
		}
	}
	return Promise.reject(error);
};

// 请求拦截器
const requestHandler = (
	config: AxiosRequestConfig
): AxiosRequestConfig | Promise<AxiosRequestConfig> => {
	const savedToken = localStorage.get(STORAGE_TOKEN_KEY);
	// 如果 token 存在
	// 让每个请求携带自定义 token, 请根据实际情况修改
	if (savedToken) {
		config.headers[REQUEST_TOKEN_KEY] = savedToken;
	}
	return config;
};

// Add a request interceptor
request.interceptors.request.use(requestHandler, errorHandler);

// 响应拦截器
const responseHandler = (
	response: AxiosResponse
): ResponseBody<any> | AxiosResponse<any> | Promise<any> | any => {
	return response.data;
};

// Add a response interceptor
request.interceptors.response.use(responseHandler, errorHandler);

export { AxiosResponse };

export default request;
```

## 请求方式

### get 请求

```js
import qs from "qs";
axios.get("/user?ID=12345");

axios.get("/user", {
	params: {
		ID: 12345,
	},
});
// get传数组 https://www.cnblogs.com/lou-0820/p/13807466.html
axios.get(`/aaa/cccc`, {
	params: "数据",
	// `paramsSerializer` 是一个负责 `params` 序列化的函数
	paramsSerializer: function (params) {
		return qs.stringify(params, { arrayFormat: "repeat" });
	},
});
// let params = { order_id: [1, 2, 3] }
//  axios.get('www.baidu.com', params); // www.baidu.com?order_id=1&order_id=2&order_id=3
```

### post 请求

```js
axios.post("/user", {
	firstName: "Fred",
	lastName: "Flintstone",
});

axios.post("/api/user", "", { params: { a: 1 } });

//form表单化
axios.post("/api/user", qs.stringify("数据"), {
	headers: { "Content-Type": "application/x-www-form-urlencoded" },
});
```

### put

```js
axios.put(`/aa/cc`, "数据");
//user?a=1
axios.put("/api/user", "", { params: { a: 1 } });
//form表单化
axios.put("/api/user", qs.stringify("数据"), {
	headers: { "Content-Type": "application/x-www-form-urlencoded" },
});
```

### 执行多个并发请求

```js
function getUserAccount() {
	return axios.get("/user/12345");
}

function getUserPermissions() {
	return axios.get("/user/12345/permissions");
}

axios.all([getUserAccount(), getUserPermissions()]).then(
	axios.spread(function (acct, perms) {
		// 两个请求现在都执行完成
	})
);
```

## axios API

### axios(config)

```js
// 发送 POST 请求
axios({
	method: "post",
	url: "/user/12345",
	data: {
		firstName: "Fred",
		lastName: "Flintstone",
	},
});
```

```js
// 获取远端图片
axios({
	method: "get",
	url: "http://bit.ly/2mTM3nY",
	responseType: "stream",
}).then(function (response) {
	response.data.pipe(fs.createWriteStream("ada_lovelace.jpg"));
});
```

### axios(url[, config])

```js
// 发送 GET 请求（默认的方法）
axios("/user/12345");
```

### 请求方法别名

- axios.request(config)
- axios.get(url[, config])
- axios.delete(url[, config])
- axios.head(url[, config])
- axios.options(url[, config])
- axios.post(url[, data[, config]])
- axios.put(url[, data[, config]])
- axios.patch(url[, data[, config]])
- 注意: 在使用别名方法时， `url、method、data` 这些属性都不必在配置中指定。

### 并发函数

- axios.all(iterable)
- axios.spread(callback)

## 创建实例

- 可以使用自定义配置新建一个 axios 实例

### axios.create([config])

```js
const instance = axios.create({
	baseURL: "https://some-domain.com/api/",
	timeout: 1000,
	headers: { "X-Custom-Header": "foobar" },
});
```

### 实例方法

- axios#request(config)
- axios#get(url[, config])
- axios#delete(url[, config])
- axios#head(url[, config])
- axios#options(url[, config])
- axios#post(url[, data[, config]])
- axios#put(url[, data[, config]])
- axios#patch(url[, data[, config]])

## 请求配置

- 这些是创建请求时可以用的配置选项。只有 url 是必需的。如果没有指定 method，请求将默认使用 get 方法

```js

{
   // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // default

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data, headers) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

   // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

   // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // default

  // `adapter` 允许自定义处理请求，以使测试更轻松
  // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
  adapter: function (config) {
    /* ... */
  },

 // `auth` 表示应该使用 HTTP 基础验证，并提供凭据
  // 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

   // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // default

  // `responseEncoding` indicates encoding to use for decoding responses
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // default

   // `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
  xsrfHeaderName: 'X-XSRF-TOKEN', // default

   // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

   // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,

  // `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },

  // `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
  // 如果设置为0，将不会 follow 任何重定向
  maxRedirects: 5, // default

  // `socketPath` defines a UNIX Socket to be used in node.js.
  // e.g. '/var/run/docker.sock' to send requests to the docker daemon.
  // Only either `socketPath` or `proxy` can be specified.
  // If both are specified, `socketPath` is used.
  socketPath: null, // default

  // `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  // `keepAlive` 默认没有启用
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名称和端口
  // `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  // 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定用于取消请求的 cancel token
  // （查看后面的 Cancellation 这节了解更多）
  cancelToken: new CancelToken(function (cancel) {
  })
}
```

## 响应结构

```js
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 服务器响应的头
  headers: {},

   // `config` 是为请求提供的配置信息
  config: {},
 // 'request'
  // `request` is the request that generated this response
  // It is the last ClientRequest instance in node.js (in redirects)
  // and an XMLHttpRequest instance the browser
  request: {}
}
```

- 使用 then 时，你将接收下面这样的响应 :

```js
axios.get("/user/12345").then(function (response) {
	console.log(response.data);
	console.log(response.status);
	console.log(response.statusText);
	console.log(response.headers);
	console.log(response.config);
});
```

- 在使用 catch 时，或传递 rejection callback 作为 then 的第二个参数时，响应可以通过 error 对象可被使用，正如在错误处理这一节所讲。

## 配置默认值

### 全局的 axios 默认值

```js
axios.defaults.baseURL = "https://api.example.com";
axios.defaults.headers.common["Authorization"] = AUTH_TOKEN;
axios.defaults.headers.post["Content-Type"] =
	"application/x-www-form-urlencoded";
```

### 自定义实例默认值

```js
// Set config defaults when creating the instance
const instance = axios.create({
	baseURL: "https://api.example.com",
});

// Alter defaults after instance has been created
instance.defaults.headers.common["Authorization"] = AUTH_TOKEN;
```

### 配置的优先顺序

- 配置会以一个优先顺序进行合并。这个顺序是：在 lib/defaults.js 找到的库的默认值，然后是实例的 defaults 属性，最后是请求的 config 参数。后者将优先于前者。这里是一个例子：

```js
// 使用由库提供的配置的默认值来创建实例
// 此时超时配置的默认值是 `0`
var instance = axios.create();

// 覆写库的超时默认值
// 现在，在超时前，所有请求都会等待 2.5 秒
instance.defaults.timeout = 2500;

// 为已知需要花费很长时间的请求覆写超时设置
instance.get("/longRequest", {
	timeout: 5000,
});
```

## 拦截器

- 在请求或响应被 then 或 catch 处理前拦截它们。

```js
// 添加请求拦截器
axios.interceptors.request.use(
	function (config) {
		// 在发送请求之前做些什么
		return config;
	},
	function (error) {
		// 对请求错误做些什么
		return Promise.reject(error);
	}
);

// 添加响应拦截器
axios.interceptors.response.use(
	function (response) {
		// 对响应数据做点什么
		return response;
	},
	function (error) {
		// 对响应错误做点什么
		return Promise.reject(error);
	}
);
```

- 如果你想在稍后移除拦截器，可以这样:

```js
const myInterceptor = axios.interceptors.request.use(function () {
	/*...*/
});
axios.interceptors.request.eject(myInterceptor);
```

- 可以为自定义 axios 实例添加拦截器

```js
const instance = axios.create();
instance.interceptors.request.use(function () {
	/*...*/
});
```

## 错误处理

```js
axios.get("/user/12345").catch(function (error) {
	if (error.response) {
		// The request was made and the server responded with a status code
		// that falls out of the range of 2xx
		console.log(error.response.data);
		console.log(error.response.status);
		console.log(error.response.headers);
	} else if (error.request) {
		// The request was made but no response was received
		// `error.request` is an instance of XMLHttpRequest in the browser and an instance of
		// http.ClientRequest in node.js
		console.log(error.request);
	} else {
		// Something happened in setting up the request that triggered an Error
		console.log("Error", error.message);
	}
	console.log(error.config);
});
```

- 可以使用 validateStatus 配置选项定义一个自定义 HTTP 状态码的错误范围。

```js
axios.get("/user/12345", {
	validateStatus: function (status) {
		return status < 500; // Reject only if the status code is greater than or equal to 500
	},
});
```
## 取消
* 使用 cancel token 取消请求
> Axios 的 cancel token API 基于cancelable promises proposal，它还处于第一阶段。
* 可以使用 CancelToken.source 工厂方法创建 cancel token，像这样：
```js
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');
```
* 还可以通过传递一个 executor 函数到 CancelToken 的构造函数来创建 cancel token：
```js

var CancelToken = axios.CancelToken;
var cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

// 取消请求
cancel();
```

## 使用 application/x-www-form-urlencoded format

默认情况下，axios将JavaScript对象序列化为JSON。 要以application / x-www-form-urlencoded格式发送数据，您可以使用以下选项之一。

### 浏览器

在浏览器中，您可以使用URLSearchParams API，如下所示：
```js
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);

// 请注意，所有浏览器都不支持URLSearchParams（请参阅caniuse.com），但可以使用polyfill（确保填充全局环境）。
```
或者，您可以使用qs库编码数据
```js
const qs = require('qs');
axios.post('/foo', qs.stringify({ 'bar': 123 }));
```
或者以另一种方式（ES6），
```js
import qs from 'qs';
const data = { 'bar': 123 };
const options = {
  method: 'POST',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  data: qs.stringify(data),
  url,
};
axios(options);
```
### Node.js

在node.js中，您可以使用querystring模块，如下所示：
```js
const querystring = require('querystring');
axios.post('http://something.com/', querystring.stringify({ foo: 'bar' }));
```
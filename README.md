JavaScript Babel Import Export Transform Demo
=============================================

```
npm install
npm run demo
```

由于我们在babel中设置了环境为node，所以Babel将使用node的模块系统(commonjs)来编译代码，
放在`compiled`目录下。


## export

```
export function toUpper(str) {
  return str.toUpperCase();
}
```

将被编译成：

```
Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.toUpper = toUpper;

function toUpper(str) {
  return str.toUpperCase();
}
```

可见基本上就是把`export function`变成了`module.exports`下对应的某个function，
同时给`module.exports`增加了一个`__esModule`作为标识符。

## import

```
import * as util from './util'

console.log(`Hello, ${util.toUpper('babel')}`);
```

将被编译成：

```
var util = _interopRequireWildcard(require("./util"));

function _interopRequireWildcard(obj) {
  if (obj && obj.__esModule) {
    return obj;
  } else {
    var newObj = {};
    if (obj != null) {
      for (var key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
          var desc = Object.defineProperty && Object.getOwnPropertyDescriptor ? Object.getOwnPropertyDescriptor(obj, key) : {};
          if (desc.get || desc.set) {
            Object.defineProperty(newObj, key, desc);
          } else {
            newObj[key] = obj[key];
          }
        }
      }
    }
    newObj.default = obj;
    return newObj;
  }
}

console.log("Hello, ".concat(util.toUpper('babel')));
```

可见它也利用了node的`require`来导入模块，同时对导入的模块进行判断：

- 如果是`__esModule`，则直接使用
- 否则的话，生成一个新的对象，把原module所有的属性copy给它，然后把原module作为新对象的`default`属性

为什么要这么做？

看起来是为了方便直接使用commonjs模块的代码，通过给它增加一个`default`属性，方便使用者使用这种语法：

```
import x from './x'
```

而不需要

```
import * as x from './x'
```

如果被import的模块本身就是es module，并且没有定义`default`，则将不会为它增加`default`，
因为这说明那个module本来就不打算提供`default`。


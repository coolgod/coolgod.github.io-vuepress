```
function reportableClassDecorator<T extends { new (...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    reportingURL = "http://www...";
  };
}

@reportableClassDecorator
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}
```

最近在看TS关于decorator的文档时看到上面这样一段，其中这部分语法非常奇怪：
```
{ new (...args: any[]): {} }
```

稍微查阅了一下资料，作如下总结。

首先，可以在一个interface中定义所谓的 _construct signature_，官方示例代码如下：
```
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}
 
interface ClockInterface {
  tick(): void;
}
 
const Clock: ClockConstructor = class Clock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log("beep beep");
  }
};
 
let clock = new Clock(12, 17);
clock.tick();
```

这里为什么要把`ClockConstructor`和`ClockInterface`分开写，而不是都合并到一个interface里，是关乎到一个class存在static side和instance side两方面信息以及`implements`关键字只检查instance side信息的原因，详细内容官方文档有专门讲述所以不再赘述。

于是，奇怪的语法看起来比较像带有construct signature的匿名inline interface，就是下面这个意思：
```
interface AnonymousInlineInterface {
  new (...args: any[]): {};
}

function reportableClassDecorator<T extends AnonymousInlineInterface>(constructor: T) {
  return class extends constructor {
    reportingURL = "http://www...";
  };
}
```

这里的return type是`{}`没有什么实际的意义，相当要求construct signature返回类型是一个空的interface，也就是没有任何要求，就是下面这个意思：
```
interface Dummy {}

interface AnonymousInlineInterface {
  new (...args: any[]): Dummy;
}

function reportableClassDecorator<T extends AnonymousInlineInterface>(constructor: T) {
  return class extends constructor {
    reportingURL = "http://www...";
  };
}
```
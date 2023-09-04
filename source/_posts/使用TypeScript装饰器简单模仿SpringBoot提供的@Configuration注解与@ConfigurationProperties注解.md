title: 使用TypeScript装饰器简单模仿SpringBoot提供的@Configuration注解与@ConfigurationProperties注解
categories: 小玩意
tags: [TypeScript,JavaScript]
date: 2022-01-02 15:25:00
---
> 这两天才开始学习SpringBoot，遇到了两个很有意思的注解，打算用TypeScript模拟一下

## 示例
配置文件
```yml
name: UpYou
map: {
    name: UpYou,
    age: 18
}
```

```ts
@ConfigurationProperties({ prefix: 'map' })
@Configuration
class Preson {
    public age?: number = undefined;

    public name?: string = undefined;

    constructor() {}
}

// 获取组件
const component = getComponentInIOC(Preson);
```
输出：`Preson { age: 18, name: 'UpYou' }`

## @Configuration
这个注解是用来注册组件的，在SpringBoot也是比较重量级的东西了，ts中如何使用以及书写装饰器我就不多说了，网上有很多案例与教程

这是一个容器，所有注册的组件都在这
```ts
interface IOCMapType {
    [index: string]: any;
}
// IOC容器
const IOCMap: IOCMapType = {};
```

### Configuration方法
```ts
function Configuration(target: Class) {
    const name = target.name;
    const inIOC = findComponentInIOCByName(name);
    if (!inIOC) {
        throw new Error('the component exists in container.');
    }
    addComponentToCantainer(target);
}
```

### findComponentInIOCByName方法
这个方法主要查找容器中是否存在某个主键
```ts
// 在容器中通过组件名查找组件
function findComponentInIOCByName(name: string) {
    return IOCMap[name] === void 0;
}
```


### addComponentToCantainer方法
这个主要用来注册组件到容器
```ts
// 将组件注册到容器
function addComponentToCantainer(component: Class) {
    IOCMap[component.name] = new component();
}
```
现在这个修饰器就写好了

## @ConfigurationProperties
这个注解是用来绑定配置的
例如在某个规定位置下有一个`application.yml`文件，这是一个配置文件，那么通过这个注解可以将配置文件中对应的配置映射到被标注的类中。

例如我们在某个位置有一个`application.yml`文件
```ts
name: UpYou
map: {
    name: UpYou,
    age: 18
}
```
我们需要使用`js-yaml`插件来读取它
```ts
// 获取配置文件
const yml = fs.readFileSync('application.yml');
const config: any = yaml.load(yml.toString());
```
读取出来就是JSON格式
`{ name: 'UpYou', map: { name: 'UpYou', age: 18 } }`

### ConfigurationProperties方法
```ts
type ConfigurationPropertiesParam = {
    prefix: string;
};
// 绑定配置文件
function ConfigurationProperties(args: ConfigurationPropertiesParam) {
    const nodes = args.prefix.split('.');
    return function (target: Class) {
        const component = getComponentInIOC(target.name);
        let conf = config;
        // 过滤配置文件
        for (const node of nodes) {
            if (conf[node] === void 0) {
                break;
            }
            conf = conf[node]
        }
        // 绑定配置文件对应值
        for (const field in conf) {
            if (Object.prototype.hasOwnProperty.call(component, field)) {
                if (component[field] instanceof Function) {
                    break;
                }
                component[field] = conf[field];
            }
        }
    };
}
```
这个循环主要将对应的配置文件过滤出来，例如`prefix`是`map.test`，需要拿的就是配置文件中`map`里边的`test`，通过`.`分割`prefix`的内容，然后一层一层循环进去就能拿到对应配置了。
```ts
const nodes = args.prefix.split('.');
for (const node of nodes) {
    if (conf[node] === void 0) {
        break;
    }
    conf = conf[node]
}
```
直接设置组件的字段就可以了。
```ts
// 绑定配置文件对应值
for (const field in conf) {
    if (Object.prototype.hasOwnProperty.call(component, field)) {
        if (component[field] instanceof Function) {
            break;
        }
        component[field] = conf[field];
    }
}
```

### getComponentInIOC方法
这个方法用来获取容器中的组件
```ts
type Class = { new (...args: any[]): {} };
function getComponentInIOC(component: string | Class) {
    const name = component instanceof Function ? component.name : component;
    if (findComponentInIOCByName(name)) {
        throw new Error('no such component in container...');
    }
    return IOCMap[name];
}
```


> 模拟完之后对SpringBoot了解更深了...
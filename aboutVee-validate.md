# Vee-Validate笔记
## 关于Validation Provider
1.原理:ValidationProvider 通过 **scoped-slots**(作用域插槽) 将校验产生的数据(比如校验不通过的错误信息)注入到**slot props(插槽props)**中，使插槽模板组件`<ValidationProvider></ValidationProvider>`能够读取校验信息.

2.注册一个`Validation Provider`组件:
  2.1 组件内注册:
  ```
  import { ValidationProvider } from 'vee-validate';
  export default {
    components: {
      ValidationProvider
    }
  };
  ```  
  2.2 全局注册:
  ```
  import { ValidationProvider } from 'vee-validate';
  Vue.component('ValidationProvider', ValidationProvider);
  ```
3.注册组件完成后,就可以在template模板中使用`ValidationProvide`组件了。一般来说，需要用`ValidationProvider`元素包裹住你将要校验的`input`元素:
```
<ValidationProvider v-slot="v">
  <input type="text" v-model="value">
</ValidationProvider>
```
**注意，需要校验的`input`元素务必要用`v-model`或者`v-bind`绑定一个响应式数据，用于提示`ValidationProvider`将要校验`VueComponent(组件实例)`身上的哪个属性**

4.至此，我们可以通过`slot props(插槽props)`去获取该`field(字段)` 校验不通过时的错误提示:
```
<ValidationProvider v-slot="v">
  <input v-model="value" type="text">
  <span>{{ v.errors[0] }}</span>
</ValidationProvider>
```
---
## 添加校验规则
默认情况下,VeeValidate并不会安装任何校验规则以保证包体尽量的精简

1.`extend`函数  
  1.1 extend(name,function(value)) 接收两个参数,其中name是规则的名称;function(value)是`validator function(校验函数)`
  ```
  import { extend } from 'vee-validate';
  extend('positive', value => {
    return value >= 0;
  });
  ```
  *也可以通过ES6语法中的解构赋值,来简化上面的写法*
  ```
  <ValidationProvider rules="positive" v-slot="{ errors }">
    <input v-model="value" type="text">
    <span>{{ errors[0] }}</span>
  </ValidationProvider>
  ```
  **一般情况下,我们会将添加的rules(规则)放到单独的`validate.js`文件中,然后在入口文件`main.js`中通过`import 'validate'`来加载js文件**
  
  1.2 当想要将多种校验规则同时加到一个input元素时,可以通过` | `管道符去分离这些rules
  ```
  <ValidationProvider rules="positive|odd|prime|fib" v-slot="{ errors }">
    <input v-model="value" type="number">
    <span>{{ errors[0] }}</span>
  </ValidationProvider>
  ```
  
  1.3 extend函数的第二个参数还可以传入一个对象,extend(name,options)
  ```
  import { extend } from 'vee-validate';
  extend('odd', {
    validate: value => {
      return value % 2 !== 0;
    }
  });
  ```

## 关于extend(name,options)中options中的配置项
1.params:[],该数组可以指定多个字段，用于在validate function中从外部传入指定参数，提高规则的复用率
```
import { extend } from 'vee-validate';
extend('minmax', {
  validate(value, args) {
    const length = value.length;
    return length >= args.min && length <= args.max;
  },
  params: ['min', 'max']
});
```
*也可以用ES6语法解构赋值简化↓*
```
extend('minmax', {
  validate(value, { min, max }) {
    return value.length >= min && value.length <= max;
  },
  params: ['min', 'max']
});
```
*在rules属性中,给指定的规则后用`:`来按顺序传递params参数↓*
```
<ValidationProvider rules="minmax:3,8" v-slot="{ errors }">
  <input v-model="value" type="text">
  <span>{{ errors[0] }}</span>
</ValidationProvider>
```






















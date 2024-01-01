# Vee-Validate笔记
## 关于Validation Provider
1.原理:ValidationProvider 通过 **scoped-slots**(作用域插槽) 将校验产生的数据(比如校验不通过的错误信息)注入到**slot props(插槽props)**中，使插槽模板组件`<ValidationProvider></ValidationProvider>`能够读取校验信息.

2.注册一个`Validation Provider`组件:
  1.组件内注册:
  ```
  import { ValidationProvider } from 'vee-validate';
  export default {
    components: {
      ValidationProvider
    }
  };
  ```
  2.全局注册:
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
  1.extend(name,function(value)) 接收两个参数,其中name是规则的名称;function(value)是`validator function(校验函数)`,该函数会被注入一个`value`参数,是对应需要校验的`input`元素的`v-model`或者`v-bind`属性所绑定的值.
  ```
  import { extend } from 'vee-validate';
  extend('positive', value => {
    return value >= 0;
  });
  ```












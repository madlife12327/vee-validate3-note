# Vee-Validate笔记
## 关于Validation Provider
- 实现原理:ValidationProvider 通过 **scoped-slots**(作用域插槽) 将校验产生的数据(比如校验不通过的错误信息)注入到**slot props(插槽props)** 中，使插槽模板组件`<ValidationProvider></ValidationProvider>`能够读取校验信息.

- 1.注册一个`Validation Provider`组件:
  - 组件内注册:
  ```
  import { ValidationProvider } from 'vee-validate';
  export default {
    components: {
      ValidationProvider
    }
  };
  ```  
  - 全局注册:
  ```
  import { ValidationProvider } from 'vee-validate';
  Vue.component('ValidationProvider', ValidationProvider);
  ```
- 2.注册组件完成后,就可以在template模板中使用`ValidationProvide`组件了。一般来说，需要用`ValidationProvider`元素包裹住你将要校验的`input`元素:
  ```
  <ValidationProvider v-slot="v">
    <input type="text" v-model="value">
  </ValidationProvider>
  ```
  **注意，需要校验的`input`元素务必要用`v-model`或者`v-bind`绑定一个响应式数据，用于提示`ValidationProvider`将要校验`VueComponent(组件实例)`身上的哪个属性**

- 3.至此，我们可以通过`slot props(插槽props)`去获取该`field(字段)` 校验不通过时的错误提示:
  ```
  <ValidationProvider v-slot="v">
    <input v-model="value" type="text">
    <span>{{ v.errors[0] }}</span>
  </ValidationProvider>
  ```
---
## 添加校验规则
默认情况下,VeeValidate并不会安装任何校验规则以保证包体尽量的精简

- 1.`extend`函数  
  - extend(name,function(value)) 接收两个参数,其中name是规则的名称;function(value)是`validator function(校验函数)`
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
  
  - 当想要将多种校验规则同时加到一个input元素时,可以通过` | `管道符去分离这些rules
    ```
    <ValidationProvider rules="positive|odd|prime|fib" v-slot="{ errors }">
      <input v-model="value" type="number">
      <span>{{ errors[0] }}</span>
    </ValidationProvider>
    ```
  
  - extend函数的第二个参数还可以传入一个对象,extend(name,options)
    ```
    import { extend } from 'vee-validate';
    extend('odd', {
      validate: value => {
        return value % 2 !== 0;
      }
    });
    ```

## 关于extend(name,options)中options中的配置项
- 1.params:[],该数组可以指定params参数字段,使用该规则时可以在`rules`属性上通过` : ` 向`validate function`传递参数,比如`rules="max:3"`,提高规则的复用率
  - 定义规则:
    ```
    extend('minmax', {
      validate(value, { min, max }) {
        return value.length >= min && value.length <= max;
      },
      params: ['min', 'max']
    });
    ```
  - 使用规则:
    ```
    <ValidationProvider rules="minmax:3,8" v-slot="{ errors }">
      <input v-model="value" type="text">
      <span>{{ errors[0] }}</span>
    </ValidationProvider>
    ```
    *在rules属性中,给指定的规则后用`:`来按顺序传递params参数*
  
- 2.有时候我们需要验证某个字段是否在一个集合中,这时候就可以考虑使用 **one_of** 规则:
  - 定义规则:
    ```
    import { extend } from 'vee-validate';
    extend('one_of', (value, values) => {
      return values.indexOf(value) !== -1;
    });
    ```
  - 使用规则:
    ```
    <ValidationProvider rules="one_of:1,2,3,4,5,6,7,8,9" v-slot="{ errors }">
    <input v-model="value" type="text">
    <span>{{ errors[0] }}</span>
    </ValidationProvider>
    ```

## 错误信息message
- VeeValidate 会给要验证的字段生成错误信息,`This field is invalid` 是所有规则默认返回的错误信息.
- 1.修改规则的默认错误信息
  - 通过修改`validation function`返回值来修改默认错误信息
    ```
    import { extend } from 'vee-validate';
    extend('positive', value => {
      if (value >= 0) {
        return true;
      }
      return 'This field must be a positive number';
      });
    ```
  - 通过在options配置项中添加`message`属性来修改默认信息(常用)
    ```
    extend('odd', {
      validate: value => {
        return value % 2 !== 0;
      },
      message:'This field must be an odd number'
    });
    ```
- 2.有时候我们需要在使用规则时再决定错误信息的内容,这时候可以考虑使用vee-validate提供的简单插值语法`{_field_}`来修改错误信息内容
  - 定义规则:
    ```
    extend('odd', {
    validate: value => {
      return value % 2 !== 0;
    },
    message: 'This {_field_} must be an odd number'
    });
    ```
  - 使用规则:
    ```
    <ValidationProvider name="age" rules="positive" v-slot="{ errors }">
      <input v-model="value" type="text">
      <span>{{ errors[0] }}</span>
    </ValidationProvider>
    ```
    *使用name属性插入到{_field_}中*
    **如果ValidationProvider中没有指定`name`属性,vee-validate就会使用其所包裹的元素中的`name`或`id`属性作为`{_field_}`的值**
- 3.message可以使用params参数作为占位符**placeholders**`{}`的值
    ```
    extend('min', {
      validate(value, { length }) {
        return value.length >= length;
      },
      params: ['length'],
      message: 'The {_field_} field must have at least {length} characters'
    });
    ```
- 4.将message写成函数
  - 第一个参数 fieldName,该参数是ValidationProvider的name属性的值
  - 第二个参数 placeholders,占位符,该参数是一个对象,包含params参数以及`_field_`,`_value_`,`_rule_`其三分别表示 字段名，已验证的字段值，触发此消息的规则名
    ```
    extend('minmax', {
      validate(value, { min, max }) {
        return value.length >= min && value.length <= max;
      },
      params: ['min', 'max'],
      message: (fieldName, placeholders) => {
        return `The ${fieldName} field must have at least ${placeholders.min} characters and ${placeholders.max} characters at most`;
    }
    });
    ```
- 5.关于mutiple message(多消息)
  - 你可能想知道为什么`errors`是一个数组而不是一个字符串,这是因为虽然每条规则都只会生成一条`message`(消息),但一个`input`也可能同时需要多条规则进行校验,vee-validate就会把所有`rules`产生的`message`都存到`errors`数组中













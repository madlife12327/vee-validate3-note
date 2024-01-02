# Basics(基础知识)
## Validation Provider
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
## Registering the Validation Provider(添加校验规则)
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

## Rule Arguments(规则参数)
- 1.params参数,该参数是一个数组,可以指定外部传的params参数字段,使用规则时可以在`rules`属性上通过` : ` 向`validate function`传递参数,比如`rules="max:3"`,提高规则的复用率
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

## Messages(消息)
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

# Available Rules(可用的规则)
- 默认情况下，vee-validate并不会安装这些可用规则,需要自行安装
- 1.Importing The Rules(按需引入)
  - Validation rules(验证规则)都会在`vee-validate/dist/rules`文件中暴露
    - 定义规则:
      ```
      import { extend } from 'vee-validate';
      import { required, email } from 'vee-validate/dist/rules';
      
      // No message specified.
      extend('email', email);
      
      // Override the default message.
      extend('required', {
        ...required,
        message: 'This field is required'
      });
      ```
    - 使用规则:
      ```
      <ValidationProvider name="email" rules="required|email" v-slot="{ errors }">
        <input v-model="email" type="text">
        <span>{{ errors[0] }}</span>
      </ValidationProvider>
      ```
- 2.Installing All Rules(安装所有规则) **不推荐**
  - 2.1 `import * as rules from 'vee-validate/dist/rules'`导入所有规则后,通过`Object.keys`遍历出所有规则名,最后用`extend`定义所有vee-validate提供的默认规则
    ```
    import { extend } from 'vee-validate';
    import * as rules from 'vee-validate/dist/rules';

    Object.keys(rules).forEach(rule => {
      extend(rule, rules[rule]);
    });
    ```
  - 2.2 另一种方法是导入vee-validate的完整包,其包含所有校验规则和英文消息':  
      `import { ValidationProvider } from 'vee-validate/dist/vee-validate.full.esm';`

- 3.Rules(规则)
    - 各种可用规则的详情校验规则转至[Rules](https://vee-validate.logaretm.com/v3/guide/rules.html#rules)
    - 这里仅介绍5个常用的可用规则:
      - regex (验证字段必须匹配正则表达式)
        ```
        <ValidationProvider :rules="{ regex: /^[0-9]+$/ }" v-slot="{ errors }">
          <input type="text" v-model="value">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
        ```
      - between (验证字段必须在一个区间内)
        ```
        <ValidationProvider rules="between:1,11" v-slot="{ errors }">
          <input v-model="value" type="text">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
        ```
      - comfirmed (验证字段必须与确认字段相同)
        ```
        <ValidationProvider v-slot="{ errors }" vid="confirmation">
          <input v-model="confirmation" type="text">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
        
        <ValidationProvider rules="confirmed:confirmation" v-slot="{ errors }">
          <input v-model="value" type="text">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
        ```
        ![params](https://github.com/rjj1013404970/VeeValidateNote/blob/main/images/params.png)
      - is (验证下的字段必须匹配给定的值,使用严格的相等)
        ```
        <ValidationProvider rules="is:hello" v-slot="{ errors }">
          <input type="text" v-model="value">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
        ```
      - required (验证的字段必须非空.默认情况下,空字符串、空数组、`null`、`undefinded`都属于空值)
        ```
        <ValidationProvider rules="required" v-slot="{ errors }">
          <input type="text" v-model="value">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
        ```
# Validation State(校验状态)
- 在之前的示例中,我们只用过`slot props`中的`errors`来显示错误消息,但`errors`其实只是`Validation State`中的一部分.
- Flags(标记)
  - Validation Flags(校验标记) 是一个`boolean`值,用于获取校验字段的状态信息,以下是**vee-validate**官方文档罗列出所有可访问的`flags`:
    ![flags](https://github.com/rjj1013404970/VeeValidateNote/blob/main/images/flags.png)

# Handling Forms(处理表单)
- 到这里,我们已经学习了利用`ValidationProvider`去校验`input`元素中的字段是否通过某规则,但整体上我们并没有学习如何校验表单元素.
- 对于表单元素`form` ,我们可以使用`ValidationObserver`组件去校验表单.
- 我们可以把`ValidationProvider`组件当作一个单独的`field`字段,把`ValidationObserver`当作`form`表单,那么`ValidationObserver`就是每个子组件`ValidationProvider`的聚合器(aggregator)或者说领队(leader),这个聚合器会暴露所有字段的校验状态(Validation State)
- 基本示例
  - 假如你有一个基本的`form`表单,想在表单其他字段校验通过之前,将提交按钮disable(禁止),你可以使用`ValidationObserver`中暴露给`slot props`的`Validation State`中的`invalid`标记.
    ```
     <ValidationObserver v-slot="{ invalid }">
        <form @submit.prevent="onSubmit">
          <ValidationProvider name="E-mail" rules="required|email" v-slot="{ errors }">
            <input v-model="email" type="email">
            <span>{{ errors[0] }}</span>
          </ValidationProvider>
    
          <ValidationProvider name="First Name" rules="required|alpha" v-slot="{ errors }">
            <input v-model="firstName" type="text">
            <span>{{ errors[0] }}</span>
          </ValidationProvider>
    
          <ValidationProvider name="Last Name" rules="required|alpha" v-slot="{ errors }">
            <input v-model="lastName" type="text">
            <span>{{ errors[0] }}</span>
          </ValidationProvider>
    
          <button type="submit" :disabled="invalid">Submit</button>
        </form>
      </ValidationObserver>
    ```
    *当所有input元素中的`invalid`标记都为`false`时,ValidationObserver的`invalid`才会为`false`*
  
- Validate Before Submit(提交前验证)
  - `ValidationObserver` 提供了`handleSubmit`函数,这个函数需要我们传入一个回调函数,这个回调会在用户提交有效表单的时候调用.
    ```
    <ValidationObserver v-slot="{ handleSubmit }">
      <!--用户提交有效表单时才会调用onSubmit-->
      <form @submit.prevent="handleSubmit(onSubmit)">
        <ValidationProvider name="E-mail" rules="required|email" v-slot="{ errors }">
          <input v-model="email" type="email">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
  
        <ValidationProvider name="First Name" rules="required|alpha" v-slot="{ errors }">
          <input v-model="firstName" type="text">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
  
        <ValidationProvider name="Last Name" rules="required|alpha" v-slot="{ errors }">
          <input v-model="lastName" type="text">
          <span>{{ errors[0] }}</span>
        </ValidationProvider>
  
        <button type="submit">Submit</button>
      </form>
    </ValidationObserver>
    ```
- Resetting Forms(重置表单)







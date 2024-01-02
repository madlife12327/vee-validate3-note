# vee-validate3 笔记 
- [vee-validate官方文档](https://vee-validate.logaretm.com/v3/)
## Validation Provider
- 实现原理:ValidationProvider 通过 **scoped-slots**(作用域插槽) 将包裹的`input`元素的校验状态注入到**slot props(插槽props)** 中，使`ValidationProvide`组件能够读取到`input`的校验状态.

- 注册一个`Validation Provider`组件:
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
- 注册组件完成后,就可以在template模板中使用`ValidationProvide`组件了。一般来说，需要用`ValidationProvider`元素包裹住你将要校验的`input`元素:
  ```
  <ValidationProvider v-slot="v">
    <input type="text" v-model="value">
  </ValidationProvider>
  ```
  **注意，需要校验的`input`元素务必要用`v-model`或者`v-bind`绑定一个响应式数据，用于提示`ValidationProvider`将要校验`VueComponent(组件实例)`身上的哪个属性**

- 至此，我们可以通过`slot props(插槽props)`去获取该`field(字段)` 校验不通过时的错误提示:
  ```
  <ValidationProvider v-slot="v">
    <input v-model="value" type="text">
    <span>{{ v.errors[0] }}</span>
  </ValidationProvider>
  ```
---
## Registering the Validation Provider(添加校验规则)
默认情况下,VeeValidate并不会安装任何校验规则以保证包体尽量的精简

- `extend`函数  
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
- params参数,该参数是一个数组,可以指定外部传的params参数字段,使用规则时可以在`rules`属性上通过` : ` 向`validate function`传递参数,比如`rules="max:3"`,提高规则的复用率
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
  
- 有时候我们需要验证某个字段是否在一个集合中,这时候就可以考虑使用 **one_of** 规则:
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
- 修改规则的默认错误信息
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
- 有时候我们需要在使用规则时再决定错误信息的内容,这时候可以考虑使用vee-validate提供的简单插值语法`{_field_}`来修改错误信息内容
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
- message可以使用params参数作为占位符**placeholders**`{}`的值
    ```
    extend('min', {
      validate(value, { length }) {
        return value.length >= length;
      },
      params: ['length'],
      message: 'The {_field_} field must have at least {length} characters'
    });
    ```
- 将message写成函数
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
- 关于mutiple message(多消息)
  - 你可能想知道为什么`errors`是一个数组而不是一个字符串,这是因为虽然每条规则都只会生成一条`message`(消息),但一个`input`也可能同时需要多条规则进行校验,vee-validate就会把所有`rules`产生的`message`都存到`errors`数组中

# Available Rules(可用的规则)
- 默认情况下，vee-validate并不会安装这些可用规则,需要自行安装
- Importing The Rules(按需引入)
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
- Installing All Rules(安装所有规则) **不推荐**
  - `import * as rules from 'vee-validate/dist/rules'`导入所有规则后,通过`Object.keys`遍历出所有规则名,最后用`extend`定义所有vee-validate提供的默认规则
    ```
    import { extend } from 'vee-validate';
    import * as rules from 'vee-validate/dist/rules';

    Object.keys(rules).forEach(rule => {
      extend(rule, rules[rule]);
    });
    ```
  - 另一种方法是导入vee-validate的完整包,其包含所有校验规则和英文消息':  
      `import { ValidationProvider } from 'vee-validate/dist/vee-validate.full.esm';`

- Rules(规则)
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
  - 在成功提交表单后,如果你想重置表单,可以使用`ValidationObserver`提供的`reset`函数,该函数会将`Validation State`校验状态重置到初始状态(包括错误信息**errors**):
    ```
    <!--使用ES6语法从slot props中解构出handlSubmit和reset函数-->
    <ValidationObserver v-slot="{ handleSubmit, reset }">
      <form @submit.prevent="handleSubmit(onSubmit)" @reset.prevent="reset">
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
        <button type="reset">Reset</button>
      </form>
    </ValidationObserver>
    ```
    *使用`ValidationObserver提供的`reset函数对校验状态进行重置*

- Programmatic Access with $refs(使用$refs进行编程式访问,简单来说就是使用`vue`中的$ref获取`Dom`元素从而在js中也能访问校验信息)
  ```
   <ValidationObserver ref="form">
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

      <button type="submit">Submit</button>
    </form>
    </ValidationObserver>

  <script>
    export default {
      data: () => ({
        firstName: '',
        lastName: '',
        email: ''
      }),
      methods: {
        onSubmit () {
          this.$refs.form.validate().then(success => {
            if (!success) {
              return;
            }
    
            alert('Form has been submitted!');
    
            // Resetting Values
            this.firstName = this.lastName = this.email = '';
    
            // Wait until the models are updated in the UI
            this.$nextTick(() => {
              this.$refs.form.reset();
            });
          });
        }
      }
    };
  </script>
  ```
  *通过$refs获取dom元素,其元素会被注入validate函数,使用函数对form表单进行校验,关于validate函数会在后面API详细说明*

- Initial State Validation(初始状态校验)
    - `ValidationProvider`默认初始状态不会校验字段,可以使用`immediate`属性在渲染时立刻对字段进行校验:
      ```
      <ValidationProvider rules="required" immediate v-slot="{ errors }">
        <!-- -->
      </ValidationProvider>
      ```
      *渲染时立刻对字段进行校验*

- Persisting Provider State(ValidationProvider组件校验状态持久化)
  - 首先我们要知道,`ValidationObserver`只能监测到当前已经渲染的`ValidationProvider`的组件实例,任何被`v-if`隐藏或者`v-for`移除的`Provider`会排除在`Observer`的校验状态之外.简而言之就是不会参与到表单的验证.
  - 如果希望`Observer`保存已销毁的`provider`校验状态,比如该表单的填写需要路由跳转时,需要保存`provider`的校验状态,但我们知道,默认情况下的路由跳转,会销毁路由跳转前的组件导致校验状态丢失,这时候我们就需要用到`vue`中的`keep-alive`标签以保证第一次创建`provider`组件时将组件实例缓存下来.(有关[keep-alive](https://v2.cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%9C%A8%E5%8A%A8%E6%80%81%E7%BB%84%E4%BB%B6%E4%B8%8A%E4%BD%BF%E7%94%A8-keep-alive))
    ```
    <ValidationObserver ref="form">
      <form @submit.prevent="onSubmit">
        <!--fieldset是嵌套Observer的写法-->
        <fieldset>
          <legend >Step {{ currentStep }}</legend>
          <keep-alive>
            <ValidationProvider v-if="currentStep === 1" name="email" rules="required|email" v-slot="{ errors }">
              <input v-model="email" type="text" placeholder="Your email">
              <span>{{ errors[0] }}</span>
            </ValidationProvider>
          </keep-alive>
    
            <keep-alive>
              <ValidationProvider v-if="currentStep === 2" name="first name" rules="required" v-slot="{ errors }">
                <input v-model="fname" type="text" placeholder="Your first name">
                <span>{{ errors[0] }}</span>
              </ValidationProvider>
            </keep-alive>
            <keep-alive>
              <ValidationProvider v-if="currentStep === 2" name="last name" rules="required" v-slot="{ errors }">
                <input v-model="lname" type="text" placeholder="Your last name">
                <span>{{ errors[0] }}</span>
              </ValidationProvider>
            </keep-alive>
    
            <keep-alive>
              <ValidationProvider v-if="currentStep === 3" name="address" rules="required|min:5" v-slot="{ errors }">
                <textarea v-model="address" type="text" placeholder="Your address"></textarea>
                <span>{{ errors[0] }}</span>
              </ValidationProvider>
            </keep-alive>
        </fieldset>
    
        <button type="button" @click="goToStep(currentStep - 1)">Previous</button>
        <button type="button" @click="goToStep(currentStep + 1)">{{ currentStep === 3 ? 'Submit' : 'Next' }}</button>
      </form>
    </ValidationObserver>
    ```
    *使用Keep-alive标签缓存组件,保存`provider`校验状态*
    **注意,即使字段被隐藏/卸载，只要它们被keep-alive包裹，它们的状态也会受到validate和reset调用的影响.**

# Localization(本地化) **常用**
- vee-validate 内置了校验信息的支持,本地化配置是可选的,且默认不配置本地化.
- Using the default i18n(使用默认的i18n)
  - vee-validate 配备了小型的`i18n`词典已应对`i18n`的基本需求.
  - Adding messages(添加校验消息)
    - vee-validate默认语言是`en`,首先需要导入`localize`函数,`localize`函数需要传入一个配置对象,你可以像这样添加校验消息:
      ```
      import { localize } from 'vee-validate';

      localize({
        en: {
          messages: {
            required: 'this field is required',
            min: 'this field must have no less than {length} characters',
            max: (_, { length }) => `this field must have no more than ${length} characters`
          }
        }
      });
      ```
      *message对象的Key值是规则的名称,Value值是校验消息的内容,可以是字符串、模板字符串或者消息生成函数*  
- Installing locales(安装本地化)
  - 本地化资源默认只有`en`,我们可以按需引入本地化资源,其默认的校验消息也已自动本地化:
    ```
    import { localize } from 'vee-validate';
    import en from 'vee-validate/dist/locale/en';
    import zh_CN from 'vee-validate/dist/locale/zh_CN';
    
    // Install English and Chinese locales.
    localize({
      en,
      zh_CN
    });
    ```
- Setting the locale(设置本地化环境)
  - 你可以同时激活和添加新校验消息:
    ```
    import { localize } from 'vee-validate';
    import zh_CN from 'vee-validate/dist/locale/zh_CN';
    
    // Install and Activate the Chinese locale.
    localize('zh_CN', zh_CN);
    ```
- Localized field names(本地化字段的name属性)
  - 首先我们知道,`form`表单中,`form`元素的子元素的name值会在`form`触发`Submit`事件中,作为`queryString`的**Key**值发往`action`属性值的地址.
  - 有时候我们希望在表单校验通过后,对`name`属性值做一个本地化映射,比如`name="email"`在校验通过后希望映射为`name="邮箱"`发请求给服务器,这时候我们就需要给`locale`的配置对象添加`names`属性进行配置:
    ```
    import { localize } from 'vee-validate';

    localize({
      zh_CN: {
        names: {
          email: '邮箱',
        }
      }
    })
    ```
    *如此配置后,当`input`使用`name="email"`且校验通过时,就会被vee-validate替换成本地化的`name="邮箱"`发往服务器*

- Custom messages per field(给每个字段自定义校验消息)
  - 我们可以通过给`locale`对象添加`field`属性来给每个规则的指定字段提供一个自定义消息:
    ```
    import { localize } from 'vee-validate';

    localize({
      en: {
        messages: {
          // generic rule messages...
        },
        fields: {
          password: {
            required: 'Password cannot be empty!',
            max: 'Are you really going to remember that?',
            min: 'Too few, you want to get doxed?'
          }
        }
      }
    });
    ```
    *修改`en`环境下的`password`字段的`required`、`max`、`min`规则的校验消息*

- Lazily importing locales(本地化资源懒加载)
  - 利用*ES2020*的动态导入语法`import()`,实现资源的懒加载:
    ```
    import { localize } from 'vee-validate';

    function loadLocale(code) {
      return import(`vee-validate/dist/locale/${code}.json`).then(locale => {
        localize(code, locale);
      });
    }
    ```
    *`import()`会返回一个promise对象,我们可以指定回调实现注册本地化资源的功能,当我们执行loadLocale()时才会加载本地化资源模块*

# Interaction and UX(交互和用户体验)
  - vee-validate对于何时进行校验有多种策略,最常见的有以下四种:
    - Aggressive : 当用户在`input`框中一输入值就触发校验.
    - Passive : 当表单提交时才触发校验.
    - Lazy : 当`input`框失去焦点或者改变焦点时触发校验.
    - Eager : 该策略结合了`Aggressive`和`Lazy`.当`input`框失去焦点时会触发第一次校验,第一次校验如果不通过后续校验就会表现出`Aggressive`策略,直到校验合法了才会表现出`Lazy`策略.

  - Interaction Modes(交互模式)
    - vee-validate 提供了多种常见的校验策略,我们称这些校验策略称之为`interaction modes`(交互模式).
    - Setting The Interaction Mode Globally(设置全局交互模式)
      ```
      import { setInteractionMode } from 'vee-validate';

      setInteractionMode('lazy');
      ```
    - Setting The Interaction Mode Per Component(设置单个组件的交互模式)
      ```
      <ValidationProvider mode="lazy" rules="required" v-slot="{ errors }">
        <!-- Some input -->
      </ValidationProvider>
      ```
      *使用mode属性将交互模式设置为`lazy`模式*
**至此,你已经掌握了vee-validate的基本使用方法,如果你想了解更多关于`vee-validate`的知识,务必阅读以下对核心API的介绍以及查阅[vee-validate官方文档](https://vee-validate.logaretm.com/v3/)**

# Core Validation API(核心校验API)
## The Validate Function
- 你可以从vee-validate中导入`validate`函数,用`ValidationProvider`相同的方式直接对字段进行校验:
  ```
  import { validate } from 'vee-validate';

  validate('somval', 'required|min:3').then(result => {
    if (result.valid) {
      // Do something
    }
  });
  ```
  *validate函数继承Promise类,且该函数会返回一个关于校验结果的对象*
- `validate`的函数签名大概如下:
  ```
  interface Result {
    valid: boolean;
    errors: string[];
    failedRules: {
      [x: string]: string;
    };
  }
  
  interface validate {
    (value: any, rules: string | Record<string, any>,  options?: ValidationOptions): Result;
  }
  ```
  - 可以看到`result`对象接口中有三个常用属性值:
    - valid : 校验通过为`true`,否则为`falsa`
    - errors : 校验失败时的错误消息列表
    - failedRules : 校验失败对象,其中key为规则名,value为错误消息

  - validation(rules,value,options?),该函数需要传入三个参数:
    - value : 要校验的值
    - rules : 校验规则
    - options : ValidationOptions配置项

- Validation Options(校验配置项)
  - `ValidationOption`接口如下:
    ```
    interface ValidationOptions {
      name?: string;
      values?: { [k: string]: any };
      names?: { [k: string]: string };
      bails?: boolean;
      skipOptional:? boolean;
      isInitial?: boolean;
      customMessages?: { [k: string]: string };
    }
    ```
   - |Property|Type|Description|
     |:---------------:|:---------------:|:-------------------------:|
     |name|string|要校验的字段名|
     |values|[x: string]: string|其他字段的值(用于跨域校验)|
     |names|{ { [k: string]: string }|其他字段名(用于跨域校验)|
     |bails|boolean|如果为true,校验将会在第一个校验失败的规则停下(用于多规则校验)|
     |skipOptional|boolean|如果为true,当值为空时,会跳过可选字段的校验|
     |isInitial|boolean|如果为true,会在第一次尝试校验时,跳过所有规则|
     |customMessages|{ [k: string]: string }|自定义错误消息,key是规则名|

- 如果你只需要简单的校验,可以直接使用validate函数:
  ```
  let password = 'my password';
  let confirmation = '????';
  
  validate(password, 'required|confirmed:@confirmation', {
    name: 'Password',
    values: {
      confirmation
    }
  }).then(result => {
    if (result.valid) {
      // Do something!
    }
  });
  ```






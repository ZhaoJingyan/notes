# vue 学习笔记 #

Vue的详细教程请进入[官网](https://cn.vuejs.org/)

## 组件 ##

以下代码是一个组件最基础的结构：

```vue
<template>
    <!-- 必须有一个根标签 -->
    <div>
        <child type="type" />
    </div>
</template>

<script>
import Child from '@/component/Child' // @表示源码目录

/**
 * 组件必须以default形式导出
 */
export default {
    /**
     * 组件名称，可以不指定
     */
    name: 'TestComponent',

    /**
     * 使用的子组件，
     */    
    components: {Child, Child2: import('@/component/Child2')}

    /**
     * 组件属性
     */
    props: ['date', 'description'],

    /**
     * 组件数据，必须以function的返回值声明。
     */
    data: function(){
        return {
            index: 1,
            type: 'NOTE'
        };
    },
    
    /**
     * 计算属性
     */
    computed:{
        return `${this.date}`;
    },
    
    /**
     * 方法
     */
    methods:{
        test(){
            this.$emit('display', this.type);
        }
    },

    /**
     * 监控属性、数据变化
     */
    watch:{
        description(newValue,oldValue){
            // todo Something;
        }
    }
}
</script>

<style>
div{
    /* CSS code */
}
</style>
```

## 属性 ##

props有很多定义方法，可以定义一个数组：

```javascript
props: ['pro1', 'pro2', 'pro3']
```

 也可以通过使用类定义:

```javascript
props: {
    pro1: Number,
    pro2: String,
    pro3: Boolean,
    pro4: Function
}
```

也可以采用更详细的定义：

```javascript
props:{
    pro1: {
        type: Number,
        default: 100
    },
    pro2: {
        type: [Number, String],
        default: function(){
            return {test:'Hello'}
        }
    },
    pro3: {
        type: Array,
        required: true,
        validator: function (value) {
            // 返回布尔值
            return true;
        }
    }
}
```

## vue-cli3 ## 

## vuex ##

## vue with typescript ##

## 单元测试 ##

一般使用mocha单元测试框架

```javascript
let add = require('./add.js');
let expect = require('chai').expect;

/**
 * 测试模块
 */
describe('加法函数的测试', function() {
  /**
   * 测试用例
   */
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
});
```

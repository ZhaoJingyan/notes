# vue 学习笔记 #

Vue的详细教程请进入[官网]()

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

## vue-cli3 ## 
s
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

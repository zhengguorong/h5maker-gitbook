## 为什么要重构

学习新技术或者提高架构能力，最好从实际项目中实践总结，最近在学习UX设计和单元测试，系统重构该系统，把学习的相关知识应用到系统里，同时也验证构建一个体验良好的系统和高覆盖率的系统，需要增加多少成本，如果在企业里应用，到底值不值得，下面为需要优化的事项。

1. 引入新的技术栈

2. 优化UI和UX

3. 修复错误的架构设计

4. 单元测试全面覆盖

### 更新技术栈

##### 前端技术栈

**vue：**对vue比较熟，有过几个大型项目的使用经验，其实选择react和vue个人感觉没什么差别。

**vuex：**对于复杂的前端应用，一个设计合理的数据管理是非常重要的，同时方便进行单元测试。

**typescript：**主要尝试使用强类型，看能否提升整体代码质量。

##### 后端技术栈

**egg.js：**老系统是用express写的，但是由于自己对后端不太精通，导致编写代码过于松散，所以这次使用eggjs来规范代码。

**mongodb：**因为数据不是很复杂，mongodb做数据存储足够了。

**jwt：**实现用户登录状态控制。

### 修正设计失误

由于对技术的不熟悉，边开发项目边熟悉技术，难免会出现设计上的不合理，通过重构，修正以下问题。

##### 1、RESTful API设计不合理

RESTful API风格比较适合资源的增删改查，但是我们系统经常一些请求是非资源性的，例如用户登录，如果用RESTful设计，应该是put请求，对user这个资源进行状态更新操作，这样子无疑增加了理解的复杂度。所以对于这类偏动作型的接口，我更希望定义为一个比较好理解的/auth/login，前端在看到接口命名的时候，就大概知道接口的作用。而且在项目里，两种风格的混用是很不好的，特别如果多人协同开发的时候，容易导致项目混乱。

##### 2、vuex使用过于麻烦

vuex的标准流程中，对于提交数据修改都会定义一个mutation-type管理，我认为这里是可以省略的，因为vuex提供了namespace功能，已经可以很好的防治冲突了。

action，getter,mutation分开三个文件定义，在修改一个逻辑的时候，切换文件容易把我绕晕，所以我偏向三个文件合并到一个文件。

vuex使用原则：**简单、直观**

```js
export default {
  namespaced: true,
  state: {
    formList: []  // 问卷列表
  },
  actions: {
    // 获取问卷列表
    getFormList ({commit}) {
      api.getFormList().then(res => {
        commit('setFormList', res)
      })
    }
  },
  getters: {
      // 因为夫级已经是getter了，在定义名字的时候就没必要定义为getFormList
    formList (state) {
      return state.formList
    }
  },
  mutations: {
    // 因为mutations定义数据的修改，有可能是更新，或者设置新的值，所以这里加入前缀set/update/delete之类，直观了解该方法的含义。
    setFormList (state, list) {
      state.formList = list
    }
}
```

##### 3、随意修改对象数据

我们也许有听过不可变数据或者immutable.js，以前我也没体会到为何不可变数据有这么重要，直到我开发逻辑复杂的应用时，就深有体会了。如果函数随意改变对象的值，容易造成其他应用该对象的地方产生不可控的后果。

##### 4、组件定义不合理

需要实现父子之间的通讯，可以夫组件传递参数给子组件，也可以通过子组件通过事件广播给夫组件，但是系统中存在两种设计混用，会造成理解成本增加。


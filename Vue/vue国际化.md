# 前端国际化

## 需求

1. 公司sass系统需要做一个按钮切换中英文，按钮的样式或者icon也需要做改变。
2. 项目文案需要做中英文切换或者其他语言

## 思路

一般sass项目有比较多的定制化配置。于是在项目初始化完成后，请求后端接口，得到当前门户网站的配置项（包括语言、token、用户唯一标识、身份等等）。然后保存到缓存中，图片或者样式需要改变则直接从缓存取。文案使用`vueI18n`。

## 应用

### 目录

```js
project_name
    - src
    	- locales
			- lang.js
			- zh-CN.js
			- en-US.js
			- xx-xx.js
	index.js
```

### 代码

```js
// zh-CN.js
const cn = {
    a: '登录',
    b: '注册'
}
export default cn
// en-US同理。
```



```js
// lang.js
import Vue from 'vue';
import VueI18n from 'vue-i18n';
import enUS from './en-US';
import zhCN from './zh-CN';
Vue.use(VueI18n);

const messages = {
    enUS: { message: enUS },
    zhCN: { message: zhCN },
};
export default new VueI18n({
    locale: $localStore.get('lang') || 'zhCN',
    messages
});
```



```js
// index.js
import i18n from '@/locales';
new Vue({
    store,
    router,
    i18n,
    render: h => h(App)
}).$mount('#app');
```

### 使用

```vue
<template>
	<div>
        <div>
          {{ $t('message.a') }}
    	</div>
    </div>
</template>
```

### vue-i18n

[START]('https://kazupon.github.io/vue-i18n/zh/started.html')


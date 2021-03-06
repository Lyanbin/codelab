# 4. SPA 的 Skeleton

关于骨架屏 (Skeleton) 的概念可以参考 Lavas 的知乎专栏的[为 Vue 项目添加骨架屏](https://zhuanlan.zhihu.com/p/28465598)一文，也可以参考 Lavas 文档的 [Skeleton 和 App Shell 模型](/guide/v2/advanced/appshell)。这里会列出 SPA 模式下最基本的 Skeleton 配置方法。

## 编写 Skeleton

如果是通过 `lavas init` 命令初始化项目的，开发者可以在 `/core/Skeleton.vue` 位置看到骨架屏本身的样式。

```
<template>
    <div class="skeleton-wrapper">
        <!-- skeleton content -->
    </div>
</template>

<script>
export default {
    name: 'skeleton'
};
</script>

<style lang="stylus" scoped>
// skeleton content style rules
</style>

```

Skeleton.vue 按照普通 Vue 页面组件的开发方式进行开发即可。一般情况我们会使用一些图片(尤其最好是 base64 编码直接写入，避免再进行网络请求)来充当 Skeleton 的内容，替代加载时的白屏。

和 SSR 的 App Shell 不同的是，Skeleton 的内容和样式都必须在这里编写。

## Service Worker 配置

通过 webpack 的构建，Skeleton.vue 的内容会直接编译到 `/dist/index.html` 中。因此针对 Service Worker 无需做什么特别的配置。

尤其需要注意的是，SSR 模式使用的 `appshellUrl` __不能使用__，否则会导致 Service Worker 尝试访问这个地址并缓存，但实际上这个地址并不能访问。

## Service Worker 模板

Service Worker 模板位于 `/core/service-worker.js`，在这里我们将注册 Skeleton 的使用方式，其余 Service Worker 模板的固定内容就不再重复了。

```javascript
// Define response for HTML request.
workboxSW.router.registerNavigationRoute('/index.html');
```

和 App Shell 相关的只有 `registerNavigationRoute` 这一句。这句的作用是将所有 HTML 请求 (`request.mode === 'navigate'`)由 Service Worker 进行拦截，使用参数中注册的 `'/index.html'` 进行返回。

经过这些配置，我们的应用已经实现了 Skeleton。在首次被访问时，会将 `index.html` 缓存起来，并在后续访问时直接从缓存取出并返回，大大提升了响应速度，提供更顺滑的加载体验！

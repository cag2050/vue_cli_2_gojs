<template>
    <div id="app">
        <div>路由列表：</div>
        <router-link to="/" style="margin-right: 10px;">workflow demo(输入框只有1个)</router-link>
        <router-link to="/workflow2" style="margin-right: 10px;">workflow demo2(输入框有2个)</router-link>
        <router-link to="/tree_layout" style="margin-right: 10px;">TreeLayout demo</router-link>
        <router-link to="/hello" style="margin-right: 10px;">hello</router-link>
        <div>======下面是内容=======</div>
        <router-view/>
    </div>
</template>

<script>
export default {
    name: 'App'
}
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }

</style>

<template>
    <div id="app">
        <div>路由列表：</div>
        <router-link to="/">workflow demo</router-link>
        <router-link to="/tree_layout">TreeLayout demo</router-link>
        <router-link to="/hello">hello</router-link>
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

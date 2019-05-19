<template>
    <div style="flex:1">
        <myTask></myTask>
    </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
    import myTask from "./myTask.vue";
    module.exports = {
        data () {
            return {}
        },
        components: {
          myTask
        }
    }
</script>
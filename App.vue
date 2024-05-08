<template>
    <div id="app">
        <simulator/>
    </div>
</template>

<script>
    import Simulator from './Simulator.vue'

    export default {
        components: {Simulator}
    };
</script>

<style>
    html {
        width: 100%;
        height: 100%;
    }

    body {
        width: 100%;
        height: 100%;
        margin: 0;
    }

    #app {
        width: 100%;
        height: 100%;
        font-size: 14px;
    }
</style>

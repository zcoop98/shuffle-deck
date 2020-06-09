<template>
    <div id="app">
        <card-flip />
    </div>
</template>

<script>
import CardFlip from './components/CardFlip.vue';

export default {
    name: 'App',
    components: {
        CardFlip,
    }
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 12px;
}
</style>

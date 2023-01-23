<template>
    <main>
        <button @click="reset">RESET</button>
        <button @click="play">START</button>
        <button @click="stop">STOP</button>
        <TheGrid ref="theGrid"/>
    </main>
</template>

<script setup>
import TheGrid from '@/components/TheGrid.vue';
import {ref} from 'vue';

const theGrid = ref();

const reset = () => theGrid.value.reset();
const play = () => theGrid.value.play();
const stop = () => theGrid.value.stop();
</script>


<style scoped>
header {
    line-height: 1.5;
}

.logo {
    display: block;
    margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
    header {
        display: flex;
        place-items: center;
        padding-right: calc(var(--section-gap) / 2);
    }

    .logo {
        margin: 0 2rem 0 0;
    }

    header .wrapper {
        display: flex;
        place-items: flex-start;
        flex-wrap: wrap;
    }
}
</style>

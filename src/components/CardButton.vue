<template>
    <button class="btn" :style="{ backgroundColor: props.color}" @click="handleClick">
        {{ props.number }}
    </button>
</template>

<script setup>
import {defineProps, defineEmits} from 'vue';

const props = defineProps({
    number: Number,
    color: String,
})

const emit = defineEmits(["clickBet"])

const handleClick = () => {
    emit("clickBet", props.number, props.color)
}

</script>

<style scoped>
.btn{
    font-size: 30px;
    color: white;
    width: 50px;
    height: 50px;
    padding: 0;
}
</style>
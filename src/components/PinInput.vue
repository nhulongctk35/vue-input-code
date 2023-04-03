<template>
    <input
        ref="inputEl"
        :type="mask ? 'password' : 'text'"
        :class="{ valued: !!modelValue }"
        :value="modelValue"
        maxlength="1"
        autofocus
        @focus="onFocus(true)"
        @blur="onFocus(false)"
        @input="onInput"
        @paste.prevent="onPaste"
        @keydown.backspace="onClear"
        @keydown.arrow-left="onLeft"
        @keydown.arrow-right="onRight"
    >
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue';

const props = defineProps({
    modelValue: String,
    validate: Function,
    mask: Boolean,
    last: Boolean
})
const emits = defineEmits(['update:modelValue', 'paste', 'validate', 'backward', 'forward'])

const inputEl = ref<HTMLInputElement | null>(null)
const isFocus = ref<boolean>(false)

const onFocus = (bool: boolean) => {
    isFocus.value = bool
}

const onInput = (event: Event) => {
    const value = (((event.target as HTMLInputElement).value) ?? '').trim()
    const validated = props.validate?.(value) ?? ''
    if (inputEl.value) {
        inputEl.value.value = validated
    }
    emits('update:modelValue', validated)
    emits('validate', !!validated)
}

const onPaste = (event: ClipboardEvent) => {
    emits('paste', event)
}

let backspaceHit = 0
const onClear = async () => {
    backspaceHit += 1
    await nextTick()
    if (props.last && backspaceHit < 2) {
        return
    }
    if (props.last && backspaceHit >= 2) {
        backspaceHit = 0
    }

    emits('backward')
}

const onLeft = () => emits('backward')
const onRight = () => emits('forward')

defineExpose({
    el: inputEl
})
</script>

<style scoped>
input {
    display: inline-block;
    width: 60px;
    height: 80px;
    border: none;
    border-radius: 5px;
    background-color: #f2f2f2;
    text-align: center;
    font-size: 20px;
    font-weight: bold;
    color: #333;
    box-shadow: inset 0 2px 4px 0 rgb(0 0 0 / 0.05);
    transition: all 0.3s ease-in-out;
    cursor: text;
    z-index: 0;
}

input:hover,
input:focus {
    outline: none;
    background-color: #fff;
    color: #333;
}
input:hover {
    box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
}
input:focus {
    box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
}
/* input:focus ~ input {
    background-color: #d9d9d9;
    color: #999;
} */
input.valued {
    background-color: #fff;
    color: #333;
    box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
}
</style>

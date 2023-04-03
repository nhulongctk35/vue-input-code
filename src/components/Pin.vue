<template>
    <div class="pincode">
        <template v-for="item in length">
            <input ref="inputs" v-model="digits[item]" :type="isSecretMode ? 'password' : 'text'" @input="onInput(item)"
                @paste="onPaste" :maxlength="1" :autofocus="true" />
        </template>
    </div>
</template>
  
<script lang="ts">
import { ref } from 'vue';

export default {
    props: {
        length: {
            type: Number,
            default: 4
        },
        isSecretMode: {
            type: Boolean,
            default: false
        }
    },

    emits: ['complete'],

    setup(props, { emit }) {
        const digits = ref<string[]>(Array(props.length).fill(''));
        const inputs = ref<(HTMLInputElement | null)[]>([]);

        const setRefs = (index: number) => (el: HTMLInputElement | null) => {
            if (el !== null) {
                inputs.value[index] = el;
                return el;
            }
        };

        const onInput = (index: number) => {
            const input = inputs.value[index];
            const value = input?.value ?? '';

            if (/^\d$/.test(value)) {
                digits.value[index] = value;

                if (index < props.length - 1 && digits.value[index] !== '') {
                    const nextInput = inputs.value[index + 1];
                    nextInput?.focus();
                }

                if (index === props.length - 1 && digits.value.every(d => /^\d$/.test(d))) {
                    emit('complete', digits.value.join(''));
                }
            }
        };

        const onPaste = (event: ClipboardEvent) => {
            event.preventDefault();

            const text = event.clipboardData?.getData('text/plain')?.replace(/[^\d]/g, '') ?? '';
            const newDigits = text.slice(0, props.length).split('');

            newDigits.forEach((char, index) => {
                const input = inputs.value[index];

                if (input) {
                    input.value = char;
                    input.dispatchEvent(new Event('input', { bubbles: true }));
                }
            });
        };

        return {
            inputs,
            setRefs,
            digits,
            onInput,
            onPaste
        };
    }
};
</script>
  
<style scoped>
.pincode {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    gap: 12px;
}

.pincode input {
    width: 60px;
    height: 80px;
    border: none;
    border-radius: 5px;
    background-color: #f2f2f2;
    text-align: center;
    font-size: 20px;
    font-weight: bold;
    color: #333;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease-in-out;
}

.pincode input:focus {
    outline: none;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
    background-color: #fff;
    color: #333;
}

.pincode input:focus~input {
    background-color: #d9d9d9;
    color: #999;
}
</style>
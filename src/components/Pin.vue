<template>
    <div class="pincode">
        <PinInput
            v-for="(item, idx) in digits" :key="idx" class="pin"
            ref="pinInputs"
            v-model="digits[idx]"
            :last="idx === digits.length - 1"
            :mask="mask"
            :validate="validateFn"
            @paste="onPaste"
            @validate="onValidate($event, idx)"
            @backward="onBackward(idx)"
            @forward="onForward(idx)"
        />
    </div>
</template>

<script lang="ts">
import { ref, computed } from 'vue'
import PinInput from './PinInput.vue'

export default {
    components: {
        PinInput
    },
    props: {
        length: {
            type: Number,
            default: 4
        },
        tester: {
            type: String,
            default: () => '\d' // eslint-disable-line no-useless-escape
        },
        mask: Boolean
    },

    emits: ['complete'],

    setup(props, { emit }) {
        const digits = ref<string[]>(Array(props.length).fill(''))
        const pinInputs = ref<(typeof PinInput | null)[]>([])
        const inputTester = computed(() => new RegExp(`^\\${props.tester}$`))
        const pasteTester = computed(() => new RegExp(`[^\\${props.tester}]`, 'g'))
        console.log(inputTester.value, pasteTester.value)

        const validateFn = (value: string) => {
            if (inputTester.value.test(value)) {
                return value
            }

            return ''
        }

        const onValidate = async (result: boolean, idx: number) => {
            if (!result) {
                return
            }

            if (pinInputs.value[idx + 1]) {
                (pinInputs.value[idx + 1]?.el as HTMLInputElement)?.focus()
            }

            const value =  digits.value.join('')
            if (value.length >= digits.value.length) {
                setTimeout(() => emit('complete', value))
            }
        }

        const onBackward = (idx: number) => {
            if (pinInputs.value[idx - 1]) {
                (pinInputs.value[idx - 1]?.el as HTMLInputElement)?.focus()
            }
        }

        const onForward = (idx: number) => {
            if (pinInputs.value[idx + 1]) {
                (pinInputs.value[idx + 1]?.el as HTMLInputElement)?.focus()
            }
        }

        const onPaste = (event: ClipboardEvent) => {
            const text = event.clipboardData?.getData('text/plain')?.replace(pasteTester.value, '') ?? ''
            const newDigits = text.slice(0, props.length).split('')

            newDigits.forEach((char, index) => {
                const inputEl = pinInputs.value[index]?.el

                if (inputEl) {
                    inputEl.value = char
                    inputEl.dispatchEvent(new Event('input', { bubbles: true }))
                }
            })
        }

        return {
            pinInputs,
            digits,
            onPaste,
            onBackward,
            onForward,
            onValidate,
            validateFn
        }
    }
}
</script>

<style scoped>
.pincode {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    gap: 12px;
}
</style>

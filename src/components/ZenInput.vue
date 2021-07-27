<template>
    <label class="block">
        <span v-if="label" class="text-gray-700">{{ label }}</span>
        <input class="mt-1 block w-full rounded-md border-gray-300 focus:ring-0"
            :value="modelValue"
            @input="$emit('update:modelValue', $event.target.value)"
            :placeholder="placeholder"
            :type="typeNow"
        >
    </label>
</template>
<script>
export default {
    props: ['label', 'placeholder', 'modelValue', 'type-input'],
    computed: {
        typeNow() {
            return this.typeInput || "text";
        },
    },
    methods: {

    },
}
</script>

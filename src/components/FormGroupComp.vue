<script setup lang="ts">


const model = defineModel<string | undefined | number>()
const props = defineProps<{
    label: string,
    name: string,
    required: boolean,
    type: string,
    icon : string,
    placeholder: string,
    disable?: boolean,
    maxlength?: number
}>()

const handleInput = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (props.type === 'tel') {
        let value = target.value.replace(/\D/g, '')
        if (props.maxlength) {
            value = value.slice(0, props.maxlength)
        }
        target.value = value
        model.value = value
    }
}
</script>

<template>
    <UFormField :label="props.label" :name="props.name" :required="props.required" class=" w-full">
        <UInput :type="props.type" :disabled="props.disable" :icon="props.icon" :placeholder="props.placeholder" v-model="model" size="xl"
            class="w-full" :maxlength="props.maxlength" @input="handleInput" />
    </UFormField>
</template>
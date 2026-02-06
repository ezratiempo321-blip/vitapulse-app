<script setup lang="ts">
import FormGroupComp from '@/components/FormGroupComp.vue';
import errorMessage from '@/components/errorMessage.vue';
import { useAxios } from '@/axios/useAxios.ts';
import * as z from 'zod';
import type { FormSubmitEvent } from '@nuxt/ui';
import { reactive, ref, watch } from 'vue';
import { useRouter } from 'vue-router';
const router = useRouter();

const emit = defineEmits(['change'])
const props = defineProps<{
    id: number
}>();


const schema = z.object({
    password: z.string().min(8)
});

interface errorType {
    response: {
        data: {
            message: string
        }
    }
}

type Schema = z.infer<typeof schema>

const isLoading = ref<boolean>(false)
const errorMess = ref<string | null>(null);
const showPassword = ref<boolean>(false);
const state = reactive<Partial<Schema>>({
   
    password: undefined,
})


const toast = useToast()

const Submit = async (event: FormSubmitEvent<Schema>) => {
    try {
        isLoading.value = true

        await useAxios.post(`/auth/admin/userManagement/delete/${props?.id}`, event.data, {
            headers: {
                'Content-Type': 'application/json'
            }
        });
        toast.add({ title: 'Success', description: 'User deleted successfully', color: 'success' },)
        emit('change')
        router.push({ name: 'users' })
        return

    } catch (error) {
        toast.add({ title: 'Failed', description: 'An error occured, please try again later', color: 'warning' },)
        errorMess.value = (error as errorType)?.response?.data?.message
        console.log(error)
    } finally {
        isLoading.value = false
    }


}
</script>

<template>

    <UForm :schema="schema" :state="state" @submit="Submit" class="w-full flex flex-col gap-3 z-50">
        <errorMessage v-if="errorMess" :message="errorMess" />

        <div class="w-full relative">
            <FormGroupComp label="Password" name="password" required :type="showPassword ? 'text' : 'password'" icon="i-lucide-contact"
                placeholder="Confirm your password" v-model="state.password" />
            <UButton 
                @click="showPassword = !showPassword"
                :icon="showPassword ? 'i-lucide-eye-off' : 'i-lucide-eye'"
                variant="ghost"
                class="absolute right-2 top-8 text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200"
                size="sm"
            />
        </div>

        <div class="flex items-center self-start gap-3 justify-end w-full">
            <div class="self-end">
                <UButton loading-icon="i-lucide-loader-circle" :disabled="isLoading" :loading="isLoading" type="submit"
                    class="self-end md:text-base text-sm bg-red-500  w-full">
                    Confirm delete</UButton>
            </div>
        </div>


    </UForm>

</template>
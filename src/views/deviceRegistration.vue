<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useAxios } from '@/axios/useAxios';
import AdminLayout from '@/layouts/adminLayout.vue';
import { useToast } from '@nuxt/ui/runtime/composables/useToast.js';

const toast = useToast();
const deviceStatus = ref<any>(null);
const loading = ref(false);
const showScanner = ref(false);
const manualDeviceId = ref('');

onMounted(() => {
    checkDeviceStatus();
});

const checkDeviceStatus = async () => {
    try {
        const { data } = await useAxios.get('/auth/device/status');
        deviceStatus.value = data;
    } catch (err) {
        console.error('Failed to check device status:', err);
    }
};

const registerDevice = async (deviceId: string) => {
    loading.value = true;

    try {
        // const { data } = await useAxios.post('/auth/device/register', {
        //     deviceId: deviceId,
        // });
        // toast.add({ title: 'Success', description: 'Device registered successfully!', color: 'success' });
        // showScanner.value = false;

        // await checkDeviceStatus();

        const response = await useAxios.post('/auth/device/register', {
            deviceId: deviceId,
        });

        if (response.status === 201) {
            toast.add({ title: 'Success', description: response.data.message, color: 'success' });
            showScanner.value = false;
            await checkDeviceStatus();
        } else {
            toast.add({ title: 'Error', description: response.response.data.message, color: 'error' });
        }
    } catch (err: any) {
        console.log(err);
        toast.add({ title: 'Error', description: err.response?.data?.error || 'Failed to register device', color: 'error' });
    }
    loading.value = false;
};

const disconnectDevice = async () => {
    loading.value = true;
    try {
        await useAxios.delete('/auth/device/disconnect');
        toast.add({ title: 'Success', description: 'Device disconnected successfully!', color: 'success' });
        await checkDeviceStatus();
    } catch (err) {
        toast.add({ title: 'Error', description: 'Failed to disconnect device', color: 'error' });
    }
    loading.value = false;
};

const handleManualSubmit = () => {
    if (manualDeviceId.value.length === 12) {
        registerDevice(manualDeviceId.value.toUpperCase());
    } else {
        toast.add({ title: 'Invalid', description: 'Device ID must be 12 characters', color: 'warning' });
    }
};
</script>

<template>
    <AdminLayout>
        <h1 class="text-2xl mx-2 font-bold self-start">Device Registration</h1>

        <div class="max-w-2xl mx-auto  px-2">
            <!-- Device Connected -->
            <UCard v-if="deviceStatus?.has_device">
                <div class="flex items-center justify-between flex-wrap gap-4">
                    <div class="flex items-start gap-4">
                        <div class="p-3 bg-green-100 dark:bg-green-900 rounded-full">
                            <UIcon name="i-lucide-wifi" class="size-6 text-green-600 dark:text-green-400" />
                        </div>
                        <div>
                            <p class="font-semibold text-green-600 dark:text-green-400 flex items-center gap-2">
                                <UIcon name="i-lucide-circle-check" class="size-4" />
                                Device Connected
                            </p>
                            <p class="text-sm text-gray-600 dark:text-gray-400 mt-1">
                                Device ID:
                                <UBadge color="neutral" variant="subtle" class="ml-1 font-mono">
                                    {{ deviceStatus.device_id }}
                                </UBadge>
                            </p>
                            <p class="text-xs text-gray-500 mt-1">
                                Registered: {{ new Date(deviceStatus.registered_at).toLocaleString() }}
                            </p>
                        </div>
                    </div>
                    <UButton
                        color="error"
                        variant="soft"
                        icon="i-lucide-unplug"
                        :loading="loading"
                        @click="disconnectDevice"
                    >
                        Disconnect
                    </UButton>
                </div>
            </UCard>

            <!-- Register Device -->
            <UCard v-else>
                <template #header>
                    <div class="flex items-center gap-2">
                        <UIcon name="i-lucide-scan-qr-code" class="size-5 text-primary" />
                        <h3 class="font-semibold text-lg">Register Your Device</h3>
                    </div>
                </template>

                <!-- QR Scanner Toggle -->
                <UButton
                    block
                    :icon="showScanner ? 'i-lucide-x' : 'i-lucide-qr-code'"
                    color="primary"
                    variant="soft"
                    class="mb-4"
                    @click="showScanner = !showScanner"
                >
                    {{ showScanner ? 'Close Scanner' : 'Scan QR Code' }}
                </UButton>

                <div v-if="showScanner" class="mb-4 p-4 border border-dashed rounded-lg text-center text-sm text-gray-500 dark:text-gray-400">
                    <UIcon name="i-lucide-scan-line" class="size-8 mx-auto mb-2 text-gray-400" />
                    QR Scanner coming soon. Use manual input below.
                </div>

                <USeparator label="or enter manually" class="my-4" />

                <!-- Manual Input -->
                <div class="space-y-3">
                    <p class="text-sm text-gray-600 dark:text-gray-400">Enter your 12-character Device ID:</p>
                    <div class="flex gap-2">
                        <UInput
                            v-model="manualDeviceId"
                            placeholder="e.g. A1B2C3D4E5F6"
                            :maxlength="12"
                            class="flex-1 font-mono"
                            icon="i-lucide-hash"
                            @input="manualDeviceId = manualDeviceId.toUpperCase()"
                        />
                        <UButton
                            icon="i-lucide-plug-zap"
                            color="success"
                            :disabled="manualDeviceId.length !== 12"
                            :loading="loading"
                            @click="handleManualSubmit"
                        >
                            Register
                        </UButton>
                    </div>
                    <UProgress
                        :value="(manualDeviceId.length / 12) * 100"
                        color="primary"
                        size="sm"
                        class="mt-1"
                    />
                    <p class="text-xs text-gray-400 text-right">{{ manualDeviceId.length }}/12 characters</p>
                </div>

                <!-- Instructions -->
                <template #footer>
                    <div class="flex items-start gap-3">
                        <UIcon name="i-lucide-info" class="size-5 text-primary mt-0.5 shrink-0" />
                        <div>
                            <p class="text-sm font-semibold mb-2">How to register:</p>
                            <ol class="text-sm text-gray-600 dark:text-gray-400 space-y-1 list-decimal list-inside">
                                <li>Find the sticker on your BP device</li>
                                <li>Scan the QR code or enter the Device ID (12 characters)</li>
                                <li>Click Register to link the device to your account</li>
                            </ol>
                        </div>
                    </div>
                </template>
            </UCard>
        </div>
    </AdminLayout>
</template>
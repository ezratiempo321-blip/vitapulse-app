<script setup lang="ts">
import {
  onMounted,
  onBeforeUnmount,
  ref,
  defineAsyncComponent,
} from "vue";
import SkeletonLoader from "@/components/SkeletonLoader..vue";
import ErrorComp from "@/components/ErrorComp.vue";
import userLayout from "@/layouts/userLayout.vue";
import { useUserStore } from "@/stores/useUser";
import type { User } from "@/types/types";
import { getTheAge } from "@/utils/getTheAge";
import { useAxios } from '@/axios/useAxios';

const { getUser } = useUserStore();

const AsyncCardComp = defineAsyncComponent({
  loader: () => import("@/components/CardComp.vue"),
  loadingComponent: SkeletonLoader,
  delay: 200,
  errorComponent: ErrorComp,
  timeout: 3000,
});
const AsyncPulseRateCard = defineAsyncComponent({
  loader: () => import("@/components/PulseRateCard.vue"),
  loadingComponent: SkeletonLoader,
  delay: 200,
  errorComponent: ErrorComp,
  timeout: 3000,
});
const AsyncInforList = defineAsyncComponent({
  loader: () => import("@/components/InforList.vue"),
  loadingComponent: SkeletonLoader,
  delay: 200,
  errorComponent: ErrorComp,
  timeout: 3000,
});
const AsyncPage = defineAsyncComponent({
  loader: () => import("@/views/auth/user/historicalData.vue"),
  loadingComponent: SkeletonLoader,
  delay: 200,
  errorComponent: ErrorComp,
  timeout: 3000,
});

const user = ref<User | null>(null);
const data = ref<{
  systolic: number;
  diastolic: number;
  pulse: number;
  bpStatus: string;
  pulseStatus: string;
  clinicalBpLabel: string;
  timestamp: string;
} | null>(null);

let interval: ReturnType<typeof setInterval> | null = null;

const fetchLatestBp = async () => {
  try {
    const res = await useAxios.get("/auth/bp/latest");

    data.value = res.data;
  } catch (error) {
    data.value = null;
  }
};

onMounted(async () => {
  user.value = await getUser();
  await fetchLatestBp();
  interval = setInterval(fetchLatestBp, 5000);
});

onBeforeUnmount(() => {
  if (interval) {
    clearInterval(interval);
    interval = null;
  }
});
</script>

<template>
  <userLayout>
    <UContainer class="py-4 px-2 w-full mt-4">
      <div v-if="user">
        <AsyncInforList :auth="user" />
      </div>

      <h1 class="text-3xl my-10 font-bold self-start">Latest Reading</h1>
      <div
        v-if="data && user?.birthday"
        class="my-5 flex lg:flex-nowrap flex-wrap w-full gap-4"
      >
        <AsyncCardComp
          :bpAndPulse="{
            systolic: data.systolic,
            diastolic: data.diastolic,
            age: Number(getTheAge(String(user?.birthday))),
            pulse: data.pulse,
          }"
        />
        <AsyncPulseRateCard
          :PulseRate="{
            pulse_rate: data.pulse,
            age: Number(getTheAge(String(user?.birthday))),
          }"
        />
      </div>
      <div
        v-else
        class="p-3 my-2 bg-amber-500 text-black rounded-sm shadow-sm flex items-center gap-5"
      >
        <div class="flex flex-col">
          <p class="text-lg font-semibold flex gap-3 items-center">
            <UIcon name="i-lucide-activity" class="size-5" /> No recent readings
          </p>
          <ul class="list-item list-disc ms-15">
            <li>No BP records found</li>
            <li>Please take a measurement first</li>
          </ul>
        </div>
      </div>
    </UContainer>

    <UContainer>
      <AsyncPage />
    </UContainer>
  </userLayout>
</template>
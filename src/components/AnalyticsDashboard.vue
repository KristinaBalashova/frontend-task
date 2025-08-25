<script setup lang="ts">
import { ref, reactive, onMounted } from "vue";
import DataCard from "./DataCard.vue";
import {
  FaceIcon,
  CubeIcon,
  StarIcon,
  RocketIcon,
} from "@radix-icons/vue";

// Types
interface AnalyticsData {
  totalUsers: number;
  totalProducts: number;
  totalRevenue: number;
  activeSessions: number;
  revenueByMonth: Array<{
    month: string;
    revenue: number;
  }>;
}

const loading = ref(false);

// Data
const analyticsData = reactive<AnalyticsData>({
  totalUsers: 0,
  totalProducts: 0,
  totalRevenue: 0,
  activeSessions: 0,
  revenueByMonth: [],
});

const fetchData = () => {
  loading.value = true;
  // Simulating an API call
  setTimeout(() => {
    analyticsData.totalUsers = 100;
    analyticsData.totalProducts = 200;
    analyticsData.totalRevenue = 50000;
    analyticsData.activeSessions = 75;
    analyticsData.revenueByMonth = [
      { month: "January", revenue: 10000 },
      { month: "February", revenue: 15000 },
      { month: "March", revenue: 25000 },
    ];
    loading.value = false;
  }, 1000);
};

onMounted(() => {
  fetchData();
  setInterval(() => {
    fetchData();
  }, 5000);
});
</script>

<template>
  <div class="space-y-6">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-3xl font-bold text-gray-900">Analytics Dashboard</h1>
        <p class="text-gray-500">Overview of key metrics and performance</p>
      </div>
    </div>
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
      <DataCard title="Total Users" :value="analyticsData.totalUsers" :loading="loading">
        <template #icon><FaceIcon /></template>
      </DataCard>
      <DataCard title="Total Products" :value="analyticsData.totalProducts" :loading="loading">
        <template #icon><CubeIcon /></template>
      </DataCard>
      <DataCard title="Total Revenue" :value="analyticsData.totalRevenue" :loading="loading">
        <template #icon><StarIcon /></template>
      </DataCard>
      <DataCard title="Active Sessions" :value="analyticsData.activeSessions" :loading="loading">
        <template #icon><RocketIcon /></template>
      </DataCard>
    </div>
  </div>
</template>

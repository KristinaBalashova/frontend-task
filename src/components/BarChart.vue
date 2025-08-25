<script setup lang="ts">
import { Bar } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
} from "chart.js";


ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
);

interface Props {
  labels: string[];
  datasets: { data: number[] }[];
}

const props = defineProps<Props>();

const chartData = {
    labels: props.labels,
    datasets: props.datasets
}

const chartOptions = {
  responsive: true,
  plugins: {
    legend: {
      display: false
    }
  }
};

</script>

<template>
  <Bar id="my-chart-id" :options="chartOptions" :data="chartData" />
</template>

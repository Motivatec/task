<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from "vue";
import { useRouter } from "vue-router";
import * as echarts from "echarts";

const router = useRouter();
const currentUserRole = localStorage.getItem("role") || "";
const isAdmin = currentUserRole === "admin";

const tickets = ref([
  {
    id: "001",
    project: "Road Project A",
    overtime: true,
    hours: 3.5,
    created_at: "2024-04-10 10:30"
  },
  {
    id: "002",
    project: "Bridge Maintenance B",
    overtime: false,
    hours: 2,
    created_at: "2024-04-09 13:00"
  },
  {
    id: "003",
    project: "Pipeline Fix C",
    overtime: true,
    hours: 4.5,
    created_at: "2024-04-08 08:00"
  },
  {
    id: "004",
    project: "Bridge Maintenance B",
    overtime: true,
    hours: 3,
    created_at: "2024-04-07 16:45"
  },
  {
    id: "005",
    project: "Tunnel Cleaning D",
    overtime: false,
    hours: 8.1,
    created_at: "2024-04-03 11:43"
  }
]);

const chartRef = ref(null);
let chartInstance = null;
let resizeHandler = null;

const chartLabels = computed(() => tickets.value.map((item) => `${item.id} ${item.project}`));
const chartValues = computed(() => tickets.value.map((item) => Number(item.hours)));

function formatOvertime(value) {
  return value ? "Yes" : "No";
}

function formatHours(value) {
  return Number(value).toFixed(1);
}

function deleteTicket(id) {
  if (!isAdmin) {
    return;
  }
  tickets.value = tickets.value.filter((item) => item.id !== id);
}

// 替换截断函数为换行函数
function wrapLabelText(value, maxLength = 6) {
  if (!value) return "";
  // 按指定长度自动换行（支持中文/英文）
  let result = "";
  let lineLength = maxLength;
  for (let i = 0; i < value.length; i += lineLength) {
    result += value.substring(i, i + lineLength) + "\n";
  }
  return result.trim();
}

function ensureChart() {
  if (!chartRef.value) return;
  if (chartInstance) return;
  chartInstance = echarts.init(chartRef.value);
}

function updateChart() {
  ensureChart();
  if (!chartInstance) return;

  const labels = chartLabels.value;
  const values = chartValues.value;
  const maxValue = values.length ? Math.max(...values) : 1;

  // 增大底部间距，确保换行文字完整显示在X轴下方
  const commonGrid = { left: 42, right: 20, top: 20, bottom: 140 };

  const option =
    labels.length === 0
      ? {
          backgroundColor: "#fbfdff",
          title: {
            text: "No Data",
            left: "center",
            top: "middle",
            textStyle: { color: "#6f8092", fontSize: 16 }
          },
          xAxis: { show: false },
          yAxis: { show: false },
          grid: commonGrid
        }
      : {
          backgroundColor: "#fbfdff",
          tooltip: {
            trigger: "axis",
            axisPointer: { type: "shadow" },
            formatter: (params) => {
              if (!params || !params.length) return "";
              const p = params[0];
              const hours = Number(p.data);
              return `${p.axisValue}<br/>Hours: ${hours.toFixed(1)}`;
            }
          },
          grid: commonGrid,
          xAxis: {
            type: "category",
            data: labels,
            axisTick: { show: false },
            axisLine: { lineStyle: { color: "#8ba2b8", width: 1.5 } },
            axisLabel: {
              color: "#243b4e",
              fontSize: 13,
              interval: 0,
              rotate: 0,
              position: "bottom", // 强制标签在轴线下方
              verticalAlign: "top", // 文字从轴线向下排列
              align: "center",
              lineHeight: 18,
              padding: [8, 0, 0, 0], // 上方留白，避免贴在轴线上
              formatter: (value) => wrapLabelText(value, 8)
            }
          },
          yAxis: {
            type: "value",
            min: 0,
            max: maxValue,
            axisLine: { show: true, lineStyle: { color: "#8ba2b8", width: 1.5 } },
            axisLabel: {
              color: "#385166",
              fontSize: 14,
              formatter: (value) => Number(value).toFixed(1)
            },
            splitLine: { show: true, lineStyle: { color: "#dde5ee" } }
          },
          series: [
            {
              type: "bar",
              data: values,
              barGap: "18%",
              itemStyle: {
                color: "rgba(74, 137, 194, 0.78)",
                borderColor: "rgba(31, 63, 87, 0.95)",
                borderWidth: 1
              },
              label: {
                show: true,
                position: "top",
                formatter: (p) => Number(p.value).toFixed(1),
                color: "#243b4e",
                fontSize: 13
              }
            }
          ]
        };

  chartInstance.resize();
  chartInstance.setOption(option, true);
}

function logout() {
  localStorage.removeItem("role");
  localStorage.removeItem("username");
  router.push("/login");
}

onMounted(async () => {
  if (!currentUserRole) {
    router.replace("/login");
    return;
  }
  await nextTick();
  updateChart();

  resizeHandler = () => {
    chartInstance?.resize();
  };
  window.addEventListener("resize", resizeHandler);
});

watch(
  () => tickets.value,
  async () => {
    await nextTick();
    updateChart();
  },
  { deep: true }
);

onUnmounted(() => {
  if (resizeHandler) window.removeEventListener("resize", resizeHandler);
  resizeHandler = null;

  if (chartInstance) {
    chartInstance.dispose();
    chartInstance = null;
  }
});
</script>

<template>
  <main class="home-panel">
    <div class="home-header">
      <h1 class="title">Tasks</h1>
      <button class="logout-btn" type="button" @click="logout">退出登录</button>
    </div>
    <section class="layout">
      <!-- 补充table-container类名，适配CSS样式 -->
      <div class="table-container">
        <table aria-label="工单列表">
          <thead>
            <tr>
              <th>ID</th>
              <th>Project Name</th>
              <th>Overtime</th>
              <th>Hours</th>
              <!-- <th>Created At</th> -->
              <th v-if="isAdmin" class="actions">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ticket in tickets" :key="ticket.id">
              <td>{{ ticket.id }}</td>
              <td>{{ ticket.project }}</td>
              <td>{{ formatOvertime(ticket.overtime) }}</td>
              <td>{{ formatHours(ticket.hours) }}</td>
              <!-- <td>{{ ticket.created_at }}</td> -->
              <td v-if="isAdmin" class="actions">
                <button class="delete-btn" type="button" @click="deleteTicket(ticket.id)">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
        <p class="empty-tip" :hidden="tickets.length !== 0">当前没有工单数据。</p>
      </div>

      <div class="chart-card">
        <h2 class="chart-title">Project Hours Distribution</h2>
        <div
          ref="chartRef"
          id="barChart"
          aria-label="项目工时分布柱状图"
        ></div>
      </div>
    </section>
  </main>
</template>
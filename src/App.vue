<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import * as echarts from "echarts";
import anime from "animejs";
import Datepicker from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

const startDate = ref(new Date(2024, 5, 1));
const endDate = ref(new Date(2024, 11, 30));
const format = ref("yyyy-MM-dd");
const salesChartRef = ref(null);
const trendChartRef = ref(null);

const newGameName = ref("");
const newGameError = ref("");

const games = ref([
  { id: "dave", name: "潛水員戴夫", selected: true },
  { id: "rdr2", name: "碧血狂殺2", selected: true },
  { id: "valheim", name: "瓦爾海姆", selected: true },
  { id: "minecraft", name: "當個創世神", selected: true },
]);

const charts = ref({
  sales: null,
  trend: null,
});

const generateRandomData = () => {
  const data = {};
  games.value.forEach((game) => {
    data[game.id] = Array(12)
      .fill(0)
      .map(() => Math.floor(Math.random() * (2000 - 300 + 1)) + 300);
  });
  return data;
};

const salesData = ref(generateRandomData());

const addNewGame = () => {
  if (!newGameName.value.trim()) {
    newGameError.value = "遊戲名稱不能為空";
    anime({
      targets: '.input-group',
      translateX: [
        { value: -10 },
        { value: 10 },
        { value: -10 },
        { value: 10 },
        { value: 0 }
      ],
      duration: 100,
      easing: 'linear'
    });
    return;
  }

  const existingGame = games.value.find(
    (game) => game.name === newGameName.value
  );
  if (existingGame) {
    newGameError.value = "此遊戲已存在";
    anime({
      targets: '.input-group',
      translateX: [
        { value: -10 },
        { value: 10 },
        { value: -10 },
        { value: 10 },
        { value: 0 }
      ],
      duration: 100,
      easing: 'linear'
    });
    return;
  }

  const newGameId = newGameName.value.toLowerCase().replace(/\s+/g, "-");

  const newGame = {
    id: newGameId,
    name: newGameName.value,
    selected: true,
  };
  games.value.push(newGame);

  salesData.value[newGameId] = Array(12)
    .fill(0)
    .map(() => Math.floor(Math.random() * (2000 - 300 + 1)) + 300);

  newGameName.value = "";
  newGameError.value = "";

  anime({
    targets: `#${newGameId}`,
    scale: [1, 1.2, 1],
    rotate: [0, 5, -5, 0],
    duration: 500,
    easing: "easeInOutQuad",
  });
};

const removeGame = (gameId) => {
  if (games.value.length > 1) {
    const index = games.value.findIndex((game) => game.id === gameId);
    if (index !== -1) {
      anime({
        targets: `#${gameId}`,
        scale: [1, 0],
        opacity: [1, 0],
        duration: 300,
        easing: "easeInOutQuad",
        complete: () => {
          games.value.splice(index, 1);
          delete salesData.value[gameId];
        }
      });
    }
  } else {
    newGameError.value = "至少一款遊戲";
    anime({
      targets: '.game-tags',
      translateX: [
        { value: -10 },
        { value: 10 },
        { value: -10 },
        { value: 10 },
        { value: 0 }
      ],
      duration: 100,
      easing: 'linear'
    });
  }
};

const toggleGameSelection = (gameId) => {
  const game = games.value.find((g) => g.id === gameId);
  if (game) {
    const selectedGames = games.value.filter((g) => g.selected);
    if (selectedGames.length > 1 || !game.selected) {
      game.selected = !game.selected;
      anime({
        targets: `#${gameId}`,
        scale: [1, 1.2, 1],
        rotate: game.selected ? [0, 5, -5, 0] : 0,
        duration: 300,
        easing: "easeInOutQuad",
      });
    } else {
      newGameError.value = "至少選中一款遊戲";
      anime({
        targets: '.game-tags',
        translateX: [
          { value: -10 },
          { value: 10 },
          { value: -10 },
          { value: 10 },
          { value: 0 }
        ],
        duration: 100,
        easing: 'linear'
      });
    }
  }
};

const monthRange = computed(() => {
  const months = [];
  let currentDate = new Date(startDate.value);

  while (currentDate <= endDate.value) {
    months.push(`${currentDate.getMonth() + 1}月`);
    currentDate.setMonth(currentDate.getMonth() + 1);
  }
  return months;
});

const filteredSalesData = computed(() => {
  const startMonth = startDate.value.getMonth();
  const endMonth = endDate.value.getMonth();
  const filtered = {};

  Object.keys(salesData.value).forEach((gameId) => {
    filtered[gameId] = salesData.value[gameId].slice(startMonth, endMonth + 1);
  });

  return filtered;
});

const updateCharts = () => {
  const selectedGames = games.value.filter((game) => game.selected);

  const salesOption = {
    title: { text: "柱狀圖", top: 0 },
    tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
    legend: {
      data: selectedGames.map((game) => game.name),
      top: 25,
    },
    grid: { top: 80, containLabel: true },
    xAxis: {
      type: "category",
      data: monthRange.value,
    },
    yAxis: {
      type: "value"
    },
    series: selectedGames.map((game) => ({
      name: game.name,
      type: "bar",
      data: filteredSalesData.value[game.id],
      animationDuration: 1000,
      animationEasing: "cubicInOut",
    })),
  };

  const trendOption = {
    title: { text: "折線圖", top: 0 },
    tooltip: { trigger: "axis" },
    legend: {
      data: selectedGames.map((game) => game.name),
      top: 25,
    },
    grid: { top: 80, containLabel: true },
    xAxis: {
      type: "category",
      data: monthRange.value,
    },
    yAxis: {
      type: "value",
    },
    series: selectedGames.map((game) => ({
      name: game.name,
      type: "line",
      data: filteredSalesData.value[game.id],
      smooth: true,
      animationDuration: 1000,
      animationEasing: "cubicInOut",
    })),
  };

  charts.value.sales.setOption(salesOption, true);
  charts.value.trend.setOption(trendOption, true);
};

const initCharts = () => {
  charts.value.sales = echarts.init(salesChartRef.value, null, {
    renderer: "canvas",
    width: "auto",
    height: "auto",
  });

  charts.value.trend = echarts.init(trendChartRef.value, null, {
    renderer: "canvas",
    width: "auto",
    height: "auto",
  });
  anime({
    targets: [salesChartRef.value, trendChartRef.value],
    opacity: [0, 1],
    translateY: [50, 0],
    duration: 1000,
    delay: anime.stagger(200),
    easing: 'easeOutQuad'
  });
  updateCharts();
};

onMounted(() => {
  initCharts();
  window.addEventListener("resize", handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
});

watch(
  games,
  () => {
    updateCharts();
  },
  { deep: true }
);

watch([startDate, endDate], () => {
  updateCharts();
});

const handleResize = () => {
  if (charts.value.sales) {
    charts.value.sales.resize({
      width: 'auto',
      height: 'auto'
    })
  }

  if (charts.value.trend) {
    charts.value.trend.resize({
      width: 'auto',
      height: 'auto'
    })
  }
}
</script>

<template>
  <div class="container mt-4">
    <h2 class="text-center mb-4">遊戲價格趨勢儀表板</h2>

    <div class="row mb-4">
      <div class="col-md-6">
        <label class="mb-2">起始日期：</label>
        <Datepicker
          v-model="startDate"
          :enable-time-picker="false"
          locale="zh-TW"
          :format="format"
          auto-apply
          class="w-100"
        />
      </div>
      <div class="col-md-6">
        <label class="mb-2">結束日期：</label>
        <Datepicker
          v-model="endDate"
          :enable-time-picker="false"
          locale="zh-TW"
          :format="format"
          auto-apply
          class="w-100"
        />
      </div>
    </div>

    <div class="row mb-4">
      <div class="col-md-4">
        <div class="input-group">
          <input
            type="text"
            class="form-control"
            v-model="newGameName"
            placeholder="請輸入遊戲名稱"
            @keyup.enter="addNewGame"
          />
          <button class="btn btn-primary" @click="addNewGame">新增遊戲</button>
        </div>
        <small v-if="newGameError" class="text-danger">
          {{ newGameError }}
        </small>
      </div>
    </div>

    <div class="row mb-4">
      <div class="col-md-12">
        <div class="game-tags">
          <div
            v-for="game in games"
            :key="game.id"
            class="game-tag"
            :class="{ selected: game.selected }"
          >
            <button class="remove-tag" @click="removeGame(game.id)">✕　</button>
            <span @click="toggleGameSelection(game.id)">
              {{ game.name }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <div class="row">
      <div class="col-md-6 mb-4">
        <div
          ref="salesChartRef"
          class="chart-container"
          style="width: 100%; height: 400px"
        ></div>
      </div>
      <div class="col-md-6 mb-4">
        <div
          ref="trendChartRef"
          class="chart-container"
          style="width: 100%; height: 400px"
        ></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dp__theme_light {
  --dp-background-color: #ffffff;
  --dp-text-color: #212121;
  --dp-hover-color: #f3f3f3;
  --dp-hover-text-color: #212121;
  --dp-hover-icon-color: #959595;
  --dp-primary-color: #1976d2;
  --dp-primary-text-color: #f8f9fa;
  --dp-secondary-color: #c0c4cc;
  --dp-border-color: #ddd;
  --dp-menu-border-color: #ddd;
  --dp-border-color-hover: #aaaeb7;
  --dp-disabled-color: #f6f6f6;
  --dp-scroll-bar-background: #f3f3f3;
  --dp-scroll-bar-color: #959595;
  --dp-success-color: #76d275;
  --dp-success-color-disabled: #a3d9b1;
  --dp-icon-color: #959595;
  --dp-danger-color: #ff6f60;
}

.form-check-inline {
  margin-right: 1rem;
}

.game-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.game-tag {
  display: inline-flex;
  align-items: center;
  background-color: #f0f0f0;
  border: 2px solid rgb(178, 178, 178);
  border-radius: 16px;
  padding: 4px 8px;
  margin: 2px;
  transition: all 0.3s ease;
}

.game-tag.selected {
  background-color: transparent;
}

.game-tag:hover {
  transform: scale(1.2);
}

.remove-tag {
  background: none;
  border: none;
  color: #ff4d4f;
  cursor: pointer;
  font-size: 14px;
  padding: 0;
  line-height: 1;
}

.remove-tag:hover {
  color: #fcb4b3;
  transform: scale(1.2);
}

.chart-container {
  width: 100%;
  height: 400px;
  resize: both;
  overflow: auto;
}
</style>

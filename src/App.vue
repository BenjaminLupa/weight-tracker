<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(104.0);

onMounted(() => {
  const savedData = JSON.parse(localStorage.getItem("weight"));

  if (savedData) {
    weights.value = savedData;
  }
});

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
  saveWeightsToLocalStorage();
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => new Date(w.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .slice(-7);

      weightChart.value.update();

      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),

          datasets: [
            {
              label: " weight",
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              backgroundColor: "rgba(0, 117, 10, 0.3)",
              borderColor: "rgb(0, 117, 10)",
              borderwidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);

const saveWeightsToLocalStorage = () => {
  localStorage.setItem("weight", JSON.stringify(weights.value));
};
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>

    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />
      <input type="submit" value="Add weight" />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2>Last 7 Days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span> {{ weight.weight }} Kg </span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "montserrat", sans-serif;
}

body {
  background-color: #00750a67;
  width: 100%;
  height: 100vh;
}

main {
  padding: 1.5rem;
  background-color: #00750a67;
  margin: -10px;
}

h1 {
  font-size: 2em;
  text-align: center;
  margin-bottom: 2rem;
  color: #00750a;
}

h2 {
  display: flex;
  margin-bottom: 1rem;
  color: #4c4c4c;
  font-weight: 400;
  justify-content: center;
}

.current {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  width: 200px;
  height: 200px;

  text-align: center;
  background-color: #6aff79;
  border-radius: 999px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border: 5px solid #00750a;

  margin: 0 auto 2rem;
}

.current span {
  display: block;
  font-size: 2em;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.current small {
  color: #888;
  font-style: italic;
}

form {
  display: flex;
  margin-bottom: 2rem;
  border: 1px solid #aaa;
  border-radius: 0.5rem;
  overflow: hidden;
  transition: 200ms linear;
  max-width: 720px;
  margin-inline: auto;
}

form:focus-within,
form:hover {
  border-color: #00750a;
  border-width: 2px;
}

form input[type="number"] {
  appearance: none;
  outline: none;
  border: none;
  background-color: white;

  flex: 1 1 0%;
  padding: 1rem 1.5rem;
  font-size: 1.25rem;
}

form input[type="submit"] {
  appearance: none;
  outline: none;
  border: none;
  cursor: pointer;
  background-color: #00750a;

  padding: 0.5rem 1rem;

  color: white;
  font-size: 1.25rem;
  font-weight: 700;
  transition: 200ms linear;
  border-left: 3px solid transparent;
}

form input[type="submit"]:hover {
  background-color: white;
  color: #00750a;
  border-left-color: #00750a;
}

.canvas-box {
  width: 100%;
  max-width: 720px;
  background-color: #6aff79;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
  margin-inline: auto;
}

.weight-history ul {
  list-style: none;
  padding: 0;
  margin-inline: auto;
  max-width: 720px;
  border: 2px solid #00750a;
  border-radius: 8px;
}

.weight-history ul li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  cursor: pointer;
  background-color: #6aff797a;
  color: #6f6282;
  transition: 0.6s;
}

.weight-history ul li:nth-child(even) {
  background-color: #6aff79;
  color: #6f6282;
}

.weight-history ul li:first-child {
  border-radius: 8px 8px 0 0;
}

.weight-history ul li:last-child {
  border-radius: 0 0 8px 8px;
}
.weight-history ul li:hover {
  background-color: #f8f8f8;
  color: #000;
}

.weight-history ul li:last-of-type {
  border-bottom: none;
}

.weight-history ul li span {
  display: block;
  font-size: 1.25rem;
  font-weight: 700;
  margin-right: 1rem;
}

.weight-history ul li small {
  color: #6f6282;
  font-style: italic;
}
</style>

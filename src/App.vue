<template>
  <main>
    <!-- Page title -->
    <h1>Weight Tracker</h1>

    <!-- Display current weight -->
    <div class="current">
      <span>{{ currentWeight }}</span>
      <small>Current weight (kg)</small>
    </div>

    <!-- Form to input new weight -->
    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="newWeight" />
      <input type="submit" value="Add weight" />
    </form>

    <!-- Display weight data if available -->
    <div v-if="weightData && weightData.length > 0">
      <!-- Last 7 days chart -->
      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <!-- Weight history list -->
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weightData">
            <span>
              {{ weight.weight }}kg
            </span>
            <small>
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<script setup>
// Import Vue 3 features
import { ref, shallowRef, computed, watch, nextTick } from 'vue'
import { Chart } from 'chart.js/auto'

// Reactive variables
const weightData = ref([]) // Stores weight data
const weightChartEl = ref(null) // Chart canvas element
const weightChart = shallowRef(null) // Chart instance
const newWeight = ref(60.0) // Input for new weight

// Compute the current weight as the latest recorded weight
const currentWeight = computed(() => {
  // Sort the weight data by date in descending order
  const sortedWeights = weightData.value.sort((a, b) => b.date - a.date)
  // Return the weight of the latest entry or 0 if no data is available
  return sortedWeights[0]?.weight || 0
})

// Function to add a new weight entry
const addWeight = () => {
  weightData.value.push({
    weight: newWeight.value,
    date: new Date().getTime()
  })
}

// Watch for changes in weight data and update the chart
watch(weightData, newWeights => {
  const ws = [...newWeights]

  // Check if the chart instance exists
  if (weightChart.value) {
    // Update chart labels and data for the last 7 days
    weightChart.value.data.labels = ws
      .sort((a, b) => a.date - b.date)
      .map(w => new Date(w.date).toLocaleDateString())
      .slice(-7)

    weightChart.value.data.datasets[0].data = ws
      .sort((a, b) => a.date - b.date)
      .map(w => w.weight)
      .slice(-7)

    // Update the chart
    weightChart.value.update()
  }

  // Create a new chart instance
  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: ws
          .sort((a, b) => a.date - b.date)
          .map(w => new Date(w.date).toLocaleDateString()),
        datasets: [
          {
            label: 'Weight',
            data: ws
              .sort((a, b) => a.date - b.date)
              .map(w => w.weight),
            borderColor: 'rgb(255, 105, 180, 0.2)',
            backgroundColor: 'rgb(255, 105, 180)',
            borderWidth: 1,
            fill: true
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    })
  })
}, { deep: true })
</script>

<style>

/* DEFINE ROOT VARIABLES FOR COLORS */
:root {
  --background-color: #eee;
  --primary-color: hotpink;
  --secondary-color: #888;
  --border-color: hwb(330 41% 0%);
  --accent-color: white;
  --shadow-color: rgba(0, 0, 0, 0.1);
  --highlight-color: #f8f8f8;
  --even-list-item-color: #dfdfdf;
  --text-color: #888;
}

/* RESET DEFAULT STYLES AND SET FONT FAMILY */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'montserrat', sans-serif;
}

/* SET BACKGROUND COLOR FOR THE BODY */
body {
  background-color: var(--background-color);
}

/* STYLE FOR THE MAIN CONTAINER */
main {
  padding: 1.5rem;
}

/* STYLE FOR THE PAGE TITLE */
h1 {
  font-size: 2em;
  text-align: center;
  margin-bottom: 2rem;
}

/* STYLE FOR SECONDARY HEADINGS */
h2 {
  margin-bottom: 1rem;
  color: var(--secondary-color);
  font-weight: 400;
}

/* STYLE FOR THE CURRENT WEIGHT DISPLAY */
.current {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 200px;
  height: 200px;
  text-align: center;
  background-color: var(--accent-color);
  border-radius: 999px;
  box-shadow: 0 4px 12px var(--shadow-color);
  border: 5px solid var(--border-color);
  margin: 0 auto 2rem;
}

.current span {
  display: block;
  font-size: 2em;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.current small {
  color: var(--text-color);
  font-style: italic;
}

/* STYLE FOR THE FORM */
form {
  display: flex;
  margin-bottom: 2rem;
  border: 1px solid var(--border-color);
  border-radius: 0.5rem;
  overflow: hidden;
  transition: 200ms linear;
}

/* HIGHLIGHT FORM ON FOCUS AND HOVER */
form:focus-within,
form:hover {
  border-color: var(--primary-color);
  border-width: 2px;
}

/* STYLE FOR NUMBER INPUT IN THE FORM */
form input[type="number"] {
  appearance: none;
  outline: none;
  border: none;
  background-color: var(--accent-color);
  flex: 1 1 0%;
  padding: 1rem 1.5rem;
  font-size: 1.25rem;
}

/* STYLE FOR THE SUBMIT BUTTON IN THE FORM */
form input[type="submit"] {
  appearance: none;
  outline: none;
  border: none;
  cursor: pointer;
  background-color: var(--primary-color);
  padding: 0.5rem 1rem;
  color: white;
  font-size: 1.25rem;
  font-weight: 700;
  transition: 200ms linear;
  border-left: 3px solid transparent;
}

/* HIGHLIGHT SUBMIT BUTTON ON HOVER */
form input[type="submit"]:hover {
  background-color: white;
  color: var(--primary-color);
  border-left-color: var(--primary-color);
}

/* STYLE FOR THE CHART CANVAS CONTAINER */
.canvas-box {
  width: 100%;
  max-width: 720px;
  background-color: var(--accent-color);
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 4px 12px var(--shadow-color);
  margin-bottom: 2rem;
}

/* STYLE FOR THE WEIGHT HISTORY LIST */
.weight-history ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

/* STYLE FOR LIST ITEMS IN THE WEIGHT HISTORY */
.weight-history ul li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  cursor: pointer;
}

/* ALTERNATE BACKGROUND COLOR FOR LIST ITEMS */
.weight-history ul li:nth-child(even) {
  background-color: var(--even-list-item-color);
}

/* HIGHLIGHT LIST ITEM ON HOVER */
.weight-history ul li:hover {
  background-color: var(--highlight-color);
}

/* REMOVE BORDER FROM THE LAST LIST ITEM */
.weight-history ul li:last-of-type {
  border-bottom: none;
}

/* STYLE FOR WEIGHT DISPLAY IN LIST ITEMS */
.weight-history ul li span {
  display: block;
  font-size: 1.25rem;
  font-weight: 700;
  margin-right: 1rem;
}

/* STYLE FOR DATE DISPLAY IN LIST ITEMS */
.weight-history ul li small {
  color: var(--text-color);
  font-style: italic;
}

</style>


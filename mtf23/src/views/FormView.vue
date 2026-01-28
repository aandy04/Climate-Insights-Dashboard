<!-- referenced AI how does composition API work with axios and chartjs -->
<template>
  <div>
    <!-- Input form for weather data -->
    <form @submit.prevent="fetch_remote_data">
      <div class="form-group">
        <h2> Weather Data </h2>
        <p>Please fill in the form and use the submit button to load data.</p>

        <label>Location</label><br />
        <input
          class="form-control rounded"
          v-model="form_input.location"
          type="text"
          required
        />
      </div>

      <div class="form-group">
        <label>Start Date</label><br />
        <input
          class="form-control rounded"
          type="date"
          v-model="form_input.start_dt"
          required
        />
      </div>

      <div class="form-group">
        <label>End Date</label><br />
        <input
          class="form-control rounded"
          type="date"
          v-model="form_input.end_dt"
          required
        />
      </div>

      <br />
      <div>
        <button class="btn btn-danger" type="submit">Submit</button>
      </div>
    </form>

    <br /><br />

    <div v-if="dataLoaded">
      <Bar :data="chart_data" :options="chart_options" />
    </div>

    <div v-if="dataLoaded">
      <hr />
      <h5>Weather Data in a Table</h5>
      <table class="table">
        <thead>
          <tr>
            <th v-for="adate in keys" :key="adate">{{ adate }}</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td v-for="atemp in values" :key="atemp">{{ atemp }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
// Import necessary Vue, Axios, and Chart.js modules
import { ref, reactive } from "vue";
import axios from "axios";

import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
} from "chart.js";
import { Bar } from "vue-chartjs";

// register chart elements once
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

// Define the form input fields using reactive() for two-way binding
const form_input = reactive({
  start_dt: "",
  end_dt: "",
  location: "",
});

// Store chart data reactively
const chart_options = {
  responsive: true,
  maintainAspectRatio: true,
  scales: {
    x: {
      title: {
        display: true,
        text: "Time",
      },
    },
    y: {
      title: {
        display: true,
        text: "Temperature (F)",
      },
    },
  },
};

const chart_data = ref({
  labels: [],
  datasets: [
    {
      label: "Average Temperature",
      backgroundColor: "#f87979",
      data: [],
    },
  ],
});

// Variables to control UI state
const dataLoaded = ref(false);
const keys = ref([]);
const values = ref([]);

// Function to fetch weather data from the API
async function fetch_remote_data() {
  // API request configuration
  const options = {
    method: "GET",
    url: "https://weatherapi-com.p.rapidapi.com/history.json",
    params: {
      q: form_input.location,
      dt: form_input.start_dt,
      end_dt: form_input.end_dt,
      lang: "en",
    },
    headers: {
      "X-RapidAPI-Key": "46d2c5ac96msh5403bf83b9df753p163099jsn1e39d7d5ddc5",
      "X-RapidAPI-Host": "weatherapi-com.p.rapidapi.com",
    },
  };

  try {
    // Send API request and wait for response
    const resp = await axios.request(options);
    // Extract date and temperature data from API response
    const w_data = resp.data.forecast.forecastday;

    keys.value = w_data.map((item) => item.date);
    values.value = w_data.map((item) => item.day.avgtemp_f);
    
    // Update chart data for visualization
    chart_data.value = {
      labels: keys.value,
      datasets: [
        {
          label: "Average Temperature",
          backgroundColor: "#f87979",
          data: values.value,
        },
      ],
    };
    // Indicate that data is ready to display
    dataLoaded.value = true;
  } catch (error) {
    console.error(error);
  }
}
</script>

<style>
table,
th,
td {
  border: 1px solid black;
  border-collapse: collapse;
}
</style>

<template>
    <div>
        <form @submit.prevent="fetch_remote_data">
            <div class="form-group">
                <h2> Weather Data </h2>
                <p> Please fill in the form and use the submit button to load data.</p>
                <label for="">Location</label>
                <br>
                <input class="form-control rounded" v-model="form_input.location" type="text" required />
            </div>
            <div class="form-group">
                <label for="">Start Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="form_input.start_dt" required />
            </div>
            <div class="form-group">
                <label for="">End Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="form_input.end_dt" required />
            </div>
            <br>
            <div>
                <button class="btn btn-danger" type="submit">Submit</button>
            </div>
        </form>
        <br /><br />
        <div v-if="dataLoaded">
            <Bar :data="chart_data" :options="chart_options" />
        </div>
        <div v-if="dataLoaded">
            <hr>
            <h5> Weather Data in a Table</h5>
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

<script>
// Imports necessary Composition API utilities and libraries
import { ref, reactive } from 'vue';
import axios from 'axios';
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js';
import { Bar } from 'vue-chartjs';

// Registers chart.js components for rendering charts
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

// Defines chart configuration and data as reactive objects
const chart_options = {
    responsive: true,
    maintainAspectRatio: true,
    scales: {
        x: { title: { display: true, text: "Time" } },
        y: { title: { display: true, text: "Temperature (F)" } }
    }
};
      
export default {
  components: { Bar },
  setup() {
    // Convert data properties to reactive variables 
    // This replaces the data() function in the Options API with Composition API form
      const form_input = reactive({ start_dt: "", end_dt: "", location: "" }); // Holds form input data
      
      const chart_data = reactive({
          labels: [],
          datasets: [{ label: "Average Temperature", backgroundColor: "#f87979", data: [] }]
      });
      const dataLoaded = ref(false); // Controls display of chart and table
      const keys = ref([]); // Stores dates for the table headers and chart labels
      const values = ref([]); // Stores temperature values for chart and table

    // Converts methods to functions within setup() like 'fetch_remote_data' which was originally in 
    // methods but is now standalone
      async function fetch_remote_data() {
          const options = {
              method: 'GET',
              url: 'https://weatherapi-com.p.rapidapi.com/history.json',
              params: {
                  q: form_input.location,
                  dt: form_input.start_dt,
                  end_dt: form_input.end_dt,
                  lang: 'en'
              },
              headers: {
                  'X-RapidAPI-Key': '6c6f790650msh5ce4da2fced77a7p1b1776jsn247d9f531b74',
                  'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
              }
          };

          try {
            // Making the API request and handling the response
              const resp = await axios.request(options);
              const w_data = resp.data.forecast.forecastday;

            // Update 'keys' and 'values' arrays with dates and temperatures  
              keys.value = w_data.map(item => item.date);
              values.value = w_data.map(item => item.day.avgtemp_f);

            // Update chart_data to display on the chart  
              chart_data.labels = keys.value;
              chart_data.datasets[0].data = values.value;

            // Set dataLoaded to true to display chart and table  
              dataLoaded.value = true;
          } catch (error) {
              console.error(error);
          }
      }

    // Returns all functions and variables to make them avaliable in the template
      return { form_input, dataLoaded, chart_data, chart_options, keys, values, fetch_remote_data };
  }
};
</script>

<style>
table, th, td {
    border: 1px solid black;
    border-collapse: collapse;
}
</style>

// Used chatGPT to assist in ensuring that the conversion was correct and that there were no errors, QUERY: "Will you check this code
// and make sure that the conversion from the original code with options API to composition API works correctly."

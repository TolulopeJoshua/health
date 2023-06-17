<script setup>
import {Chart} from 'highcharts-vue'
import axios from 'axios'
import { ref, computed } from 'vue'

const bloodGroups = ['A', 'B', 'AB', 'O'], ageRanges = ['0-20','21-40','41-60','61-80'], persons = ref([]), live = ref(1);

if(navigator.onLine) {
  axios.get('https://health-ff5da-default-rtdb.firebaseio.com/data.json').then(res => {
    res.data.forEach(p => persons.value.push(p));
    localStorage.setItem('healthdata', res.data);
  }) 
} else {
  live.value = 0
  const data = localStorage.getItem('healthdata');
  data && data.forEach(p => persons.value.push(p));
}

const chartOptionsBG = computed(() => ({
  chart: { type: 'bar', },
        title: { text: 'Blood Groups', },
        xAxis: { categories: bloodGroups, },
        yAxis: { allowDecimals: false, title: { text: null }, },
        legend: { enabled: false },
        series: [{data: bloodGroups.map(grp => persons.value.filter(person => person.bgroup == grp).length)}]
})), chartOptionsAG = computed(() => ({
  chart: { type: 'bar', },
        title: { text: 'Age Groups', },
        xAxis: { categories: ageRanges, },
        yAxis: { allowDecimals: false, title: { text: null }, },
        legend: { enabled: false },
        series: [{data: ageRanges.map(rg => persons.value.filter(person => (person.age <= rg.split('-')[1] && person.age >= rg.split('-')[0])).length)}]
}))
</script>

<template>
  <header>
    <h1>Blood & Age Distribution</h1>
    <span v-show="!live">OFFLINE</span>
  </header>

  <main>
    <div>
      <Chart :options="chartOptionsBG"></Chart>
    </div>
    <div>
    <Chart :options="chartOptionsAG"></Chart>
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>

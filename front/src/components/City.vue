<template>
  <div id="city" @click="expand()">
      <div>{{this.name}}</div>
      <div v-if="this.expanded">
          <div v-bind:key="measurement.id" v-for="measurement in this.measurements">
              {{measurement.value}}
          </div>
      </div>
  </div>
</template>

<script>
import axios from 'axios';
//import Chart from 'chart.js'


export default {
    name: 'City',
    props: ['city','name','id'],
    components: {

    },
    data() {
    return {
        expanded: false,
        measurements: []
    }
    },
    methods: {
    expand() {
        if (this.measurements.length===0) {
            const url = 'https://api.openaq.org/v1/measurements?city='+this.city;
            axios.get(url)
                .then(response => this.measurements = response.data.results)
                .catch(function(err) { console.log(err)});
        }
        this.expanded = !this.expanded;
    }
  }
}
</script>

<style>

#city {
    cursor: pointer;
    background-color: white;
    line-height: 30px;
    padding: 5px;
    margin: 5px 0;
    font-size: 24px;
}


</style>

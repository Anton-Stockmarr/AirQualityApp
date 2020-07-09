<template>
  <div id="app">
    <h1>Open AQ</h1>
    <h2><a href="https://openaq.org">https://openaq.org</a></h2>
    <div id="data-box">
      <div v-bind:key="country.id" v-for="country in this.countries">
        <Country v-bind="country" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Country from './components/Country'

export default {
  name: 'App',
  components: {
    Country
  },
  data() {
    return {
      countries: []
    }
  },
  mounted() {
    axios.get('https://api.openaq.org/v1/countries')
      .then(response => this.countries = response.data.results)
      .catch(function(err) { console.log(err)});
  }
}
</script>

<style>
@import "reset.css";

a {
  color: black;
  text-decoration: none;
}


#app {
  margin: 0 auto;
  width: 100%;
  background-color: gray;
}

#app h1 {
  padding: 30px 0 10px;
  text-align: center;
  font-size: 32px;
}

#app h2 {
  text-align: center;
  font-size: 20px;
}

#data-box {
  margin: 20px 20px 0;
  padding-bottom: 20px;
}
</style>

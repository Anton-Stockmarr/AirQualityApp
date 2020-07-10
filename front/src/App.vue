<template>
  <div id="app">
    <h1>Open AQ</h1>
    <h2><a href="https://openaq.org">
      https://openaq.org
    </a></h2>
    <nav>
      <div id="page-navigation">
        <h3>page:</h3>
        <div class="page-btn" :key="p" v-for="p in this.maxpages">
          <button :class="{chosen: p === page}" @click="changePage(p)">{{p}}</button>
        </div>
      </div>
      <div id="coordinate-input">
        <h3>longitude:</h3>
        <input type="text" v-model="longitude">
        <h3>latitude:</h3>
        <input type="text" v-model="latitude">
        <button @click="search()">Find nearest</button>
      </div>
    </nav>
    <div v-show="searchPage">
      <button @click="toggleSearch(false)">Back</button>
      <Location
        :longitude="longitude"
        :latitude="latitude"
        :searchBus="searchBus"/>
    </div>
    <div v-if="!searchPage">
      <div v-if="error.status">{{error.message}}</div>
      <div :key="index" v-for="(country, index) in this.countries">
        <Country v-bind="country" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Country from './components/Country'
import Location from './components/Location'
import Vue from 'vue'

export default {
  name: 'App',
  components: {
    Country,
    Location
  }, 
  data() {
    return {
      searchPage: false,
      searchBus: new Vue(),
      page: 1,
      maxpages: 5,
      countries: [],
      longitude: '',
      latitude: '',
      error: {
        status: false,
        message: ''
      }
    }
  },
  mounted() {
    this.getCountries();
},
  methods: {
    search() {
      this.toggleSearch(true);
      this.searchBus.$emit('search');
    },
    changePage(newPage){
      if (newPage!==this.page){
        this.page= newPage;
        this.getCountries();
      }
      if (this.searchPage) {
        this.toggleSearch();
      }
    },
    getCountries(){
      axios.get(`https://api.openaq.org/v1/countries?limit=20&page=${this.page}`)
        .then(response => this.parseCountries(response.data.results))
        .catch(err => { this.setError(err)});
    },
    parseCountries(result){
      if (result.length === 0){
        this.setError('No countries were loaded');
      } else {
        this.clearError();
        this.countries = result;
      }
    },
    toggleSearch(newState) {
      this.searchPage = newState;
      this.renderSearchChart = newState;
    },
    setError(message){
      this.error.status = true;
      this.error.message = message;
    },
    clearError(){
      this.error.status = false;
      this.error.message = '';
    }
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
  box-sizing: border-box;
  padding: 20px;
  margin: 0 auto;
  width: 100%;
  min-height: 100vh;
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

nav {
  background-color: cadetblue;
  box-sizing: border-box;
  height: 60px;
  width: 100%;
  padding: 10px 20px;
  margin: 20px 0;
}

nav h3 {
  display: inline-block;
  margin-right: 5px;
  font-size: 20px;
}



nav div {
  display: block;
  height: 40px;
  line-height: 40px;
}

#page-navigation {
  float: left;
}

#page-navigation .page-btn{
  display: inline-block;
  height: 30px;
  width: 30px;
  margin: 0 5px;
}

.page-btn button {
  height: 100%;
  width: 100%;
}

.chosen {
  background-color: #DAAD86;
}

#coordinate-input {
  float: right;
}

#coordinate-input input {
  margin-right: 20px;
}

</style>

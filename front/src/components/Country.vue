<template>
    <div id="country" @click="expand()">
        {{name}}
        <div id='cities' v-if="expanded">
            <div v-if="error.status"><h3>{{error.message}}</h3></div>
            <div v-bind:key="index" v-for="(city, index) in this.cities">
                <City v-bind="city"/>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import City from './City'

export default {
    name: 'Country',
    props: ['code','name'],
    components: {
        City
    },
    data() {
    return {
        expanded: false,
        rendered: false,
        cities: [],
        error: {
            status: false,
            message: ''
        }
    }
    },
    methods: {
    expand() {
        if (!this.rendered) {
            axios.get(`https://api.openaq.org/v1/cities?country=${this.code}`)
                .then(response => this.parseCities(response.data.results))
                .catch(err => this.setError(err));
        }
        this.expanded = !this.expanded;
    },
    reset() {
        this.expanded = false;
        this.rendered = false;
        this.cities = [];
        this.clearError();
    },
    parseCities(result){
        if (result.length === 0) {
            this.setError("No cities found");
        } else {
            this.cities = result;
            this.rendered = true; 
            this.clearError();       
        }
    },
    setError(message){
      this.error.status = true;
      this.error.message = message;
    },
    clearError(){
      this.error.status = false;
      this.error.message = '';
    }
  },
  watch: {
      name: function() {
          this.reset();
      }
  }
}
</script>

<style>

#country {
    cursor: pointer;
    background-color: lightblue;
    line-height: 30px;
    padding: 5px;
    margin: 5px 0;
    font-size: 24px;
}

#cities {
    padding: 10px;
}

</style>

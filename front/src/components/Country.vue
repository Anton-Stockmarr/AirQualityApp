<template>
    <div id="country" @click="expand()">
        {{name}}
        <div id='cities' v-if="expanded">
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
    props: ['code','name','id'],
    components: {
        City
    },
    data() {
    return {
        expanded: false,
        rendered: false,
        cities: []
    }
    },
    methods: {
    expand() {
        if (!this.rendered) {
            const url = 'https://api.openaq.org/v1/cities?country='+this.code;
            axios.get(url)
                .then(response => {
                    this.cities = response.data.results;
                    this.rendered = true;
                })
                .catch(function(err) { console.log(err)});
        }
        this.expanded = !this.expanded;
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

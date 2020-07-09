<template>
    <div id="city" @click="expand()">
        <div>{{this.name}}</div>
        <div id="chart-container" v-if="this.expanded">
            <AirChart v-if="this.rendered" id="chart" v-bind="chart"/>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
//import chart from 'chart.js'
import AirChart from './AirChart'


export default {
    name: 'City',
    props: ['city','name','id'],
    components: {
        AirChart
    },
    data() {
        return {
            expanded: false,
            rendered: false,
            chart: {chartdata: {}, options: {}}
        }
    },
    methods: {
    expand() {
        if (!this.rendered) {
            const url = 'https://api.openaq.org/v1/measurements?city='+this.city;
            axios.get(url)
                .then(response => this.parseData(response.data.results))
                .then(measurements => this.configureChart(measurements))
                .then(() => this.rendered = true)
                .catch(function(err) { console.log(err)});
        }
        this.expanded = !this.expanded;
    },
    parseData(result) {
        let measurements = {
            values: [],
            dates: [],
            unit: ''
            }
        measurements.values = result.map(measurement => measurement.value);
        measurements.dates = result.map(measurement => measurement.date.local);
        if (result.length != 0) {
            measurements.unit = result[0].unit;
        }
        return measurements;
    },
    configureChart(measurements) {
        this.chart.chartdata = {
            labels: measurements.dates,
            datasets: [{
                label: 'Pollution ['+measurements.unit+']',
                backgroundColor: '#f87979',
                data: measurements.values,
                borderWidth: 1
            }]
        };
        this.chart.options = {
            responsive:true,
            maintainAspectRatio: false,
            scales: {
                yAxes: [{
                    ticks: {
                        beginAtZero: true
                    }
                }]
            }
        }
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

#chart-container {
    padding: 10px 30px 0;
    width: 100%;
    height: 400px;
}

#chart {
    width: calc(100% - 60px);
    height: calc(100% - 10px);
}


</style>

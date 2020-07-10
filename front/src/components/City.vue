<template>
    <div id="city" @click="expand($event)">
        <div>{{this.name}}</div>
        <div v-if="this.expanded">
            <div v-if="this.error.status">
                <h3>{{this.error.message}}</h3>
            </div>
            <div  id="chart-container" v-if="this.rendered & !this.error.status">
            <AirChart id="chart" v-bind="chart"/>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import AirChart from './AirChart'


export default {
    name: 'City',
    props: ['city','name'],
    components: {
        AirChart
    },
    data() {
        return {
            expanded: false,
            rendered: false,
            chart: {
                chartdata: {},
                options: {}
            },
            error: {
                status: false,
                message: ''
            }
        }
    },
    methods: {
    expand(event) {
        event.stopPropagation();
        if (!this.rendered) {
            const url = `https://api.openaq.org/v1/measurements?city=${this.city}`;
            axios.get(url)
                .then(response => this.parseData(response.data.results))
                .then(() => this.rendered = true)
                .catch(function(err) { console.log(err)});
        }
        this.expanded = !this.expanded;
    },
    parseData(result) {
        if (result.length === 0) {
            this.setError('No measurements were found for this city');
            return 0;
        } else {
            this.clearError();
        }
        let measurements = {
            values: [],
            dates: [],
            unit: ''
            }
        measurements.values = result.map(measurement => measurement.value);
        measurements.dates = result.map(measurement => measurement.date.local);
        measurements.unit = result[0].unit;
        this.configureChart(measurements);
    },
    configureChart(measurements) {
        this.chart.chartdata = {
            labels: measurements.dates,
            datasets: [{
                label: `Pollution [${measurements.unit}]`,
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
    max-width: 100%;
    height: 400px;
}

#chart {
    width: calc(100% - 60px);
    height: calc(100% - 10px);
}


</style>

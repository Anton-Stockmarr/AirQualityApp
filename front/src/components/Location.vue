<template>
    <div id="location">
        <div v-if="!error.status">
            <div v-if="this.city!=''">{{this.city}} ({{this.country}})</div>
            <div id="chart-container">
                <AirChart v-if="this.rendered" id="chart" v-bind="chart"/>
            </div>
        </div>
        <div v-if="error.status">
            <div>{{error.message}}</div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import AirChart from './AirChart'


export default {
    name: 'Location',
    props: ['longitude','latitude','searchBus'],
    components: {
        AirChart
    },
    data() {
        return {
            country: '',
            city: '',
            location: '',
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
    created() {
        this.searchBus.$on('search', () => this.searchLocation());
    },
    methods: {
        searchLocation(){
            this.rendered = false;
            this.clearError();
            if (this.checkInput(this.longitude,this.latitude)){
                return;
            }
            this.getLocation()
                .then(() => {
                    if (!this.error.status){
                        return this.getData();
                    }
                })
                .then(() => {
                    if (!this.error.status) {
                        this.rendered = true;
                    }
                });
        },
        checkInput(long,lat) {
            let regex = /^[\d]{1,2}(\.[\d]*)?$/;
            if (long === '' || lat === '') {
                this.setError('Both longitude and longitude must be filled out properly to search.');
            }
            else if (!long.match(regex) && !lat.match(regex)){
                this.setError('You can only enter numbers, using digits and `.` or `,`');
            }
            else if (long < -180 || long> 180) {
                this.setError('Longitude must be between -180 and 180');
            }
            else if (lat < -85 || lat > 85){
                this.setError('Latitude must be between -85 and 85');
            }
            return this.error.status;
        },
        getLocation() {
            return axios.get(`https://api.openaq.org/v1/locations?coordinates=${this.latitude},${this.longitude}&order_by=distance&radius=100000`)
                .then(response => this.parseLocation(response.data.results))
                .catch(err => this.setError(err));
        },
        parseLocation(result) {            
            if (result.length === 0){
                this.setError("No locations found for these coordinates");
            } else {
                this.clearError();
                this.location = result[0].location;
                this.city = result[0].city;
                this.country = result[0].country;
            }
        },
        getData() {
            return axios.get(`https://api.openaq.org/v1/measurements?location=${this.location}`)
                .then(response => this.parseData(response.data.results))
                .catch(err => this.setError(err));
        },
        parseData(result) {
            if (result.length === 0) {
                this.setError(`${this.city} (${this.country}): No measurements were found`);
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

#location {
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

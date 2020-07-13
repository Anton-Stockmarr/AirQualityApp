<script>
import { Line } from 'vue-chartjs'

export default {
  extends: Line,
  props: ['measurements'],
  data() {
    return {
      chartdata: {},
      options: {}
    }
  },
  mounted () {
    this.parseMeasurements();
    this.renderChart(this.chartdata, this.options)
  },
  methods: {
    parseMeasurements() {
      this.chartdata = {
        labels: this.measurements.dates,
        datasets: [{
            label: `Pollution [${this.measurements.unit}]`,
            backgroundColor: '#f87979',
            data: this.measurements.values,
            borderWidth: 1
        }]
      };
      this.options = {
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
</style>

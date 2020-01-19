<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import { Doughnut } from 'vue-chartjs'
import os from 'os'

@Component({
  extends: Doughnut,
  data () {
    return {
      chartData: {
        labels: [
          'User Time (ms)',
          'System Time (ms)',
          'Idle Time (ms)'
        ]
      },
      options: {
        maintainAspectRatio: false,
        title: {
          display: true,
          text: 'CPU Activity',
          fontColor: 'rgb(250, 250, 250)',
          fontSize: 16
        },
        legend: {
          display: true,
          labels: {
            fontColor: 'rgb(250, 250, 250)',
            fontSize: 12
          }
        }
      },
      lastMeasureTimes: []
    }
  },
  mounted () {
    this.getDatasets()
    this.setLastMeasureTimes(os.cpus())
    this.renderChart(this.chartData, this.options)
    setInterval(this.updateDatasets, 2000)
  },
  methods: {
    getDatasets () {
      const datasets = []
      const cpus = os.cpus()

      for (let i = 0; i < cpus.length; i++) {
        const cpu = cpus[i]
        const cpuData = {
          data: this.getCpuTimes(cpu),
          backgroundColor: [
            'rgba(255, 99, 132, 1)',
            'rgba(54, 162, 235, 1)',
            'rgba(255, 206, 86, 1)'
          ]
        }
        datasets.push(cpuData)
      }
      // testCpus = os.cpus()
      this.chartData.datasets = datasets
    },
    setLastMeasureTimes (cpus) {
      for (let i = 0; i < cpus.length; i++) {
        this.lastMeasureTimes[i] = this.getCpuTimes(cpus[i])
      }
    },
    updateDatasets () {
      const cpus = os.cpus()
      for (let i = 0; i < cpus.length; i++) {
        const cpu = cpus[i]
        this.chartData.datasets[i].data = this.getCpuTimes(cpu)
        this.chartData.datasets[i].data[0] -= this.lastMeasureTimes[i][0]
        this.chartData.datasets[i].data[1] -= this.lastMeasureTimes[i][1]
        this.chartData.datasets[i].data[2] -= this.lastMeasureTimes[i][2]
      }
      this.$data._chart.update()
      this.setLastMeasureTimes(cpus)
    },
    getCpuTimes (cpu) {
      return [cpu.times.user, cpu.times.sys, cpu.times.idle]
    }
  }
})
export default class App extends Vue {}
</script>

<style lang="stylus">
#app
  font-family 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  text-align center
  color #2c3e50
  margin-top 60px

html, body, .container-fluid
  height 100%
  background-color #111

html
  -webkit-app-region drag

.container-fluid
  padding 25px
</style>

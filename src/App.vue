<template>
  <div>
    <date-picker :date="date" :option="option"></date-picker>
  </div>
</template>

<script>
import DatePicker from './vue2-material-datetimepicker'
import moment from 'moment'

export default {
  name: 'app',
  components: {
    'date-picker': DatePicker
  },
  data() {
    return {
      date: moment().format('YYYY-MM-DD'),
      option: {
        limit: [
          {
            type: 'from',
            date: moment().add(4, 'days').format('YYYY-MM-DD')
          },
          {
            type: 'to',
            date: moment().subtract(4, 'days').format('YYYY-MM-DD')
          }
        ]
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

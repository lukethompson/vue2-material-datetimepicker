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
      date: moment('2017-07-20 10:30').format('YYYY-MM-DD'),
      option: {
        limit: [
          {
            type: 'from',
            date: moment('2017-07-20 10:30').add(4, 'days').format('YYYY-MM-DD HH:mm')
          },
          {
            type: 'to',
            date: moment('2017-07-20 10:30').subtract(1, 'days').format('YYYY-MM-DD HH:mm')
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

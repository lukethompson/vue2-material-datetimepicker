<template>
  <div>
    <date-picker :date="date"></date-picker>
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
      date: {
        time: moment()
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

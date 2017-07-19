<template>
  <div class="vdp-datepicker">
    <div class="vdp-input-container">
      <input type="text" @click="show('picker')" @focus="show('picker')" :value="date.time.format('DD/MM/YYYY')" :title="option.inputTitle" class="vdp-input" readonly="readonly" />
    </div>
    <div class="vdp-overlay" v-if="showPicker.picker" @click="overlayClick($event)">
      <div class="vdp-dialog">
        <div class="vdp-dialog-header">
          <div class="vdp-dialog-title">
            <span class="vdp-selected-year" @click="show('year')">{{ titleMoment.format('YYYY') }}</span>
            <span class="vdp-selected-date" @click="setDisplayMoment(titleMoment)">{{ titleMoment.format('dddd') }}, {{ titleMoment.format('MMM') }} {{ titleMoment.date() }}</span>
          </div>
        </div>
        <div class="vdp-dialog-content">
          <div class="vdp-picker-container" v-if="showPicker.year">
            <div class="vdp-picker">
              <div class="vdp-date-list" ref="yearList">
                <ul>
                  <li class="year" v-for="(year,index) in years" :key="index" @click="setYear(year.value)" :class="{'checked':year.checked}" ref="year">{{ year.display }}</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="vdp-picker-container" v-if="showPicker.month">
            <div class="vdp-picker">
              <div class="vdp-date-list" ref="monthList">
                <ul>
                  <li class="month" v-for="(month,index) in months" :key="index" @click="setMonth(month.value)" :class="{'checked':month.checked}" ref="month">{{ month.display }}</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="vdp-picker-container" v-if="showPicker.day">
            <div class="vdp-month-selection">
              <div class="vdp-previous" @click="previousMonth()">«</div>
              <div class="vdp-month-selection-month" @click="show('month')">
                {{ displayMoment.format('MMM') + ' ' + displayMoment.format('YYYY') }}
              </div>
              <div class="vdp-next" @click="nextMonth()">»</div>
            </div>
            <div class="vdp-datepicker">
              <div class="vdp-weekday-labels">
                <ul>
                  <li v-for="weekLabel in weekLabels">{{ weekLabel }}</li>
                </ul>
              </div>
              <div class="vdp-day" v-for="(day,index) in days" @click="setSelectedMoment(day)" :key="index" :class="{'checked':day.checked,'unavailable':day.unavailable,'passive': !(day.inMonth)}">
                {{ day.value }}
              </div>
            </div>
          </div>
          <div class="vdp-picker-container" v-if="showPicker.time">
            <div class="vdp-picker">
              <div class="vdp-date-list vdp-hours">
                <ul>
                  <li v-for="hour in hours">{{ hour.value }}</li>
                </ul>
              </div>
              <div class="vdp-date-list vdp-minutes">
                <ul>
                  <li v-for="minute in minutes">{{ minute.value }}</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="vdp-dialog-footer">
            <span class="vdp-btn" @click="close()">Cancel</span>
            <span class="vdp-btn" @click="save()">OK</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import moment from 'moment'

  export default {
    computed: {
      titleMoment: function () {
        let titleMoment = this.selectedMoment !== null
          ? this.selectedMoment
          : this.defaultMoment

        return titleMoment
      }
    },
    data() {
      function getHours() {
        let list = []
        let hour = 0
        while (hour < 24) {
          list.push({
            checked: false,
            value: hour < 10 ? '0' + hour : hour
          })
          hour++
        }

        return list
      }

      function getMinutes() {
        let list = []
        let minute = 0
        while (minute < 60) {
          list.push({
            checked: false,
            value: minute < 10 ? '0' + minute : minute
          })
          minute++
        }

        return list
      }

      function getWeekLabels() {
        let weekLabels = ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su']
        return weekLabels
      }

      return {
        days: null,
        displayMoment: null,
        hours: getHours(),
        minutes: getMinutes(),
        months: null,
        selectedMoment: null,
        showPicker: {
          day: false,
          month: false,
          picker: false,
          year: false,
          time: false
        },
        years: null,
        weekLabels: getWeekLabels()
      }
    },
    methods: {
      close () {
        this.show('none')
      },
      nextMonth () {
        this.displayMoment = moment(this.displayMoment).add(1, 'month')
      },
      previousMonth () {
        this.displayMoment = moment(this.displayMoment).subtract(1, 'month')
      },
      overlayClick (event) {
        if (event.target.className === 'vdp-overlay') {
          this.close()
        }
      },
      save () {
        this.date.time = this.selectedMoment
        this.close(true)
      },
      scrollList(listRef, itemRef) {
        let displayMoment = this.displayMoment
        let listItemHeight = this.$refs[itemRef][0].offsetHeight
        let scrollTop = 0
        this.years.map(function(year, index) {
          if (year.value === moment(displayMoment).year())
            scrollTop = listItemHeight * index
        })

        let listDOM = this.$refs
        listDOM[listRef].scrollTop = scrollTop
      },
      setDisplayMoment(displayMoment) {
        this.displayMoment = displayMoment
        this.show('picker')
      },
      setDays() {
        let currentMoment = this.displayMoment
        let firstDay = moment(currentMoment).date(1).day()
        let previousMonth = moment(currentMoment).subtract(1, 'month')
        let nextMonth = moment(currentMoment).add(1, 'month')

        let currentMonthDays = currentMoment.daysInMonth()

        let days = []
        for (let i = 0; i < currentMonthDays; ++i) {
          days.push({
            value: i + 1,
            inMonth: true,
            unavailable: false,
            checked: false,
            moment: moment(currentMoment).date(i + 1)
          })

          if (days[i].moment.format('YYYY-MM-DD') === this.selectedMoment.format('YYYY-MM-DD'))
            days[i].checked = true
        }

        if (firstDay === 0) 
          firstDay = 7

        for (let i = 0; i < firstDay - 1; i++) {
          let passiveDay = {
            value: previousMonth.daysInMonth() - i,
            inMonth: false,
            action: 'previous',
            unavailable: false,
            checked: false,
            moment: moment(currentMoment).date(1).subtract(i + 1, 'days')
          }
          days.unshift(passiveDay)
        }

        let passiveDaysAtEnd = 42 - days.length
        for (let i = 0; i < passiveDaysAtEnd; i++) {
          let passiveDay = {
            value: i + 1,
            inMonth: false,
            action: 'next',
            unavailable: false,
            checked: false,
            moment: moment(nextMonth).add(1, 'months').date(i + 1)
          }
          days.push(passiveDay)
        }

        this.days = days
      },
      setMonth(month) {
        this.displayMoment = moment(this.displayMoment).set('month', month)
        this.show('picker')
      },
      setMonths() {
        let currentMoment = this.displayMoment
        let selectedMoment = this.selectedMoment

        let list = []
        let months = moment.months()
        for (let i = 0; i < months.length; i++) {
          list.push({
            checked: false,
            display: months[i],
            value: i + 1
          })

          if (list[i].value == moment(selectedMoment).month())
            list[i].checked = true
        }

        this.months = list
      },
      setSelectedMoment(day) {
        this.selectedMoment = day.moment
        this.displayMoment = day.moment
      },
      setYear(year) {
        this.displayMoment =  moment(this.displayMoment).set('year', year)
        this.show('picker')
      },
      setYears() {
        let currentMoment = this.displayMoment
        let selectedMoment = this.selectedMoment

        let list = []
        let year = moment(currentMoment).year()
        let yearMax = year + 10
  
        year = year - 10
        while (year < yearMax) {
          let data = {
            checked: false,
            display: year,
            value: year
          }

          if (year == moment(selectedMoment).year())
            data.checked = true

          list.push(data)
          year++
        }

        this.years = list
      },
      show(type) {
        switch (type) {
          case 'picker':
            this.showPicker.day = true
            this.showPicker.month = false
            this.showPicker.picker = true
            this.showPicker.year = false
            this.showPicker.time = false
            break;

          case 'month':
            this.showPicker.day = false
            this.showPicker.month = true
            this.showPicker.picker = true
            this.showPicker.year = false
            this.showPicker.time = false

            this.$nextTick(function() {
              this.scrollList('monthList', 'month')
            })
            break;

          case 'year':
            this.showPicker.day = false
            this.showPicker.month = false
            this.showPicker.picker = true
            this.showPicker.year = true
            this.showPicker.time = false

            this.$nextTick(function() {
              this.scrollList('yearList', 'year')
            })
            break;
          
          case 'none': 
            this.showPicker.day = false
            this.showPicker.month = false
            this.showPicker.picker = false
            this.showPicker.year = false
            this.showPicker.time = false
            break;
        }
      }
    },
    props: {
      required: false,
      date: {
        type: Object,
        required: true
      },
      option: {
        type: Object,
        default() {
          return {

          }
        }
      }
    },
    created: function() {
      if (!(this.date.time instanceof moment)) {
        this.date.time = moment(this.date.time)
      }

      this.displayMoment = this.date.time
      this.selectedMoment = this.date.time
    },
    watch: {
      'displayMoment': function() {
        this.setDays()
        this.setMonths()
        this.setYears()
      }
    }
  }
</script>

<style scoped>
  .vdp-datepicker { display: inline-block; }
  .vdp-overlay { height: 100vh; left: 0; position: fixed; top: 0; width: 100vw; z-index: 998;
    -webkit-animation: fadein 0.5s;
    -moz-animation: fadein 0.5s;
    -ms-animation: fadein 0.5s;
    -o-animation: fadein 0.5s;
    animation: fadein 0.5s;
  }
  .vdp-dialog { background: #fff; box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.2); display: block; font-family: 'Roboto'; font-weight: 400; left: 50%; max-width: 100%; position: absolute; top: 30%; width: 400px; z-index: 999;
    -webkit-transform: translate(-50%, -50%);
    -ms-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }
  .vdp-dialog-header { background: #3F51B5; color: #fff; height: 120px; text-align: center; }
  .vdp-dialog-title { box-sizing: border-box; display: inline-block; font-size: 16px; height: 100%; padding: 30px 0; text-align: left; }
  .vdp-dialog-content { box-sizing: border-box; width: 100%; }
  .vdp-dialog-footer { padding: 10px 25px 20px 25px; text-align: right; }

  .vdp-picker-container { box-sizing: border-box; height: 320px; overflow: hidden; }
  .vdp-picker { height: 100%; }
  .vdp-date-list { height: 100%; overflow-y: scroll; }
  .vdp-date-list ul { list-style: none; margin-top: 0; padding: 0; text-align: center; }
  .vdp-date-list ul li { cursor: pointer; padding: 10px 0; }
  .vdp-date-list ul li:hover { background: #e0e0e0; }

  .vdp-btn { cursor: pointer; padding: 0 10px; }
  
  .vdp-datepicker { padding: 0 25px; }
  
  .vdp-month-selection { font-size: 0; height: 30px; line-height: 30px; margin: 25px 0; padding: 0 12px; }
  .vdp-previous,
  .vdp-next { color: #fff; display: inline-block; cursor: pointer; height: 100%; overflow: hidden; position: relative; text-indent: -300px; width: 15% !important; }
  .vdp-previous:after,
  .vdp-next:after { background: #000; content: ''; height: 2px; position: absolute; line-height: 0; margin-top: 6px; right: 0; text-align: center; top: 50%; width: 20px;
    -webkit-transform: rotate(-45deg);
    -moz-transform: rotate(-45deg);
    transform: rotate(-45deg);
  }
  .vdp-previous:before,
  .vdp-next:before { background: #000; content: ''; height: 2px; line-height: 0; margin-top: -7px; position: absolute; right: 0; text-align: center; top: 50%; width: 20px;
    -webkit-transform: rotate(45deg);
    -moz-transform: rotate(45deg);
    transform: rotate(45deg);
  }
  .vdp-month-selection-month { cursor: pointer; display: inline-block; font-size: 18px; text-align: center; vertical-align: top; width: 70%; }
  .vdp-previous::after { left: 0; right: auto;
    -webkit-transform: rotate(45deg);
    -moz-transform: rotate(45deg);
    transform: rotate(45deg);
  }
  .vdp-previous::before { left: 0; right: auto;
    -webkit-transform: rotate(-45deg);
    -moz-transform: rotate(-45deg);
    transform: rotate(-45deg);
  }
  .vdp-weekday-labels { }
  .vdp-weekday-labels ul { list-style: none; margin: 0; padding-left: 0; text-align: center; }
  .vdp-weekday-labels ul li { color: #000; display: inline-block; font-weight: bold; text-align: center; width: 14.2%; }
  .vdp-day { color: #000; cursor: pointer; display: inline-block; height: 34px; line-height: 34px; padding: 0; vertical-align: middle; text-align: center; width: 14.2%; }
  .vdp-day.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .vdp-day.checked:hover { background: #FF4F8E; }
  .vdp-day.passive { color: #bbb; }
  .vdp-day:hover { background: #eaeaea; }
  .month.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .month.checked:hover { background: #FF4F8E; }
  .year.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .year.checked:hover { background: #FF4F8E; }
  .vdp-selected-date { cursor: pointer; font-size: 28px; }
  .vdp-selected-year { cursor: pointer; display: block; font-weight: 200; margin-bottom: 5px; }
  ::-webkit-scrollbar { width: 2px; }
  ::-webkit-scrollbar-track { background: #E3E3E3; }
  ::-webkit-scrollbar-thumb { background: #C1C1C1; border-radius: 2px; }

  @keyframes fadein {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  /* Firefox < 16 */
  @-moz-keyframes fadein {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  /* Safari, Chrome and Opera > 12.1 */
  @-webkit-keyframes fadein {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  /* Internet Explorer */
  @-ms-keyframes fadein {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  /* Opera < 12.1 */
  @-o-keyframes fadein {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
</style>
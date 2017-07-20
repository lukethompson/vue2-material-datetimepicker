<template>
  <div class="vmdtp-datepicker">
    <div class="vmdtp-input-container">
      <input type="text" @click="show('picker')" @focus="show('picker')" :value="savedMoment.format('YYYY-MM-DD HH:mm')" :title="option.inputTitle" class="vmdtp-input" readonly="readonly" />
    </div>
    <div class="vmdtp-overlay" v-if="showPicker.picker" @click="overlayClick($event)">
      <div class="vmdtp-dialog">
        <div class="vmdtp-dialog-header">
          <div class="vmdtp-dialog-title">
            <span class="vmdtp-selected-year" @click="show('year')">{{ titleMoment.format('YYYY') }}</span>
            <span class="vmdtp-selected-date" @click="setDisplayMoment(titleMoment)">{{ titleMoment.format('dddd') }}, {{ titleMoment.format('MMM') }} {{ titleMoment.date() }}</span>
            <span class="vmdtp-selected-time" @click="show('time')">{{ titleMoment.format('HH:mm') }}</span>
          </div>
        </div>
        <div class="vmdtp-dialog-content">
          <div class="vmdtp-picker-container" v-if="showPicker.year">
            <div class="vmdtp-picker">
              <div class="vmdtp-date-list" ref="yearList">
                <ul>
                  <li class="year" v-for="(year,index) in years" :key="index" @click="setYear(year.value)" :class="{'checked':year.checked,'unavailable':year.unavailable}" ref="year">{{ year.display }}</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="vmdtp-picker-container" v-if="showPicker.month">
            <div class="vmdtp-picker">
              <div class="vmdtp-date-list" ref="monthList">
                <ul>
                  <li class="month" v-for="(month,index) in months" :key="index" @click="setMonth(month.value)" :class="{'checked':month.checked,'unavailable':month.unavailable}" ref="month">{{ month.display }}</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="vmdtp-picker-container" v-if="showPicker.day">
            <div class="vmdtp-month-selection">
              <div class="vmdtp-previous" @click="previousMonth()">«</div>
              <div class="vmdtp-month-selection-month" @click="show('month')">
                {{ displayMoment.format('MMM') + ' ' + displayMoment.format('YYYY') }}
              </div>
              <div class="vmdtp-next" @click="nextMonth()">»</div>
            </div>
            <div class="vmdtp-datepicker">
              <div class="vmdtp-weekday-labels">
                <ul>
                  <li v-for="weekLabel in weekLabels">{{ weekLabel }}</li>
                </ul>
              </div>
              <div class="vmdtp-day" v-for="(day,index) in days" @click="setSelectedMoment(day)" :key="index" :class="{'checked':day.checked,'unavailable':day.unavailable,'passive': !(day.inMonth)}">
                {{ day.value }}
              </div>
            </div>
          </div>
          <div class="vmdtp-picker-container" v-if="showPicker.time">
            <div class="vmdtp-picker">
              <div class="vmdtp-date-list vmdtp-hours" ref="hourList">
                <input name="hour" type="number" ref="hourInput" v-model="hourInputModel" max="23" min="0" />
                <ul>
                  <li class="hour" ref="hour" v-for="hour in hours" @click="setHour(hour.value)" :class="{'checked':hour.checked,'unavailable':hour.unavailable}">{{ hour.value }}</li>
                </ul>
              </div>
              <div class="vmdtp-date-list vmdtp-minutes" ref="minuteList">
                <input name="minute" type="number" ref="minuteInput" v-model="minuteInputModel" max="59" min="0" />
                <ul>
                  <li class="minute" ref="minute" v-for="minute in minutes" @click="setMinute(minute.value)" :class="{'checked':minute.checked,'unavailable':minute.unavailable}">{{ minute.value }}</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="vmdtp-dialog-footer">
            <span class="vmdtp-btn" @click="close()">Cancel</span>
            <span class="vmdtp-btn" @click="save()">OK</span>
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
      return {
        days: null,
        displayMoment: null,
        hours: null,
        hourInputModel: null,
        minuteInputModel: null,
        minutes: null,
        months: null,
        savedMoment: moment(this.date),
        selectedMoment: null,
        showPicker: {
          day: false,
          month: false,
          picker: false,
          year: false,
          time: false
        },
        years: null,
        weekLabels: null
      }
    },
    methods: {
      close () {
        this.show('none')
      },
      getItemDisplayMoment (item) {
        return moment(this.displayMoment).set(item.type, item.value)
      },
      isItemDisabled(item, limit) {
        if (typeof item.moment == 'undefined')
          item.moment = this.getItemDisplayMoment(item)

        switch (limit.type) {
          case 'from':
            return item.moment.isAfter(limit.date)

          case 'to':
            return item.moment.isBefore(limit.date)
        }
      },
      limitItems (items, limit) {
        items.map((item) => {
          if (this.isItemDisabled(item, limit))
            item.unavailable = true

        })

        return items
      },
      nextMonth () {
        this.displayMoment = moment(this.displayMoment).add(1, 'month')
      },
      pad(value) {
        return value < 10 ? '0' + value : value
      },
      previousMonth () {
        this.displayMoment = moment(this.displayMoment).subtract(1, 'month')
      },
      overlayClick (event) {
        if (event.target.className === 'vmdtp-overlay') {
          this.close()
        }
      },
      save () {
        this.savedMoment = moment(this.selectedMoment)
        this.close(true)
      },
      scrollList(items, selectedItem, listRef, itemRef, setSelected=false) {
        let listItemHeight = this.$refs[itemRef][0].offsetHeight
        let scrollTop = 0

        let that = this
        items.map(function(item, index) {
          if (item.value == selectedItem) {
            scrollTop = listItemHeight * index
            if (setSelected == 'hour')
              that.setHour(item.value)

            if (setSelected == 'minute')
              that.setMinute(item.value)
          }
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

          if (this.selectedMoment !== null && days[i].moment.format('YYYY-MM-DD') === this.selectedMoment.format('YYYY-MM-DD'))
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

        if (this.option.limit.length > 0) {
          for (let limit of this.option.limit) {
            days = this.limitItems(days, limit)
          }
        }

        this.days = days
      },
      setHour(hour) {
        let newSelectedMoment = moment(this.selectedMoment).set('hour', hour)

        this.displayMoment = newSelectedMoment
        this.selectedMoment = newSelectedMoment
        this.show('time')
      },
      setHours() {
        let selectedMoment = this.selectedMoment

        let list = []
        let hour = 0
        while (hour < 24) {
          let display = this.pad(hour)
          let data = {
            checked: false,
            display: display,
            type: 'hour',
            unavailable: false,
            value: hour,
          }
          if (hour == moment(selectedMoment).hour())
            data.checked = true

          list.push(data)
          hour++
        }

        if (this.option.limit.length > 0) {
          for (let limit of this.option.limit) {
            list = this.limitItems(list, limit)
          }
        }

        this.hours = list
      },
      setMinute(minute) {
        let newSelectedMoment = moment(this.selectedMoment).set('minute', minute)

        this.displayMoment = newSelectedMoment
        this.selectedMoment = newSelectedMoment
        this.show('time')
      },
      setMinutes() {
        let selectedMoment = this.selectedMoment

        let list = []
        let minute = 0
        while (minute < 60) {
          let display = this.pad(minute)
          let data = {
            checked: false,
            display: display,
            type: 'minute',
            value: minute
          }

          if (minute == moment(selectedMoment).minute())
            data.checked = true

          list.push(data)
          minute++
        }

        if (this.option.limit.length > 0) {
          for (let limit of this.option.limit) {
            list = this.limitItems(list, limit)
          }
        }

        this.minutes = list
      },
      setMonth(month) {
        this.setDisplayMoment(moment(this.displayMoment).set('month', month))
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
            type: 'month',
            value: i + 1
          })

          if (list[i].value == moment(selectedMoment).month())
            list[i].checked = true
        }

        if (this.option.limit.length > 0) {
          for (let limit of this.option.limit) {
            list = this.limitItems(list, limit)
          }
        }

        this.months = list
      },
      setSelectedMoment(day) {
        if (day.unavailable)
          return false

        this.displayMoment = day.moment
        this.selectedMoment = day.moment
      },
      setYear(year) {
        this.setDisplayMoment(moment(this.displayMoment).set('year', year))
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
            type: 'year',
            value: year
          }

          if (year == moment(selectedMoment).year())
            data.checked = true

          list.push(data)
          year++
        }

        if (this.option.limit.length > 0) {
          for (let limit of this.option.limit) {
            list = this.limitItems(list, limit)
          }
        }

        this.years = list
      },
      setWeekLabels() {
        this.weekLabels = moment.weekdaysShort()
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
              this.scrollList(this.months, moment(this.selectedMoment).month(), 'monthList', 'month')
            })
            break;

          case 'time':
            this.showPicker.day = false
            this.showPicker.month = false
            this.showPicker.picker = true
            this.showPicker.year = false
            this.showPicker.time = true
            this.$nextTick(function() {
              this.scrollList(this.hours, moment(this.selectedMoment).hour(), 'hourList', 'hour')
              this.scrollList(this.minutes, moment(this.selectedMoment).minute(),'minuteList', 'minute')

              this.hourInputModel === null || this.hourInputModel.length < 2
                ? this.$refs.hourInput.focus()
                : this.$refs.minuteInput.focus()

              if (this.minuteInputModel !== null && this.minuteInputModel.length >= 2)
                this.$refs.minuteInput.blur()

            })
            break;

          case 'year':
            this.showPicker.day = false
            this.showPicker.month = false
            this.showPicker.picker = true
            this.showPicker.year = true
            this.showPicker.time = false

            this.$nextTick(function() {
              this.scrollList(this.years, moment(this.selectedMoment).year(), 'yearList', 'year')
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
        type: String,
        required: false
      },
      option: {
        type: Object,
        default() {
          return {
            default: moment(),
            limit: []
          }
        }
      }
    },
    created: function() {
      this.defaultMoment = typeof this.date != 'undefined'
        ? moment(this.date)
        : moment(this.option.default)

      this.setWeekLabels()
      this.selectedMoment = typeof this.date != 'undefined'
        ? moment(this.date)
        : null

      this.displayMoment = this.defaultMoment
    },
    watch: {
      'displayMoment': function() {
        this.setDays()
        this.setHours()
        this.setMinutes()
        this.setMonths()
        this.setYears()
      },
      'hourInputModel': function(value) {
        if (value > 23)
          this.hourInputModel = 23

        this.scrollList(this.hours, this.hourInputModel, 'hourList', 'hour', 'hour')
      },
      'minuteInputModel': function(value) {
        if (value > 59)
          this.minuteInputModel = 59

        this.scrollList(this.minutes, this.minuteInputModel, 'minuteList', 'minute', 'minute')
      }
    }
  }
</script>

<style scoped>
  .vmdtp-datepicker { display: inline-block; }
  .vmdtp-overlay { height: 100vh; left: 0; position: fixed; top: 0; width: 100vw; z-index: 998;
    -webkit-animation: fadein 0.5s;
    -moz-animation: fadein 0.5s;
    -ms-animation: fadein 0.5s;
    -o-animation: fadein 0.5s;
    animation: fadein 0.5s;
  }
  .vmdtp-dialog { background: #fff; box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.2); display: block; font-family: 'Roboto'; font-weight: 400; left: 50%; max-width: 100%; position: absolute; top: 30%; width: 400px; z-index: 999;
    -webkit-transform: translate(-50%, -50%);
    -ms-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }
  .vmdtp-dialog-header { background: #3F51B5; color: #fff; height: 120px; text-align: center; }
  .vmdtp-dialog-title { box-sizing: border-box; display: inline-block; font-size: 16px; height: 100%; padding: 25px 0; text-align: left; }
  .vmdtp-dialog-content { box-sizing: border-box; width: 100%; }
  .vmdtp-dialog-footer { padding: 10px 25px 20px 25px; text-align: right; }

  .vmdtp-picker-container { box-sizing: border-box; height: 320px; overflow: hidden; }
  .vmdtp-picker { font-size: 0; height: 100%; }
  .vmdtp-date-list { font-size: 16px; height: 100%; overflow-y: scroll; }
  .vmdtp-date-list input { position: absolute; left: -9999px; }
  .vmdtp-date-list ul { list-style: none; margin-top: 0; padding: 0; text-align: center; }
  .vmdtp-date-list ul li { cursor: pointer; padding: 10px 0; }
  .vmdtp-date-list ul li:hover { background: #e0e0e0; }
  .vmdtp-date-list.vmdtp-hours { display: inline-block; width: 50%; }
  .vmdtp-date-list.vmdtp-minutes { display: inline-block; width: 50%; }

  .vmdtp-btn { cursor: pointer; padding: 0 10px; }
  
  .vmdtp-datepicker { padding: 0 25px; }
  
  .vmdtp-month-selection { font-size: 0; height: 30px; line-height: 30px; margin: 25px 0; padding: 0 12px; }
  .vmdtp-previous,
  .vmdtp-next { color: #fff; display: inline-block; cursor: pointer; height: 100%; overflow: hidden; position: relative; text-indent: -300px; width: 15% !important; }
  .vmdtp-previous:after,
  .vmdtp-next:after { background: #000; content: ''; height: 2px; position: absolute; line-height: 0; margin-top: 6px; right: 0; text-align: center; top: 50%; width: 20px;
    -webkit-transform: rotate(-45deg);
    -moz-transform: rotate(-45deg);
    transform: rotate(-45deg);
  }
  .vmdtp-previous:before,
  .vmdtp-next:before { background: #000; content: ''; height: 2px; line-height: 0; margin-top: -7px; position: absolute; right: 0; text-align: center; top: 50%; width: 20px;
    -webkit-transform: rotate(45deg);
    -moz-transform: rotate(45deg);
    transform: rotate(45deg);
  }
  .vmdtp-month-selection-month { cursor: pointer; display: inline-block; font-size: 18px; text-align: center; vertical-align: top; width: 70%; }
  .vmdtp-previous::after { left: 0; right: auto;
    -webkit-transform: rotate(45deg);
    -moz-transform: rotate(45deg);
    transform: rotate(45deg);
  }
  .vmdtp-previous::before { left: 0; right: auto;
    -webkit-transform: rotate(-45deg);
    -moz-transform: rotate(-45deg);
    transform: rotate(-45deg);
  }
  .vmdtp-weekday-labels ul { list-style: none; margin: 0; padding-left: 0; text-align: center; }
  .vmdtp-weekday-labels ul li { color: #000; display: inline-block; font-weight: bold; text-align: center; width: 14.2%; }
  .vmdtp-day { color: #000; cursor: pointer; display: inline-block; height: 34px; line-height: 34px; padding: 0; vertical-align: middle; text-align: center; width: 14.2%; }
  .vmdtp-day.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .vmdtp-day.checked:hover { background: #FF4F8E; }
  .vmdtp-day.passive { color: #bbb; }
  .vmdtp-day:hover { background: #eaeaea; }
  .hour.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .hour.checked:hover { background: #FF4F8E; }
  .minute.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .minute.checked:hover { background: #FF4F8E; }
  .month.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .month.checked:hover { background: #FF4F8E; }
  .year.checked { background: #F50057; color: #FFF !important; border-radius: 3px; }
  .year.checked:hover { background: #FF4F8E; }
  .unavailable { color: #ccc; cursor: not-allowed !important; }
  .vmdtp-selected-date { cursor: pointer; font-size: 28px; }
  .vmdtp-selected-time { cursor: pointer; display: block; font-size: 14px; margin-top: 5px; text-align: right; }
  .vmdtp-selected-year { cursor: pointer; display: block; font-weight: 200; margin-bottom: 5px; }
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
<template>
<div class="scheduler-container">
  <div>
    <select v-model="selectedView">
      <option v-for="(options, index) in viewModeOptions" :value="options.value" :key="index">
        {{options.title}}
      </option>
    </select>
    <span @click="onClickNavi($event)">
      <button type="button" class="btn btn-default btn-sm move-today" data-action="move-today">Today</button>
      <button type="button" class="btn btn-default btn-sm move-day" data-action="move-prev">Prev</button>
      <button type="button" class="btn btn-default btn-sm move-day" data-action="move-next">Next</button>
    </span>
    <span class="render-range">{{dateRange}}</span>
  </div>
  <calendar style="height: 800px"
            ref="tuiCal"
            :useDetailPopup="useDetailPopup"
            :view="selectedView"
            :calendars="calendarList"
            :schedules="scheduleList"
            :theme="theme"
            :template="template"
            :taskView="taskView"
            :scheduleView="scheduleView"
            :month="month"
            :week="week"
            :timezones="timezones"
            :disableDblClick="disableDblClick"
            :isReadOnly="isReadOnly"
            @clickSchedule="onClickSchedule"
            @clickDayname="onClickDayname"
            @beforeDeleteSchedule="onBeforeDeleteSchedule"
            @afterRenderSchedule="onAfterRenderSchedule"
            @clickTimezonesCollapseBtn="onClickTimezonesCollapseBtn"
            @beforeCreateSchedule="createSchedule"
            @beforeUpdateSchedule="updateSchedule"
  />
</div>
</template>

<script>
import { Calendar } from '@toast-ui/vue-calendar'

import 'tui-calendar/dist/tui-calendar.css'

// If you use the default popups, use this.
import 'tui-time-picker/dist/tui-time-picker.css'
import 'tui-date-picker/dist/tui-date-picker.css'
// import './scheduler.css'
import myTheme from './myTheme.js'
import moment from 'moment'

const today = new Date()
const getDate = (type, start, value, operator) => {
  start = new Date(start)
  type = type.charAt(0).toUpperCase() + type.slice(1)
  if (operator === '+') {
    start[`set${type}`](start[`get${type}`]() + value)
  } else {
    start[`set${type}`](start[`get${type}`]() - value)
  }
  return start
}

const getTimeTemplate = (schedule, isAllDay) => {
  const html = []

  if (!isAllDay) {
    html.push('<strong>' + moment(schedule.start.getTime()).format('HH:mm') + '</strong> ')
  }
  if (schedule.isPrivate) {
    html.push('<span class="calendar-font-icon ic-lock-b"></span>')
    html.push(' Private')
  } else {
    if (schedule.isReadOnly) {
      html.push('<span class="calendar-font-icon ic-readonly-b"></span>')
    } else if (schedule.recurrenceRule) {
      html.push('<span class="calendar-font-icon ic-repeat-b"></span>')
    } else if (schedule.attendees.length) {
      html.push('<span class="calendar-font-icon ic-user-b"></span>')
    } else if (schedule.location) {
      html.push('<span class="calendar-font-icon ic-location-b"></span>')
    }
    html.push(' ' + schedule.title)
  }

  return html.join('')
}

export default {
  components: {
    Calendar
  },
  data () {
    return {
      viewModeOptions: [
        {
          title: 'Monthly',
          value: 'month'
        },
        {
          title: 'Weekly',
          value: 'week'
        },
        {
          title: 'Daily',
          value: 'day'
        }
      ],
      dateRange: '',
      selectedView: 'month',
      // calendars[] : 달력의 type 리스트 (색상도 지정이 가능하다)
      calendarList: [
        {
          id: '0',
          name: '인표',
          bgColor: '#9e5fff',
          borderColor: '#9e5fff',
          dragBgColor: '#9e5fff'
        },
        {
          id: '1',
          name: '회사',
          bgColor: '#00a9ff',
          borderColor: '#00a9ff',
          dragBgColor: '#00a9ff'
        },
        {
          id: '3',
          name: '생일',
          bgColor: '#03bd9e',
          borderColor: '#03bd9e',
          dragBgColor: '#03bd9e'
        }

      ],
      // schedules[] : 달력의 스케쥴리스트 (스테쥴을 보관하는 장소, data가 변경되면 달력은 자동으로 변경됨)
      scheduleList: [
        {
          id: '1',
          calendarId: '0',
          title: '집청소',
          category: 'time',
          dueDateClass: '',
          start: today.toISOString(),
          end: getDate('hours', today, 3, '+').toISOString(),
          raw: { class: "private" }, 
        },
        {
          id: '2',
          calendarId: '1',
          title: '기사시험',
          category: 'milestone',
          dueDateClass: '',
          start: getDate('date', today, 1, '+').toISOString(),
          end: getDate('date', today, 1, '+').toISOString(),
          isReadOnly: true
        },
        {
          id: '3',
          calendarId: '1',
          title: '테스트',
          category: 'allday',
          dueDateClass: '',
          start: getDate('date', today, 2, '-').toISOString(),
          end: getDate('date', today, 1, '-').toISOString()
          // isReadOnly: true
        },
        {
          id: '4',
          calendarId: '1',
          title: '책상설치',
          category: 'time',
          dueDateClass: '',
          start: today.toISOString(),
          end: getDate('hours', today, 1, '+').toISOString()
        },
        {
          id: '5',
          calendarId: '3',
          title: '프라이빗테스트',
          category: 'time',
          dueDateClass: '',
          start: getDate('date', today, 1, '+').toISOString(),
          end: getDate('date', today, 2, '+').toISOString(),
          recurrenceRule: '반복됨'
        }
      ],
      timezones: [{
        timezoneOffset: 540,
        displayLabel: 'GMT+09:00',
        tooltip: 'Seoul'
      }, {
        timezoneOffset: -420,
        displayLabel: 'GMT-08:00',
        tooltip: 'Los Angeles'
      }],
      // theme은 달력 스타일(css)
      theme: myTheme,
      template: {
        milestone (schedule) {
          return `<span style="color:#fff;background-color: ${schedule.bgColor};">${schedule.title}</span>`
        },
        milestoneTitle () {
          return 'Milestone'
        },
        allday (schedule) {
          return `${schedule.title}<i class="fa fa-refresh"></i>`
        },
        alldayTitle () {
          return '올 Day'
        },
        popupIsAllDay () {
          return 'All Day'
        },
        popupStateFree () {
          return 'Free'
        },
        popupStateBusy () {
          return 'Busy'
        },
        titlePlaceholder () {
          return '제목'
        },
        locationPlaceholder () {
          return '위치'
        },
        startDatePlaceholder () {
          return 'Start date'
        },
        endDatePlaceholder () {
          return 'End date'
        },
        popupSave () {
          return '저장'
        },
        popupEdit () {
          return '수정'
        },
        popupDelete () {
          return '삭제'
        },
        time (schedule) {
          return getTimeTemplate(schedule, false)
        }

      },
      // month 커스터마이징
      month: {
        daynames: ['일', '월', '화', '수', '목', '금', '토'],
        startDayOfWeek: 0
      },
      // week and day 커스터마이징
      week: {
        showTimezoneCollapseButton: true,
        timezonesCollapsed: true
      },
      // taskView속성은 일간/주간 뷰에서 마일스톤/태스크 영역의 노출 여부를 설정합니다. 기본값이 true
      // (weekly,daily view에서만) : boolean, ['mileston'] or ['task'].
      taskView: true,
      // scheduleView(weekly,daily view에서만) : boolean, ['allday'] or ['time'].
      scheduleView: true,
      // useDetailPopup: : 이벤트 클릭시 이벤트 정보 보여주는 팝업
      useDetailPopup: true,
      // 스케쥴생성 팝업 띄우고 닫으려면, 더블클릭을 막아야한다
      disableDblClick: true,
      // readonly
      isReadOnly: false
    }
  },
  mounted () {
    this.init()
  },
  watch: {
    selectedView (newValue) {
      // 현재화면을 다음의 화면으로 전환('day', 'week', 'month')
      console.log(newValue)
      this.$refs.tuiCal.invoke('changeView', newValue, true)
      this.setRenderRangeText()
    }
  },
  methods: {
    init () {
      this.setRenderRangeText()
    },
    setRenderRangeText () {
      const { invoke } = this.$refs.tuiCal
      const view = invoke('getViewName')
      const calDate = invoke('getDate')
      const rangeStart = invoke('getDateRangeStart')
      const rangeEnd = invoke('getDateRangeEnd')
      let year = calDate.getFullYear()
      let month = calDate.getMonth() + 1
      let date = calDate.getDate()
      let dateRangeText = ''
      let endMonth, endDate, start, end
      switch (view) {
        case 'month':
          dateRangeText = `${year}-${month}`
          break
        case 'week':
          year = rangeStart.getFullYear()
          month = rangeStart.getMonth() + 1
          date = rangeStart.getDate()
          endMonth = rangeEnd.getMonth() + 1
          endDate = rangeEnd.getDate()
          start = `${year}-${month}-${date}`
          end = `${endMonth}-${endDate}`
          dateRangeText = `${start} ~ ${end}`
          break
        default:
          dateRangeText = `${year}-${month}-${date}`
      }
      this.dateRange = dateRangeText
    },
    onClickNavi (event) {
      if (event.target.tagName === 'BUTTON') {
        const { target } = event
        let action = target.dataset ? target.dataset.action : target.getAttribute('data-action')
        action = action.replace('move-', '')
        console.log(action) // 버튼클릭시 무슨 이벤트인지 확인
        this.$refs.tuiCal.invoke(action)
        this.setRenderRangeText()
      }
    },
    onClickSchedule () {
      // console.group('onClickSchedule')
      // console.log('MouseEvent : ', res.event)
      // console.log('Calendar Info : ', res.calendar)
      // console.log('Schedule Info : ', res.schedule)
      // console.groupEnd()
     
     
     // const button = document.querySelectorAll('button')
      // const editButton = document.querySelector('.tui-full-calendar-popup-edit')
      // const editSpan = document.querySelectorAll('span.tui-full-calendar-content')[1]
      // console.log(editButton)
      // console.log(editSpan)
      // editSpan.addEventListener('click', () => {
      //     const privateButton = document.querySelectorAll('.tui-full-calendar-popup')
      //     console.log(privateButton)
      // })

      
      // res.schedule.isPrivate = true
    },
    onClickDayname (res) {
      // view : week, day
      console.group('onClickDayname')
      console.log(res.date)
      console.groupEnd()
    },
    onBeforeDeleteSchedule (res) {
      console.group('onBeforeDeleteSchedule')
      console.log('Schedule Info : ', res.schedule)
      console.groupEnd()
      // const idx = this.scheduleList.findIndex(item => item.id === res.schedule.id)
      // this.scheduleList.splice(idx, 1)
      const { schedule } = res
      this.$refs.tuiCal.invoke('deleteSchedule', schedule.id, schedule.calendarId)
    },
    onAfterRenderSchedule (res) {
      console.group('onAfterRenderSchedule')
      console.log('Schedule Info : ', res.schedule)
      console.groupEnd()
    },
    onClickTimezonesCollapseBtn (timezonesCollapsed) {
      // view : week, day
      console.group('onClickTimezonesCollapseBtn')
      console.log('Is Collapsed Timezone? ', timezonesCollapsed)
      console.groupEnd()
      if (timezonesCollapsed) {
        this.theme['week.timegridLeft.width'] = '100px'
        this.theme['week.daygridLeft.width'] = '100px'
      } else {
        this.theme['week.timegridLeft.width'] = '50px'
        this.theme['week.daygridLeft.width'] = '50px'
      }
    },
    createSchedule (e) {
      console.log(e)
      console.log(e.calendarId)
      console.log(e.title)
      console.log(e.state)
      console.log(e.isAllDay)
      console.log(e.start)
      console.log(e.end)
      console.log(e.location)

      this.$refs.tuiCal.invoke('createSchedules', [{
        id: String(Math.random() * 100000000000000000),
        calendarId: e.calendarId,
        title: e.title,
        state: e.state,
        category: e.isAllDay ? 'allday' : 'time',
        isAllDay: e.isAllDay,
        start: e.start,
        end: e.end,
        location: e.location, // 장소 정보도 입력할 수 있네요!
        isPrivate: e.isPrivate
      }])
    },
    updateSchedule (e) {
      console.log(e)
      const { schedule, changes } = e
      // Chainging private status by clicking lock button only gives 'null'.   
      // Expected behavior was giving true or false value. b
      console.log(changes)    // null
      console.log(e.isPrivate)


      this.$refs.tuiCal.invoke('updateSchedule', schedule.id, schedule.calendarId, changes)
    }

  }
}
</script>

<style>
  /* #tui-full-calendar-schedule-private{
    display: none;
  } */


  .btn {
    border-radius: 25px;
    border-color: #ddd;
  }

  .btn:hover {
    border: solid 1px #bbb;
    background-color: #fff;
  }

  .btn:active {
    background-color: #f9f9f9;
    border: solid 1px #bbb;
    outline: none;
  }

  .btn:disabled {
    background-color: #f9f9f9;
    border: solid 1px #ddd;
    color: #bbb;
  }

  .btn:focus:active, .btn:focus, .btn:active {
    outline: none;
  }

  .open > .dropdown-toggle.btn-default {
    background-color: #fff;
  }

  .dropdown-menu {
    top: 25px;
    padding: 3px 0;
    border-radius: 2px;
    border: 1px solid #bbb;
  }

  .dropdown-menu > li > a {
    padding: 9px 12px;
    cursor: pointer;
  }

  .dropdown-menu > li > a:hover {
    background-color: rgba(81, 92, 230, 0.05);
    color: #333;
  }

  .bi15 {
    width: 15px;
    height: 15px;
  }

  .scheduler-container {
    font-family: 'Noto Sans', sans-serif;
    color: '#333';
    font-size: 12px;
  }

    /** custom fontawesome - end */
  .move-today {
    padding: 0 16px;
    line-height: 30px;
  }

  .move-day {
    padding: 8px;
  }

  .render-range {
    padding-left: 12px;
    font-size: 19px;
    vertical-align: middle;
  }
</style>

<style>
  /** custom fontawesome - end */
  .calendar-icon {
    width: 14px;
    height: 14px;
  }

  #top {
    height: 49px;
    border-bottom: 1px solid #bbb;
    padding: 16px;
    font-size: 10px;
  }

  #lnb {
    position: absolute;
    width: 200px;
    top: 49px;
    bottom: 0;
    border-right: 1px solid #d5d5d5;
    padding: 12px 10px;
    background: #fafafa;
  }

  #right {
    position: absolute;
    left: 200px;
    top: 49px;
    right: 0;
    bottom: 0;
  }

  #lnb label {
    margin-bottom: 0;
    cursor: pointer;
  }

  .lnb-new-schedule {
    padding-bottom: 12px;
    border-bottom: 1px solid #e5e5e5;
  }

  .lnb-new-schedule-btn {
    height: 100%;
    font-size: 14px;
    background-color: #ff6618;
    color: #ffffff;
    border: 0;
    border-radius: 25px;
    padding: 10px 20px;
    font-weight: bold;
  }

  .lnb-new-schedule-btn:hover {
    height: 100%;
    font-size: 14px;
    background-color: #e55b15;
    color: #ffffff;
    border: 0;
    border-radius: 25px;
    padding: 10px 20px;
    font-weight: bold;
  }

  .lnb-new-schedule-btn:active {
    height: 100%;
    font-size: 14px;
    background-color: #d95614;
    color: #ffffff;
    border: 0;
    border-radius: 25px;
    padding: 10px 20px;
    font-weight: bold;
  }

  .lnb-calendars > div {
    padding: 12px 16px;
    border-bottom: 1px solid #e5e5e5;
    font-weight: normal;
  }

  .lnb-calendars-d1 {
    padding-left: 8px;
  }

  .lnb-calendars-d1 label {
    font-weight: normal;
  }

  .lnb-calendars-item {
    min-height: 14px;
    line-height: 14px;
    padding: 8px 0;
  }

  .lnb-footer {
    color: #999;
    font-size: 11px;
    position: absolute;
    bottom: 12px;
    padding-left: 16px;
  }

  #menu {
    padding: 16px;
  }

  #dropdownMenu-calendarType {
    padding: 0 8px 0 11px;
  }

  #calendarTypeName {
    min-width: 62px;
    display: inline-block;
    text-align: left;
    line-height: 30px;
  }

  .dropdown-menu-title .calendar-icon {
    margin-right: 8px;
  }

  .calendar-bar {
    width: 16px;
    height: 16px;
    margin-right: 5px;
    display: inline-block;
    border: 1px solid #eee;
    vertical-align: middle;
  }

  .calendar-name {
    font-size: 14px;
    font-weight: bold;
    vertical-align: middle;
  }

  .schedule-time {
    color: #005aff;
  }

  #calendar {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 5px;
    top: 64px;
  }

  /** custom fontawesome */

   .fa {
    width: 12px;
    height: 12px;
    margin-right: 2px;
  }

  .tui-full-calendar-month-week-item .tui-full-calendar-today .tui-full-calendar-weekday-grid-date-decorator {
    background-color: #009688;
  }
</style>

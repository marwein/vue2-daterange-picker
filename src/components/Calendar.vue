<template>
    <table class="table-condensed">
        <thead>
        <tr>
            <th class="prev available" @click="$emit('prevMonth')"><i :class="[arrowLeftClass]"></i></th>
            <th colspan="5" class="month">{{monthName}} {{year}}</th>
            <th class="next available" @click="$emit('nextMonth')"><i :class="[arrowRightClass]"></i></th>
        </tr>
        </thead>
        <tbody>
        <tr>
            <th v-for="weekDay in locale.daysOfWeek">{{weekDay}}</th>
        </tr>
        <tr v-for="dateRow in calendar">
            <slot name="date-slot" v-for="date in dateRow">
                <td :class="dayClass(date)" @click="$emit('dateClick', date)" @mouseover="$emit('hoverDate', date)">{{date | dateNum}}</td>
            </slot>
        </tr>
        </tbody>
    </table>
</template>

<script>
    import moment from 'moment'

    export default {
        name: 'calendar',
        props: ['monthDate', 'locale', 'start', 'end'],
        methods: {
            dayClass(date) {
                let dt = new Date(date)
                return {
                    off: date.month() !== this.month,
                    weekend: date.isoWeekday() > 5,
                    today: dt.setHours(0,0,0,0) == new Date().setHours(0,0,0,0),
                    active: dt.setHours(0,0,0,0) == new Date(this.start).setHours(0,0,0,0) || dt.setHours(0,0,0,0) == new Date(this.end).setHours(0,0,0,0),
                    'in-range': dt >= new Date(this.start).setHours(0,0,0,0) && dt <= new Date(this.end).setHours(0,0,0,0)
                }
            }
        },
        computed: {
            arrowLeftClass() {
                return 'fa fa-chevron-left glyphicon glyphicon-chevron-left'
            },
            arrowRightClass() {
                return 'fa fa-chevron-right glyphicon glyphicon-chevron-right'
            },
            monthName() {
                return this.locale.monthNames[this.monthDate.getMonth()]
            },
            year() {
                return this.monthDate.getFullYear()
            },
            month () {
                return this.monthDate.getMonth()
            },
            calendar () {
                let month = this.month
                let year = this.monthDate.getFullYear()
                let daysInMonth = new Date(year, month, 0).getDate()
                let firstDay = new Date(year, month, 1)
                let lastDay = new Date(year, month, daysInMonth)
                let lastMonth = moment(firstDay).subtract(1, 'month').month()
                let lastYear = moment(firstDay).subtract(1, 'month').year()
                let daysInLastMonth = moment([lastYear, lastMonth]).daysInMonth()

                let dayOfWeek = firstDay.getDay()

                let calendar = []

                for (let i = 0; i < 6; i++) {
                    calendar[i] = [];
                }

                let startDay = daysInLastMonth - dayOfWeek + this.locale.firstDay + 1
                if (startDay > daysInLastMonth)
                    startDay -= 7

                if (dayOfWeek === this.locale.firstDay)
                    startDay = daysInLastMonth - 6;

                let curDate = moment([lastYear, lastMonth, startDay, 12, 0, 0]);
                for (let i = 0, col = 0, row = 0; i < 6 * 7; i++, col++, curDate = moment(curDate).add(1, 'day')) {
                    if (i > 0 && col % 7 === 0) {
                        col = 0;
                        row++;
                    }
                    calendar[row][col] = curDate.clone()
                    curDate.hour(12);
                }

                return calendar
            }
        },
        filters: {
            dateNum(value) {
                return value.date()
            }
        }
    }
</script>

<style>
    td.today {
        font-weight: bold;
    }
</style>
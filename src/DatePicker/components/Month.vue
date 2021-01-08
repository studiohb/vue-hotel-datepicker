<template>
  <div class="vhd__datepicker__month">
    <p class="vhd__datepicker__month-name">
      {{ getMonth(activeMonth.days[15].date) }}
    </p>
    <div class="vhd__datepicker__week-row vhd__hide-up-to-tablet">
      <div
        class="vhd__datepicker__week-name"
        v-for="(dayName, ixName) in dayNames"
        :key="`weekname-${dayName}-${ixName}`"
      >
        {{ dayName }}
      </div>
    </div>
    <div
      class="vhd__square"
      v-for="(day, ixDay) in activeMonth.days"
      @mouseenter="mouseEnterDay(day)"
      :key="`${datepickerDayKey}-${activeMonthIndex + month}-${ixDay}`"
    >
      <Day
        :date="day.date"
        :belongsToThisMonth="day.belongsToThisMonth"
        :activeMonthIndex="activeMonthIndex"
        :bookings="bookings"
        :checkIn="checkIn"
        :checkIncheckOutHalfDay="checkIncheckOutHalfDay"
        :checkInPeriod="checkInPeriod"
        :checkOut="checkOut"
        :disableCheckoutOnCheckin="disableCheckoutOnCheckin"
        :duplicateBookingDates="duplicateBookingDates"
        :hoveringDate="hoveringDate"
        :hoveringPeriod="hoveringPeriod"
        :i18n="i18n"
        :isOpen="isOpen"
        :minNightCount="minNightCount"
        :nextDisabledDate="nextDisabledDate"
        :nextPeriodDisableDates="nextPeriodDisableDates"
        :screenSize="screenSize"
        :showCustomTooltip="showCustomTooltip"
        :showPrice="showPrice"
        :sortedDisabledDates="sortedDisabledDates"
        :sortedPeriodDates="sortedPeriodDates"
        :tooltipMessage="tooltipMessage"
        :options="options"
        @clear-selection="clearSelection"
        @booking-clicked="bookingClicked"
        @day-clicked="dayClicked"
      />
    </div>
  </div>
</template>

<script>
import fecha from 'fecha'

import Day from './Day.vue'
import Helpers from '../../helpers'

export default {
  name: 'Month',
  components: {
    Day,
  },
  props: {
    month: {
      type: Number,
      required: true,
    },
    monthIndex: {
      type: Number,
      required: true,
    },
    alwaysVisible: {
      type: Boolean,
      default: false,
    },
    closeDatepickerOnClickOutside: {
      type: Boolean,
      default: true,
    },
    disabledDates: {
      type: Array,
      default() {
        return []
      },
    },
    disabledDaysOfWeek: {
      type: Array,
      default() {
        return []
      },
    },
    disabledWeekDays: {
      type: Object,
      default() {
        return {}
      },
    },
    displayClearButton: {
      type: Boolean,
      default: true,
    },
    enableCheckout: {
      type: Boolean,
      default: false,
    },
    endDate: {
      type: [Date, String, Number],
      default: Infinity,
    },
    endingDateValue: {
      type: [Date, null],
      default: null,
    },
    format: {
      type: String,
      default: 'YYYY-MM-DD',
    },
    gridStyle: {
      type: Boolean,
      default: true,
    },
    halfDay: {
      type: Boolean,
      default: true,
    },
    lastDateAvailable: {
      type: [Number, Date],
      default: Infinity,
    },
    maxNights: {
      type: [Number, null],
      default: null,
    },
    minNights: {
      type: Number,
      default: 1,
    },
    positionRight: {
      type: Boolean,
      default: false,
    },
    showSingleMonth: {
      type: Boolean,
      default: false,
    },
    showYear: {
      type: Boolean,
      default: true,
    },
    singleDaySelection: {
      type: Boolean,
      default: false,
    },
    startDate: {
      type: [Date, String],
      default() {
        return new Date()
      },
    },
    startingDateValue: {
      type: [Date, null],
      default: null,
    },
    bookings: {
      type: Array,
      default: () => [],
    },
    activeMonthIndex: {
      type: Number,
      required: true,
    },
    checkIn: {
      type: [Date, null],
    },
    checkIncheckOutHalfDay: {
      type: Object,
      default: () => ({}),
    },
    checkInPeriod: {
      type: Object,
      default: () => ({}),
    },
    checkOut: {
      type: [Date, null],
    },
    disableCheckoutOnCheckin: {
      type: Boolean,
      default: false,
    },
    duplicateBookingDates: {
      type: Array,
      default: () => [],
    },
    hoveringDate: {
      type: Date,
    },
    hoveringPeriod: {
      type: Object,
      default: () => ({}),
    },
    hoveringTooltip: {
      default: true,
      type: Boolean,
    },
    i18n: {
      type: Object,
      default: () => ({}),
    },
    isOpen: {
      type: Boolean,
    },
    minNightCount: {
      type: Number,
      default: 0,
    },
    nextDisabledDate: {
      type: [Date, Number, String],
    },
    nextPeriodDisableDates: {
      type: Array,
      default: () => [],
    },
    options: {
      type: Object,
    },
    screenSize: {
      type: String,
      default: '',
    },
    customTooltip: {
      default: false,
      type: Boolean,
    },
    showPrice: {
      type: Boolean,
      default: false,
    },
    sortedDisabledDates: {
      type: Array,
      default: () => [],
    },
    sortedPeriodDates: {
      type: Array,
      default: () => [],
    },
    tooltipMessage: {
      default: null,
      type: String,
    },
    firstDayOfWeek: {
      type: Number,
      default: 0,
    },
    months: {
      type: Array,
    },
  },
  data() {
    return {
      datepickerDayKey: 0,
    }
  },
  computed: {
    dayNames() {
      // eslint-disable-next-line prefer-const
      let names = [...this.i18n['day-names']]
      const moveNames = names.splice(0, this.firstDayOfWeek)

      names.splice(names.length, moveNames.length, ...moveNames)

      return names
    },
    activeMonth() {
      return this.months[this.activeMonthIndex + this.month]
    },
    showCustomTooltip: {
      get() {
        return this.customTooltip
      },
      set(value) {
        this.$emit('update:customTooltip', value)
      },
    },
  },
  created() {
    fecha.i18n = {
      dayNames: this.i18n['day-names'],
      dayNamesShort: this.shortenString(this.i18n['day-names'], 3),
      monthNames: this.i18n['month-names'],
      monthNamesShort: this.shortenString(this.i18n['month-names'], 3),
      amPm: ['am', 'pm'],
      // D is the day of the month, function returns something like...  3rd or 11th
      DoFn(D) {
        return D + ['th', 'st', 'nd', 'rd'][D % 10 > 3 ? 0 : ((D - (D % 10) !== 10) * D) % 10]
      },
    }
  },
  methods: {
    ...Helpers,
    setCustomTooltipOnHover(day) {
      this.$emit('set-custom-tooltip-on-hover', day)
    },
    clearSelection() {
      this.$emit('clear-selection')
    },
    bookingClicked(event, date, currentBooking) {
      this.$emit('booking-clicked', event, date, currentBooking)
    },
    dayClicked(event, date, formatDate, resetCheckin) {
      this.$emit('day-clicked', event, date, formatDate, resetCheckin)
    },
    getMonth(date) {
      return this.i18n['month-names'][fecha.format(date, 'M') - 1] + (this.showYear ? fecha.format(date, ' YYYY') : '')
    },
    mouseEnterDay(day) {
      const formatDate = this.dateFormater(day.date)
      const halfDays = Object.keys(this.checkIncheckOutHalfDay)
      const disableDays = this.disabledDates
        .filter((disableDate) => !halfDays.includes(disableDate))
        .includes(formatDate)

      if (!this.dayIsDisabled(day.date) && day.belongsToThisMonth && !disableDays) {
        this.setCustomTooltipOnHover(day)
      }
    },
    dayIsDisabled(date) {
      if (
        this.checkIn &&
        !this.checkOut &&
        !this.isDateLessOrEquals(date, this.nextDisabledDate) &&
        this.nextDisabledDate !== Infinity
      ) {
        return true
      }

      if (this.checkIn && !this.checkOut && this.isDateLessOrEquals(date, this.checkIn)) {
        return true
      }

      return false
    },

    ///////
    setCurrentPeriod(date, eventType) {
      this.$emit('set-current-period', date, eventType)
    },
    setCustomTooltipOnHover(day) {
      const { date } = day

      this.hoveringDate = date
      if (this.showCustomTooltip) this.showCustomTooltip = false

      this.setCurrentPeriod(date, 'hover')

      if (Object.keys(this.hoveringPeriod).length > 0) {
        // Create tooltip
        if (this.hoveringPeriod.periodType === 'weekly_by_saturday') {
          const dayCode = 6
          const text = this.i18n.tooltip.saturdayToSaturday

          this.showTooltipWeeklyOnHover(date, dayCode, text)
        } else if (this.hoveringPeriod.periodType === 'weekly_by_sunday') {
          const dayCode = 0
          const text = this.i18n.tooltip.sundayToSunday

          this.showTooltipWeeklyOnHover(date, dayCode, text)
        } else if (this.hoveringPeriod.periodType === 'nightly') {
          this.showTooltipNightlyOnHover(date)
        } else {
          // Clean tooltip
          this.showCustomTooltip = false
          this.customTooltip = ''
        }
      } else {
        this.hoveringPeriod = {}
      }

      if (this.halfDay) {
        this.createHalfDayTooltip(day.date)
      }
    },
    showTooltipWeeklyOnHover(date, periodDayType, text) {
      const countDaysBetweenCheckInCurrentDay = this.countDays(this.checkIn, date)
      const notOnPeriodDayType = date.getDay() !== periodDayType
      const isCheckInCheckOut = this.checkIn && this.checkOut
      const notCheckInNotPeriodDayType = !this.checkIn && notOnPeriodDayType
      const isCheckInNotCheckOut = this.checkIn && !this.checkOut
      const isNotBetweenCheckInAndCheckOut = !this.validateDateBetweenTwoDates(this.checkIn, this.checkOut, date)
      const notAllowDaysBetweenCheckInAndNextValidDate =
        this.hoveringPeriod.nextValidDate &&
        this.validateDateBetweenTwoDates(this.checkIn, this.hoveringPeriod.nextValidDate, this.hoveringDate) &&
        this.dateFormater(this.checkIn) !== this.dateFormater(this.hoveringDate) &&
        this.dateFormater(this.hoveringPeriod.nextValidDate) !== this.dateFormater(this.hoveringDate)
      const hasHalfDayOnWeeklyPeriod =
        Object.keys(this.checkIncheckOutHalfDay).length > 0 &&
        this.checkIncheckOutHalfDay[this.dateFormater(date)] &&
        this.checkIncheckOutHalfDay[this.dateFormater(date)].checkIn

      // Show tooltip on not-allowed day
      if (notCheckInNotPeriodDayType) {
        this.showCustomTooltip = true
        this.customTooltip = text
      } else {
        this.showCustomTooltip = false
        this.customTooltip = ''
      }

      // Show tooltip when CheckIn
      if (isCheckInNotCheckOut) {
        const nextDayValid = this.addDays(this.checkIn, this.minNightCount)
        const isDateAfterMinimumDuration = this.getDayDiff(date, nextDayValid) <= 0

        if (isDateAfterMinimumDuration && notOnPeriodDayType) {
          this.showCustomTooltip = true
          this.customTooltip = text
        } else if (notOnPeriodDayType || notAllowDaysBetweenCheckInAndNextValidDate) {
          if (this.checkInPeriod && this.checkInPeriod.periodType === 'nightly') {
            this.showCustomTooltip = false
            this.customTooltip = ''
          } else {
            // Show default message on currentDay
            const night = this.pluralize(this.minNightCount, 'week')

            this.showCustomTooltip = true
            this.customTooltip = this.completeTrad(this.i18n.tooltip.minimumRequiredPeriod, {
              minNightInPeriod: this.minNightCount / 7,
              night,
            })
          }
        } else if (hasHalfDayOnWeeklyPeriod) {
          // Show the correct wording in comparison to periodType of this.checkInPeriod equal to "nightly" / "weekly"
          if (this.checkInPeriod.periodType !== 'nightly') {
            this.customTooltip = `${countDaysBetweenCheckInCurrentDay / 7} ${this.pluralize(
              this.minNightCount,
              'week',
            )}`
          } else if (this.checkInPeriod.periodType === 'nightly') {
            this.customTooltip = `${countDaysBetweenCheckInCurrentDay} ${
              countDaysBetweenCheckInCurrentDay !== 1 ? this.i18n.nights : this.i18n.night
            }`
          }
        } else {
          // Clean tooltip
          this.showCustomTooltip = false
          this.customTooltip = ''
        }
        // Show tooltip when CheckIn & CheckOut on all the days that are not between checkIn and CheckOut
      } else if (isCheckInCheckOut && notOnPeriodDayType && isNotBetweenCheckInAndCheckOut) {
        this.showCustomTooltip = true
        this.customTooltip = text
      }
    },
    showTooltipWeeklyOnClick() {
      const night = this.pluralize(this.minNightCount, 'week')

      this.showCustomTooltip = true
      this.customTooltip = this.completeTrad(this.i18n.tooltip.minimumRequiredPeriod, {
        minNightInPeriod: this.minNightCount / 7,
        night,
      })
    },
    showTooltipNightlyOnHover(date) {
      if (this.checkIn && !this.checkOut) {
        const nextDayValid = this.addDays(this.checkIn, this.minNightCount)
        const isDateAfterMinimumDuration = this.getDayDiff(date, nextDayValid) <= 0
        const countOfDays = this.countDays(this.checkIn, date)
        const night = this.pluralize(Math.max(this.minNightCount, countOfDays))

        if (!isDateAfterMinimumDuration) {
          const minNightInPeriod = this.hoveringPeriod.minimumDuration

          this.showCustomTooltip = true
          this.customTooltip = this.completeTrad(this.i18n.tooltip.minimumRequiredPeriod, {
            minNightInPeriod,
            night,
          })
        } else {
          this.customTooltip = `${countOfDays} ${night}`
        }
      } else {
        this.customTooltip = ''
      }
    },
    showTooltipNightlyOnClick() {
      const minNightInPeriod = this.hoveringPeriod.minimumDuration
      const night = this.pluralize(this.minNightCount)

      this.showCustomTooltip = true
      this.customTooltip = this.completeTrad(this.i18n.tooltip.minimumRequiredPeriod, { minNightInPeriod, night })
    },
    createHalfDayTooltip(date) {
      this.customTooltipHalfday = ''
      const formatedHoveringDate = this.dateFormater(date)

      if (this.checkIncheckOutHalfDay[formatedHoveringDate]) {
        this.showCustomTooltip = true

        if (this.checkIncheckOutHalfDay[formatedHoveringDate].checkIn) {
          this.customTooltipHalfday = this.i18n.tooltip.halfDayCheckOut
        } else if (this.checkIncheckOutHalfDay[formatedHoveringDate].checkOut) {
          this.customTooltipHalfday = this.i18n.tooltip.halfDayCheckIn
        }
      }
    },
  },
}
</script>

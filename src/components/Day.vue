<template lang='pug'>
  div
    .datepicker__tooltip(v-if='showTooltip && this.options.hoveringTooltip' v-html='tooltipMessageDisplay')
    .datepicker__month-day(
      @click.prevent.stop='dayClicked(date)'
      @keyup.enter.prevent.stop='dayClicked(date)'
      :class='dayClass'
      :style='isToday ? currentDateStyle : ""'
      :tabindex="tabIndex"
      ref="day"
    )
      .day-number {{ dayNumber }}
      .day-price {{ price }}
</template>

<script>
import fecha from 'fecha';

import Helpers from '../helpers.js';

export default {
  name: 'Day',

  props: {
    isOpen: {
      type: Boolean,
      required: true
    },
    sortedDisabledDates: {
      type: Array
    },
    options: {
      type: Object
    },
    checkIn: {
      type: Date
    },
    checkOut: {
      type: Date
    },
    hoveringDate: {
      type: Date
    },
    mounseOverFunction: {
      type: Function
    },
    belongsToThisMonth: {
      type: Boolean
    },
    activeMonthIndex: {
      type: Number
    },
    date: {
      type: Date
    },
    dayNumber: {
      type: String
    },
    nextDisabledDate: {
      type: [Date, Number, String]
    },
    hoveringTooltip: {
      default: true,
      type: Boolean
    },
    tooltipMessage: {
      default: null,
      type: String
    },
    currentDateStyle: {
      type: [Object, null, String],
      required: true
    },
    price: {
      type: [Number, String],
      default: ''
    }
  },

  data() {
    return {
      isHighlighted: false,
      isDisabled: false,
      currentDate: new Date()
    };
  },

  computed: {
    tabIndex() {
      if (
        !this.isOpen ||
        !this.belongsToThisMonth ||
        this.isDisabled ||
        !this.isClickable()
      ) {
        return -1;
      }
      return 0;
    },
    nightsCount() {
      return this.countDays(this.checkIn, this.hoveringDate);
    },
    tooltipMessageDisplay() {
      return this.tooltipMessage
        ? this.tooltipMessage
        : `${this.nightsCount} ${
          this.nightsCount !== 1
            ? this.options.i18n['nights']
            : this.options.i18n['night']
        }`;
    },
    showTooltip() {
      return (
        !this.isDisabled &&
        this.date == this.hoveringDate &&
        this.checkIn !== null &&
        this.checkOut == null
      );
    },

    isToday() {
      return this.compareDay(this.currentDate, this.date) == 0;
    },

    dayClass() {
      if (this.belongsToThisMonth) {
        // If the calendar has a minimum number of nights
        if (
          !this.isDisabled &&
          this.compareDay(this.date, this.checkIn) == 1 &&
          this.options.minNights > 0 &&
          this.compareDay(
            this.date,
            this.addDays(this.checkIn, this.options.minNights)
          ) == -1
        ) {
          return 'datepicker__month-day--selected datepicker__month-day--out-of-range';
        }

        // Highlight the selected dates and prevent the user from selecting
        // the same date for checkout and checkin
        if (
          this.checkIn !== null &&
          fecha.format(this.checkIn, 'YYYYMMDD') ==
            fecha.format(this.date, 'YYYYMMDD')
        ) {
          if (this.options.minNights == 0) {
            return 'datepicker__month-day--first-day-selected';
          } else {
            return 'datepicker__month-day--disabled datepicker__month-day--first-day-selected';
          }
        }
        if (this.checkOut !== null) {
          if (
            fecha.format(this.checkOut, 'YYYYMMDD') ==
            fecha.format(this.date, 'YYYYMMDD')
          ) {
            return 'datepicker__month-day--disabled datepicker__month-day--last-day-selected';
          }
        }
        // Only highlight dates that are not disabled
        if (this.isHighlighted && !this.isDisabled) {
          return ' datepicker__month-day--selected';
        }
        if (this.isDisabled) {
          return 'datepicker__month-day--disabled';
        }
      } else if (!this.belongsToThisMonth) {
        return 'datepicker__month-day--hidden';
      }
      return 'datepicker__month-day--valid';
    }
  },

  watch: {
    hoveringDate(date) {
      if (
        this.checkIn !== null &&
        this.checkOut == null &&
        this.isDisabled == false
      ) {
        this.isDateLessOrEquals(this.checkIn, this.date) &&
        this.isDateLessOrEquals(this.date, this.hoveringDate)
          ? (this.isHighlighted = true)
          : (this.isHighlighted = false);
      }
    },
    activeMonthIndex(index) {
      this.checkIfDisabled();
      this.checkIfHighlighted();
      if (this.checkIn !== null && this.checkOut !== null) {
        this.isDateLessOrEquals(this.checkIn, this.date) &&
        this.isDateLessOrEquals(this.date, this.checkOut)
          ? (this.isHighlighted = true)
          : (this.isHighlighted = false);
      } else if (this.checkIn !== null && this.checkOut == null) {
        this.disableNextDays();
      } else {
        return;
      }
    },
    nextDisabledDate() {
      this.disableNextDays();
    },
    checkIn(date) {
      this.createAllowedCheckoutDays(date);
    }
  },
  beforeMount() {
    this.checkIfDisabled();
    this.checkIfHighlighted();
  },

  methods: {
    ...Helpers,

    isClickable() {
      if (this.$refs && this.$refs.day) {
        return getComputedStyle(this.$refs.day).pointerEvents !== 'none';
      } else {
        return true;
      }
    },

    compareDay(day1, day2) {
      const date1 = fecha.format(new Date(day1), 'YYYYMMDD');
      const date2 = fecha.format(new Date(day2), 'YYYYMMDD');

      if (date1 > date2) {
        return 1;
      } else if (date1 == date2) {
        return 0;
      } else if (date1 < date2) {
        return -1;
      }
    },

    dayClicked(date) {
      if (this.isDisabled || !this.isClickable()) {
        return;
      } else {
        let nextDisabledDate =
          (this.options.maxNights
            ? this.addDays(this.date, this.options.maxNights)
            : null) ||
          this.getNextDate(this.sortedDisabledDates, this.date)||
          Infinity;

        this.$emit('day-clicked', { date, nextDisabledDate });
      }
    },

    compareEndDay() {
      if (this.options.endDate !== Infinity) {
        return this.compareDay(this.date, this.options.endDate) == 1;
      }
    },

    checkIfDisabled() {
      this.isDisabled =
        // If this day is equal any of the disabled dates
        (this.sortedDisabledDates
          ? this.sortedDisabledDates.some(
            i => this.compareDay(i, this.date) == 0
          )
          : null) ||
        // Or is before the start date
        this.compareDay(this.date, this.options.startDate) == -1 ||
        // Or is after the end date
        this.compareEndDay();
    },

    checkIfHighlighted() {
      if (
        this.checkIn !== null &&
        this.checkOut !== null &&
        this.isDisabled == false
      ) {
        this.isDateLessOrEquals(this.checkIn, this.date) &&
        this.isDateLessOrEquals(this.date, this.checkOut)
          ? (this.isHighlighted = true)
          : (this.isHighlighted = false);
      }
    },

    disableNextDays() {
      if (
        !this.isDateLessOrEquals(this.date, this.nextDisabledDate) &&
        this.nextDisabledDate !== Infinity
      ) {
        this.isDisabled = true;
      } else if (this.isDateLessOrEquals(this.date, this.checkIn)) {
        this.isDisabled = true;
      }
      if (
        this.compareDay(this.date, this.checkIn) == 0 &&
        this.options.minNights == 0
      ) {
        this.isDisabled = false;
      }
    }
  },

};
</script>

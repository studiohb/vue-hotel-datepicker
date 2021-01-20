<template lang="pug">
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
      .day-price(v-if='showPrice') {{ price }}
</template>

<script>
import Helpers from '../helpers.js';

export default {
  name: 'Day',

  props: {
    isOpen: {
      type: Boolean,
      required: true
    },
    disabledDatesForCheckIn: {
      type: Array,
      default: () => []
    },
    disabledDatesForCheckOut: {
      type: Array,
      default: () => []
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
    choosingCheckOut: {
      type: Boolean
    },
    hoveringDate: {
      type: Date
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
      default: null
    }
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
      return this.compareDay(new Date(), this.date) == 0;
    },

    isCheckInDay() {
      return this.compareDay(this.checkIn, this.date) == 0;
    },

    isCheckOutDay() {
      return this.compareDay(this.checkOut, this.date) == 0;
    },

    isDisabled() {
      return this.disabledDates.some(i => this.compareDay(i, this.date) == 0) // If this day is explicitely disabled
          || this.compareDay(this.date, this.options.startDate) == -1 // Or is before the calendar start date
          || this.compareDay(this.date, this.options.endDate) == 1 // Or is after the calendar end date
          || (this.choosingCheckOut && this.compareDay(this.date, this.checkIn) <= 0) // Or is before (or equals) the check-in date (only when users are choosing their check-out date)
          || this.compareDay(this.date, this.nextDisabledDate) == 1 // Or is after the next disabled date
          || !this.price; // Or it doesn't have a price
    },

    isEnabled() {
      return !this.isDisabled;
    },

    isHighlighted() {
      if (!this.checkIn) return false;
      return this.isDayInRange(this.date, [this.checkIn, this.checkOut || this.hoveringDate]);
    },

    forbidsCheckIn() {
      return this.disabledDatesForCheckIn.some(i => this.compareDay(i, this.date) == 0);
    },

    forbidsCheckOut() {
      return this.disabledDatesForCheckOut.some(i => this.compareDay(i, this.date) == 0);
    },

    dayClass() {
      if (!this.belongsToThisMonth) return this.cssClass('hidden');

      const classes = [this.cssClass(this.isDisabled ? 'disabled' : 'valid')];

      // Selection
      if (this.isHighlighted) classes.push(this.cssClass('selected'));
      if (this.isCheckInDay) classes.push(this.cssClass('first-day-selected'));
      if (this.isCheckOutDay) classes.push(this.cssClass('last-day-selected'));

      // Half-days
      if (this.forbidsCheckIn) classes.push(this.cssClass('check-in-forbidden'));
      if (this.forbidsCheckOut) classes.push(this.cssClass('check-out-forbidden'));

      return classes.join(' ');
    },

    disabledDates() {
      return this.sortDates(
        this.choosingCheckOut ? this.disabledDatesForCheckOut : this.disabledDatesForCheckIn
      );
    },

    showPrice() {
      return this.isEnabled || this.isCheckInDay;
    },
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

    dayClicked(date) {
      if (!this.isDisabled && this.isClickable())
        this.$emit('day-clicked', date);
    },

    cssClass(suffix) {
      return `datepicker__month-day--${suffix}`;
    },
  },
};
</script>

<template lang="pug">
  .datepicker__input(
    @click="toggleDatepicker"
    @keyup.enter.stop.prevent="toggleDatepicker"
    data-qa='datepickerInput'
    v-text="inputDate ? inputDate : i18n[inputDateType]"
    :tabindex="tabIndex"
  )
</template>

<script>

export default {
  props: {
    isOpen: {
      type: Boolean,
      required: true,
    },
    inputDate: {
      type: String,
      default: null,
    },
    inputDateType: {
      type: String,
      default: 'check-in'
    },
    toggleDatepicker: {
      type: Function,
      required: true,
    },
    i18n: {
      type: Object,
      required: true,
    },
  },

  computed: {
    tabIndex() {
      return this.inputDateType === 'check-in' ? 0 : -1;
    }
  },
};
</script>

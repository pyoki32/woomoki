<template>
  <v-col class="pa-0">
    <v-select
      v-model="value"
      color="#AED864"
      :items="day"
      label="한 주에 며칠 보살펴줄까요?"
      suffix="일"
      outlined
    ></v-select>
  </v-col>
</template>

<script>
export default {
  name: "SeedCertificationDay",
  data: function () {
    return {
      value: 0,
      day: [1, 2, 3, 4, 5, 6, 7],
      dayRules: [
        v => !!v || '며칠은 필수 항목입니다',
        v => this.valiteateCount(v) || '인증 횟수는 숫자만 입력해야합니다',
        v => v.length <= 2 || '인증 횟수는 2자 이하로 적어주세요',
      ],
    }
  },
  methods: {
    valiteateCount: function(count) {
      const re = /^[0-9]*$/
      return re.test(count)
    },
  },
  watch: {
    value: function () {
      this.$emit('transferDay', this.value)
    }
  }
}
</script>

<style scoped>

</style>
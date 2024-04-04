<template>
  <div slot="extra" class="container">
    <div class="label">{{ fullLabel }}</div>
    <a-progress
      :percent="value"
      strokeLinecap="square"
      :strokeWidth="5"
      status="normal"
      :showInfo="false"
      class="progress-bar"
    />
  </div>
</template>

<script>
export default {
  props: {
    label: String,
    value: Number
  },
  computed: {
    fullLabel () {
      return `${this.label}: (${this.value}%)`
    }
  }
}
</script>

<style scoped>
.container {
  font-size: 8px;
  width: 90px;
}
.label {
  font-size: 10px;
  color: rgb(40, 38, 177);
  text-align: center;
  padding-bottom: 0;
  margin-bottom: 0;
}
.progress-bar {
  font-size: 3px;
  color: rgb(40, 38, 177);
  box-shadow: 1px 1px 2px 1px rgba(0, 0, 0, 0.1) !important;
}
</style>

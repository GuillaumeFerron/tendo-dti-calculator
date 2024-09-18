<template>
  <div class="card mb-0 w-75">
    <div class="card-body px-5">
      <div class="d-flex justify-content-between align-items-center">
        <div>
          <h3>DtI Calculator</h3>
        </div>
        <div v-if="tooltipContent.length > 0" v-tooltip="{
          content: tooltipContent,
          html: true,
          hideTriggers: ['click'],
          autoHide: true
        }" class="clickable">
          <h3>ⓘ</h3>
        </div>
      </div>
      <DtiCalculator />
    </div>
  </div>
</template>

<script>
import DtiCalculator from '../DtiCalculator.vue'

export default {
  components: {
    DtiCalculator
  },
  data() {
    return {
      tooltip: {
        old: 'https://docs.google.com/spreadsheets/d/14xuHU0B83wtoJ5pNWM4KhXqXmZsPOCtzmnG2qSPUHFU/edit?gid=1334544319#gid=1334544319',
        guide: 'https://scribehow.com/shared/Copying_Data_to_Tendo_DTI_Calculator__707UvZKdR8KpomOZi4thew'
      }
    }
  },
  computed: {
    tooltipContent() {
      return (this.tooltip.guide ? `Guide: <a href="${this.tooltip.guide}" target="_blank">Here</a><br>` : '') + (this.tooltip.old ? `Legacy: <a href="${this.tooltip.old}" target="_blank">Here</a>` : '')
    }
  }
}
</script>

<style lang="scss" scoped>
.card {
  -webkit-box-shadow: 5px 5px 15px 5px rgba(0, 0, 0, 0.3);
  box-shadow: 5px 5px 15px 5px rgba(0, 0, 0, 0.3);
}
</style>

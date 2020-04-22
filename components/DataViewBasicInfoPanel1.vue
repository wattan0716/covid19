<template>
  <div class="DataView-DataInfo">
    <span class="DataView-DataInfo-summary">
      {{ lText }}
      <small class="DataView-DataInfo-summary-unit">{{ unit }}</small>
    </span>
    <br />
    <small class="DataView-DataInfo-date">{{ sText1 }}</small>
    <small class="DataView-DataInfo-date">{{ sText2 }}</small>
  </div>
</template>

<style lang="scss">
.DataView {
  &-DataInfo {
    text-align: right;
    &-summary {
      display: inline-block;
      font-family: Hiragino Sans;
      font-style: normal;
      font-size: 30px;
      line-height: 30px;
      &-unit {
        font-size: 0.6em;
      }
    }
    &-date {
      white-space: nowrap;
      display: inline-block;
      font-size: 12px;
      line-height: 12px;
      color: $gray-3;
    }
  }
}
.DataView {
  @include card-container();
  height: 100%;
  &-Header {
    background-color: transparent !important;
    height: auto !important;
  }
}
</style>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'

@Component
export default class DataViewBasicInfoPanel1 extends Vue {
  @Prop() private lText!: string
  @Prop() private sText1!: string
  @Prop() private sText2!: string
  @Prop() private unit!: string
}
</script>

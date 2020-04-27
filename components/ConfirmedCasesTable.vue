<template>
  <ul
    :class="$style.container"
    :aria-label="
      ariaLabel(
        検査実施人数,
        陽性物数,
        入院中,
        軽症中等症,
        重症,
        死亡,
        退院,
        現在陽性者数,
        入院調整中,
        自宅療養,
        宿泊療養,
        療養等調整中
      )
    "
  >
    <li>
      <div :class="[$style.row, $style['is-black']]">
        <span v-text="$t('検査実施件数')">)"></span>
        <span :class="$style.value"
          >{{ 検査実施人数 }}{{ $t('件.tested') }}</span
        >
      </div>
    </li>
    <li>
      <div :class="$style.row">
        <span>{{ $t('陽性者数（累積）') }}</span>
        <span :class="$style.value">{{ 陽性物数 }}{{ $t('人') }}</span>
      </div>
      <ul :class="$style.container">
        <li>
          <div :class="[$style.row, $style['is-positive']]">
            <span>{{ $t('現在陽性者数') }}</span>
            <span :class="$style.value">{{ 現在陽性者数 }}{{ $t('人') }}</span>
          </div>
          <ul :class="$style.container">
            <li>
              <div :class="[$style.row, $style['is-gray']]">
                <span v-text="$t('入院')" />
                <span :class="$style.value">{{ 入院中 }}{{ $t('人') }}</span>
              </div>
              <ul :class="$style.container">
                <li>
                  <div :class="[$style.row, $style['is-gray']]">
                    <span>{{ $t('重症') }}</span>
                    <span :class="$style.value">{{ 重症 }}{{ $t('人') }}</span>
                  </div>
                </li>
              </ul>
            </li>
            <li>
              <div :class="[$style.row, $style['is-gray']]">
                <span v-text="$t('入院調整中')" />
                <span :class="$style.value"
                  >{{ 入院調整中 }}{{ $t('人') }}</span
                >
              </div>
            </li>
            <li>
              <div :class="[$style.row, $style['is-gray']]">
                <span>{{ $t('自宅療養') }}</span>
                <span :class="$style.value">{{ 自宅療養 }}{{ $t('人') }}</span>
              </div>
            </li>
            <li>
              <div :class="[$style.row, $style['is-gray']]">
                <span>{{ $t('宿泊療養') }}</span>
                <span :class="$style.value">{{ 宿泊療養 }}{{ $t('人') }}</span>
              </div>
            </li>
            <li>
              <div :class="[$style.row, $style['is-gray']]">
                <span>{{ $t('療養等調整中') }}</span>
                <span :class="$style.value"
                  >{{ 療養等調整中 }}{{ $t('人') }}</span
                >
              </div>
            </li>
          </ul>
        </li>
        <li>
          <div :class="[$style.row, $style['is-deceased']]">
            <span>{{ $t('死亡') }}</span>
            <span :class="$style.value">{{ 死亡 }}{{ $t('人') }}</span>
          </div>
        </li>
        <li>
          <div :class="$style.row">
            <span>{{ $t('退院・解除済累計') }}</span>
            <span :class="$style.value">{{ 退院 }}{{ $t('人') }}</span>
          </div>
        </li>
      </ul>
    </li>
  </ul>
</template>

<script>
/* eslint-disable vue/prop-name-casing */
export default {
  props: {
    検査実施人数: {
      type: Number,
      required: true
    },
    陽性物数: {
      type: Number,
      required: true
    },
    入院中: {
      type: Number,
      required: true
    },
    軽症中等症: {
      type: Number,
      required: true
    },
    重症: {
      type: Number,
      required: true
    },
    死亡: {
      type: Number,
      required: true
    },
    退院: {
      type: Number,
      required: true
    },
    現在陽性者数: {
      type: Number,
      required: true
    },
    入院調整中: {
      type: Number,
      required: true
    },
    自宅療養: {
      type: Number,
      required: true
    },
    宿泊療養: {
      type: Number,
      required: true
    },
    療養等調整中: {
      type: Number,
      required: true
    }
  },
  methods: {
    /** グラフ内容がわかる支援技術用テキストの中身を取得する **/
    ariaLabel(
      inspected,
      positive,
      hospitalized,
      mild,
      critically,
      deceased,
      discharged,
      currentlyPositive,
      hospitalizeAdjust,
      recuperatingHome,
      recuperatingLodging,
      recuperatingAdjust
    ) {
      return this.$t(
        '検査陽性者の状況: 検査実施人数は{inspected}人、うち累積の陽性者数は{positive}人で、現在陽性者数は{currentlyPositive}人です。入院中は{hospitalized}人で、重症は{critically}人です、また入院調整中は{hospitalizeAdjust}人です。自宅療養は{recuperatingHome}人、宿泊療養は{recuperatingLodging}人、また療養等調整中は{recuperatingAdjust}人です。さらに死亡は{deceased}人、退院・解除済累計は{discharged}人です。',
        {
          inspected,
          positive,
          hospitalized,
          mild,
          critically,
          deceased,
          discharged,
          currentlyPositive,
          hospitalizeAdjust,
          recuperatingHome,
          recuperatingLodging,
          recuperatingAdjust
        }
      )
    }
  }
}
</script>

<style lang="scss" module>
$rowSidePadding: 1.5em;
$rowNestPadding: 2em;

.container {
  &,
  ul {
    padding-left: 0 !important;
  }

  &,
  li {
    list-style: none;
  }

  > * + *,
  .container {
    margin-top: 4px;
  }

  // ネスト用のスタイルを吐き出す
  @for $i from 1 to 4 {
    $selector: '.container';

    @for $j from 1 to $i {
      $selector: $selector + ' .container';
    }

    $selector: $selector + ' .row';

    #{$selector} {
      padding-left: $rowNestPadding * $i + $rowSidePadding;
    }
  }
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;

  padding: 0.25em $rowSidePadding;

  font-weight: bold;

  border: solid 2px $green-1;

  color: $green-1;

  &.is-black {
    border-color: #333;
    color: #4d4d4d;
  }

  &.is-deceased {
    background: rgba(#333, 30%);
  }

  &.is-positive {
    background: rgba($green-1, 20%);
  }

  &.is-gray {
    background: rgba(#333, 10%);
  }
}

.value {
  flex: none;

  margin-left: 1em;
}
</style>

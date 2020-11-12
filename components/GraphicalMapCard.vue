<template>
  <!-- <v-col cols="12" md="6" class="DataCard"> -->
  <data-view
    :title="$t('市町村毎の感染状況(地図)')"
    :title-id="'osaka-city-map-table'"
    :date="Data.patients.date"
  >
    <template v-slot:button>
      <p :class="$style.note">
        {{ $t('（注）退院している人数を含む') }}
      </p>
      <p :class="$style.note2">{{ $t('凡例（単位は人）') }}</p>
      <table :class="$style.note2">
        <tbody>
          <tr>
            <td><span class="color-test infected-level1" />1-5</td>
            <td><span class="color-test infected-level2" />6-10</td>
            <td><span class="color-test infected-level3" />11-15</td>
          </tr>
          <tr>
            <td><span class="color-test infected-level4" />16-20</td>
            <td><span class="color-test infected-level5" />21-30</td>
            <td>
              <span class="color-test infected-level6" />31 {{ $t('以上') }}
            </td>
          </tr>
        </tbody>
      </table>
    </template>
    <!-- <ibaraki-map /> -->
    <div id="map" ref="map" class="osaka-map" />
    <date-select-slider
      :chart-data="chartData"
      :value="[0, sliderMax]"
      :slider-max="sliderMax"
      @sliderInput="sliderUpdate"
    />
  </data-view>
  <!-- </v-col> -->
</template>

<script>
import * as d3 from 'd3'
import dayjs from 'dayjs'
import Data from '@/data/data.json'
import DataView from '@/components/DataView.vue'
import DateSelectSlider from '@/components/DateSelectSlider.vue'

const popData = []
// 市町村の患者人数の連想配列
let cityPatientsNumber = {}
let map
let mapDrawn = false // map描画済みフラグ

export default {
  components: {
    DataView,
    DateSelectSlider
  },
  data() {
    // データの一番古い日付
    const dateMin = dayjs(Data.patients.data[0].date)
    // データの一番新しい日付
    const dateMax = dayjs(
      Data.patients.data[Object.keys(Data.patients.data).length - 1].date
    )
    // 日付のダミーデータ
    const chartData = []
    for (let i = 0; dateMin.add(i, 'day') <= dateMax; i++) {
      chartData[i] = { label: dateMin.add(i, 'day') }
    }
    // 日付範囲選択スライダーで選択している範囲
    const graphRange = [0, (dateMax - dateMin) / 86400000 - 1]

    const data = {
      Data,
      dateMin,
      dateMax,
      chartData,
      graphRange
    }
    return data
  },
  computed: {
    // スライダーのデータ数
    sliderMax() {
      return (this.dateMax - this.dateMin) / 86400000
    }
  },
  mounted() {
    this.loadYouseiData()
    drawOsaka(this, this.$refs.map.clientWidth)
  },
  methods: {
    // 日付範囲選択スライダー変更イベント
    sliderUpdate(sliderValue) {
      console.log(sliderValue)
      this.graphRange = sliderValue
      this.loadYouseiData()
      drawOsaka(this, this.$refs.map.clientWidth)
    },
    // 陽性者数集計
    loadYouseiData() {
      console.log('start loadYouseiData()')
      // 初期化
      popData.length = 0
      cityPatientsNumber = {}

      const patients = Data.patients.data

      const minPos = this.graphRange[0]
      const maxPos = this.graphRange[1]
      const minDate = dayjs(this.chartData[minPos].label)
      const maxDate = dayjs(this.chartData[maxPos].label)

      for (const key of patients) {
        const date = dayjs(key.date)
        if (!cityPatientsNumber[key.居住地]) cityPatientsNumber[key.居住地] = 0 // 初期化
        // 日付の範囲ならカウント
        if (minDate <= date && date <= maxDate) {
          cityPatientsNumber[key.居住地] += 1
        }
      }

      for (const key in cityPatientsNumber) {
        const popDataUnit = {}
        popDataUnit.name = key
        popDataUnit.count = cityPatientsNumber[key]

        // 陽性者数に応じて塗る色を計算
        if (popDataUnit.count > 99) {
          popDataUnit.color = 'red'
        }
        if (popDataUnit.count <= 99 && popDataUnit.count > 9) {
          popDataUnit.color = 'deeppink'
        }
        if (popDataUnit.count <= 9 && popDataUnit.count > 4) {
          popDataUnit.color = 'magenta'
        }
        if (popDataUnit.count <= 4 && popDataUnit.count > 1) {
          popDataUnit.color = 'pink'
        }
        // eslint-disable-next-line eqeqeq
        if (popDataUnit.count == 1) {
          popDataUnit.color = 'lemonchiffon'
        }
        // eslint-disable-next-line eqeqeq
        if (popDataUnit.count == 0) {
          popDataUnit.color = 'white'
        }
        popData.push(popDataUnit)
      }

      console.log('end loadYouseiData()')
    }
  }
}

// 大阪府描画
function drawOsaka(vm, elementWidth) {
  console.log('start drawOsaka()')

  // scale = 10000 のときのwidthとheight(描画後のwidth/heightから取得)
  const osakaPrefBaseSize = {
    width: 114.37,
    height: 165.2,
    scale: 10000
  }

  const osakaPrefHorizontalToVerticalRatio =
    osakaPrefBaseSize.height / osakaPrefBaseSize.width

  const osakaPrefHorizontalToScaleRatio =
    osakaPrefBaseSize.scale / osakaPrefBaseSize.width

  const osakaPrefSize = {
    width: elementWidth,
    height: osakaPrefHorizontalToVerticalRatio * elementWidth,
    scale: osakaPrefHorizontalToScaleRatio * elementWidth
  }
  // const ua = window.navigator.userAgent.toLowerCase() // ブラウザ判定 // 未使用のため、コメントアウト
  // scaleはスクリーンの大きさによって変更
  // let scale
  // let label_font_size // 未使用のため、コメントアウト
  // let label_width // 未使用のため、コメントアウト
  // let label_height // 未使用のため、コメントアウト
  // let font_size // 未使用のため、コメントアウト

  // スマートフォンの時は変数調整 // 未使用のためコメントアウト
  /*
  if (width < 601) {
    scale = 30000
    // label_font_size = '16pt' // 未使用のため、コメントアウト
    // label_width = 40 // 未使用のため、コメントアウト
    // font_size = '7pt' // 未使用のため、コメントアウト
    // graphY = height / 2 // 未使用のため、コメントアウト
  } else {
    scale = 25000
    // label_font_size = '16pt' // 未使用のため、コメントアウト
    // label_width = 80 // 未使用のため、コメントアウト
    // font_size = '10pt' // 未使用のため、コメントアウト
  }
  */

  // マップ描画領域作成（初回のみ）
  if (!mapDrawn) {
    map = d3
      .select('#map')
      .append('svg')
      .attr('width', osakaPrefSize.width)
      .attr('height', osakaPrefSize.height)
      .append('g')
  }

  // staticフォルダのgeoJSONファイルをhttp経由で読み込む
  d3.json('osakapref.json').then(function(json) {
    // 市区町村表示領域を生成
    // ツールチップ
    const tooltip = d3
      .select('body')
      .append('div')
      .attr('class', 'tooltip')

    // データの中心点を計算
    // refs: https://qiita.com/yuiken/items/1153922ad20be1d26ced
    const bounds = d3.geoBounds(json)
    const center = [
      (bounds[0][0] + bounds[1][0]) / 2,
      (bounds[0][1] + bounds[1][1]) / 2
    ]

    // 投影を処理する関数を用意する。データからSVGのPATHに変換するため。
    const projection = d3
      .geoMercator()
      .center(center)
      .translate([osakaPrefSize.width / 2, osakaPrefSize.height / 2])
      .scale(osakaPrefSize.scale)
    // pathジェネレータ関数
    const path = d3.geoPath().projection(projection)
    // これがenterしたデータ毎に呼び出されpath要素のd属性にgeoJSONデータから変換した値を入れて市町村境界描画

    if (!mapDrawn) {
      // 初回は地図表示
      map
        .selectAll('path')
        .data(json.features)
        .enter()
        .append('path')
        .attr('d', path)
        // 陽性者に対応した色で境界内を塗る
        .style('fill', function(d) {
          return popData[d.properties.index].color
        })
        // マウスオーバーでツールチップ表示
        .on('mouseover, mousemove', function(d) {
          tooltip
            .style('opacity', 0.9)
            .html(
              '<strong>' +
                vm.$t(popData[d.properties.index].name) +
                '</strong><br>' +
                popData[d.properties.index].count +
                ' ' +
                vm.$t('人')
            )
            .style('left', d3.event.pageX + 'px')
            .style('top', d3.event.pageY - 45 + 'px')
        })
        .on('mouseout', function() {
          tooltip.style('opacity', 0)
        })
      // 地図表示済みに設定
      mapDrawn = true
    } else {
      // ２回目以降は更新のみ
      map
        .selectAll('path')
        // 陽性者に対応した色で境界内を塗る
        .style('fill', function(d) {
          return popData[d.properties.index].color
        })
    }
  })
  console.log('end drawOsaka()')
}
</script>

<style lang="scss" module>
.note {
  @include font-size(12);

  margin-top: 10px;
  margin-bottom: 0;
  color: $gray-3;

  &2 {
    @include font-size(14);
  }
}
</style>
<!-- 本来ならばSVGをinline展開してそこに限定してcssを適用するべきだが、inline展開ができなかったため妥協 -->
<style lang="scss">
$infected-level1: white;
$infected-level2: lemonchiffon;
$infected-level3: pink;
$infected-level4: magenta;
$infected-level5: deeppink;
$infected-level6: red;

.color-test {
  vertical-align: middle;
  width: 2.5rem;
  height: 1rem;
  display: inline-block;
  margin: 0 0.5rem 0 0;
}
// 1-5
.infected-level1 {
  fill: $infected-level1 !important;
  background-color: $infected-level1;
}
// 6-10
.infected-level2 {
  fill: $infected-level2 !important;
  background-color: $infected-level2;
}
// 10-15
.infected-level3 {
  fill: $infected-level3 !important;
  background-color: $infected-level3;
}
// 15-20
.infected-level4 {
  fill: $infected-level4 !important;
  background-color: $infected-level4;
}
// 21-30
.infected-level5 {
  fill: $infected-level5 !important;
  background-color: $infected-level5;
}
// 31-
.infected-level6 {
  fill: $infected-level6 !important;
  background-color: $infected-level6;
}
.osaka-map {
  font-size: 0;
}

.tooltip {
  background: rgba(0, 0, 0, 0.9);
  font-size: 11px;
  color: #fff;
  border-radius: 4px;
  padding: 4px;
  transition: opacity 0.2s;
  position: absolute;
  opacity: 0;
}

//path:hover {
//  opacity: 0.5;
//}
</style>

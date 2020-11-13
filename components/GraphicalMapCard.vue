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
    <div id="map" ref="map" class="osaka-map" />
    <p :class="$style.note">
      <a
        href="https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03-v2_4.html"
        target="_blank"
        rel="noopener noreferrer"
      >
        {{ $t('「国土数値情報（行政区域）」（国土交通省）を加工して作成') }}
      </a>
    </p>
  </data-view>
  <!-- </v-col> -->
</template>

<script>
import * as d3 from 'd3'
import Data from '@/data/data.json'
import DataView from '@/components/DataView.vue'

const popData = []
// 市町村の患者人数の連想配列
const cityPatientsNumber = {}

export default {
  components: {
    // IbarakiMap,
    DataView
  },
  data() {
    const data = {
      Data
    }
    return data
  },
  mounted() {
    loadYouseiData()
    drawOsaka(this)
  }
}

function loadYouseiData() {
  console.log('start loadYouseiData()')

  const patients = Data.patients.data

  for (const key of patients) {
    cityPatientsNumber[key.居住地] = cityPatientsNumber[key.居住地] || 0
    ++cityPatientsNumber[key.居住地]
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

// 大阪府描画
function drawOsaka(vm) {
  console.log('start drawOsaka()')

  // マップ描画
  const map = d3
    .select('#map')
    .append('svg')
    .append('g')

  // staticフォルダのgeoJSONファイルをhttp経由で読み込む
  Promise.all([
    d3.json('osakapref.json'),
    d3.json('osakapref-centroid.json')
  ]).then(files => {
    const json = files[0]
    const centroid = files[1]

    // 市区町村表示領域を生成
    // ツールチップ
    const tooltip = d3
      .select('body')
      .append('div')
      .attr('class', 'tooltip')

    // 投影を処理する関数を用意する。データからSVGのPATHに変換するため。
    let projection = d3.geoMercator().fitWidth(486, json)

    // pathジェネレータ関数
    const path = d3.geoPath().projection(projection)
    // これがenterしたデータ毎に呼び出されpath要素のd属性にgeoJSONデータから変換した値を入れて市町村境界描画

    // paddingを追加し、viewBox属性のための数値を取得
    let bounds = path.bounds(json)
    projection = projection.fitExtent(
      [
        [5, 5],
        [bounds[1][0], bounds[1][1]]
      ],
      json
    )
    bounds = path.bounds(json)

    d3.select('#map')
      .select('svg')
      .attr(
        'viewBox',
        `0, 0, ${bounds[0][0] + bounds[1][0]}, ${bounds[0][1] + bounds[1][1]}`
      )

    map
      .selectAll('path')
      .data(json.features)
      .enter()
      .append('path')
      .attr('class', 'land')
      .attr('d', path)
      // 陽性者に対応した色で境界内を塗る
      .style('fill', d => {
        return popData[d.properties.index].color
      })
      .on('mouseover, mousemove', d => {
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
      .on('mouseout', () => {
        tooltip.style('opacity', 0)
      })

    // 市区町村名の表示
    const g = map
      .selectAll('g')
      .data(
        centroid.features.map(v => [
          projection(v.geometry.coordinates),
          v.properties.name
        ])
      )
      .enter()
      .append('g')

    g.append('text')
      .attr('class', 'text text-back')
      .attr('x', d => d[0][0])
      .attr('y', d => d[0][1])
      .text(d => d[1])

    g.append('text')
      .attr('class', 'text text-front')
      .attr('x', d => d[0][0])
      .attr('y', d => d[0][1])
      .text(d => d[1])

    console.log('end drawOsaka()')
  })
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

#map svg {
  width: 100%;
}

#map .land {
  stroke: #333;
  stroke-width: 1px;
}

#map .text {
  font-size: 12px;
  font-weight: bold;

  text-anchor: middle;
  dominant-baseline: middle;
}

#map .text-back {
  stroke: #fff;
  stroke-width: 4px;
  stroke-linejoin: round;
}

#map .text-front {
  fill: #000;
}
</style>

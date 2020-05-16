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
            <td><span class="color-test infected-level6" />31以上</td>
          </tr>
        </tbody>
      </table>
    </template>
    <!-- <ibaraki-map /> -->
    <div id="map" />
  </data-view>
  <!-- </v-col> -->
</template>

<script>
import * as d3 from 'd3'
import Data from '@/data/data.json'
import DataView from '@/components/DataView.vue'
// import IbarakiMap from '@/assets/ibaraki-map.svg'
// import CityData from '@/data/cities.json'

let graphY = 400
const pop_data = []
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
    drawOsaka()
    // const patients = Data.patients.data
    // 市町村の患者人数の連想配列
    // const cityPatientsNumber = {}
    // for (const key of patients) {
    //   cityPatientsNumber[key.居住地] = patients.filter(function(x) {
    //     return x.居住地 === key.居住地
    //   }).length
    // }

    // CityData.forEach(element => {
    //   if (!cityPatientsNumber[element.city]) {
    //     return
    //   }
    //   const targetElement = document.getElementById(
    //     'ibaraki-map_svg__' + element.Romaji
    //   )
    //   if (cityPatientsNumber[element.city] <= 5)
    //     targetElement.classList.add('infected-level1')
    //   else if (cityPatientsNumber[element.city] <= 10)
    //     targetElement.classList.add('infected-level2')
    //   else if (cityPatientsNumber[element.city] <= 15)
    //     targetElement.classList.add('infected-level3')
    //   else if (cityPatientsNumber[element.city] <= 20)
    //     targetElement.classList.add('infected-level4')
    //   else if (cityPatientsNumber[element.city] <= 30)
    //     targetElement.classList.add('infected-level5')
    //   else targetElement.classList.add('infected-level6')
    // })
  }
}

function loadYouseiData() {
  console.log('start loadYouseiData()')

  const xhr = new XMLHttpRequest()
  xhr.onload = function() {
    const tempArray = xhr.responseText.split('\n')
    for (let i = 0; i < tempArray.length; i++) {
      const strText = tempArray[i].split(',')
      // 陽性者数に応じて塗る色を計算
      if (strText[1] > 99) {
        strText[2] = 'red'
      }
      if (strText[1] <= 99 && strText[1] > 9) {
        strText[2] = 'deeppink'
      }
      if (strText[1] <= 9 && strText[1] > 4) {
        strText[2] = 'magenta'
      }
      if (strText[1] <= 4 && strText[1] > 1) {
        strText[2] = 'pink'
      }
      if (strText[1] == 1) {
        strText[2] = 'lemonchiffon'
      }
      if (strText[1] == 0) {
        strText[2] = 'white'
      }
      pop_data.push(strText)
    }
  }
  xhr.open('get', 'yousei.csv', true)
  xhr.send(null)
  console.log('end loadYouseiData()')
}

// 大阪府描画
function drawOsaka() {
  console.log('start drawOsaka()')

  let g
  const width = window.innerWidth
  const height = window.innerHeight
  const ua = window.navigator.userAgent.toLowerCase() // ブラウザ判定
  // scaleはスクリーンの大きさによって変更
  let scale
  let label_font_size
  let label_width
  let label_height
  let font_size

  // スマートフォンの時は変数調整
  if (width < 601) {
    scale = 30000
    label_font_size = '16pt'
    label_width = 40
    font_size = '7pt'
    graphY = height / 2
  } else {
    scale = 40000
    label_font_size = '16pt'
    label_width = 80
    font_size = '10pt'
  }

  // マップ描画
  const map = d3
    .select('#map')
    .append('svg')
    .attr('width', width)
    .attr('height', height)
    .append('g')
  // 同じディレクトリにあるgeojsonファイルをhttp経由で読み込む
  d3.json('osakapref.json')
    .then(function(json) {
      let projection, path
      // 市区町村表示領域を生成
      const tooltip = d3
        .select('body')
        .append('div')
        .attr('class', 'tip')
      // 投影を処理する関数を用意する。データからSVGのPATHに変換するため。
      projection = d3
        .geoMercator()
        .scale(scale)
        .center(d3.geoCentroid(json)) // データから中心点を計算 .center(d3.geoCentroid(json))
        .translate([width / 2, height / 2]) // ブラウザの中央に転移
      // pathジェネレータ関数
      path = d3.geoPath().projection(projection)
      // これがenterしたデータ毎に呼び出されpath要素のd属性にgeoJSONデータから変換した値を入れて市町村境界描画
      map
        .selectAll('path')
        .data(json.features)
        .enter()
        .append('path')
        .attr('d', path)
        // 陽性者に対応した色で境界内を塗る
        .style('fill', function(d) {
          return pop_data[d.properties.index][2]
        })
      // 左側にデータ表示
      for (let i = 0; i < 43; i++) {
        map
          .append('text')
          .attr({
            x: 20,
            y: i * 13 + 20
          })
          .style('font-size', 12 + 'px')
          .text(pop_data[i][0] + ':' + pop_data[i][1])
      }

      // 市町村名表示
      const xhr = new XMLHttpRequest()
      xhr.onload = function() {
        const tempArray = xhr.responseText.split('\n')
        csvArray = new Array()
        for (let i = 0; i < tempArray.length; i++) {
          csvArray[i] = tempArray[i].split(',')
          const data = csvArray[i]
          const lonlat = [data[1], data[2]]
          const xy = projection(lonlat)
          map
            .append('text')
            .attr({
              x: xy[0] - 15,
              y: xy[1]
            })
            .style('font-size', 10 + 'px')
            .text(data[0])
        }
      }
      xhr.open('get', 'cityname.csv', true)
      xhr.send(null)
    })
    .catch(function(error) {
      // エラー処理
      console.log('in drawOsaka() error:', error)
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
$infected-level1: #ccfbcc;
$infected-level2: #88f2a9;
$infected-level3: #44e5b7;
$infected-level4: #00c1d5;
$infected-level5: #004da5;
$infected-level6: #000072;

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

svg {
  max-height: 600px;
}
</style>

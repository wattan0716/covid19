<template>
  <div class="MainPage">
    <page-header
      :icon="headerItem.icon"
      :title="headerItem.title"
      :date="headerItem.date"
    />
    <whats-new class="mb-4" :items="newsItems" />
    <static-info
      class="mb-4"
      :url="'http://www.pref.osaka.lg.jp/iryo/osakakansensho/corona-denwa.html'"
      :text="$t('自分や家族の症状に不安や心配があればまずは電話相談をどうぞ')"
    />
    <v-row class="DataBlock">
      <v-col cols="12" md="6" class="DataCard">
        <svg-card
          :title="$t('検査陽性者の状況')"
          :title-id="'details-of-confirmed-cases'"
          :date="Data.inspections_summary.date"
        >
          <confirmed-cases-table v-bind="confirmedCases" />
        </svg-card>
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="$t('陽性者数')"
          :title-id="'number-of-confirmed-cases'"
          :chart-id="'time-bar-chart-patients'"
          :chart-data="patientsGraph"
          :date="Data.patients.date"
          :unit="$t('人')"
          :url="$t('./data/summary.csv')"
          :note="
            $t('※11月15日までは再陽性者を除き、11月16日以降は再陽性者を含む')
          "
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <osaka-chart-1
          :title="$t('重症病床使用率')"
          :title-id="'osaka-chart1-1'"
          :chart-id="'osaka-chart1-1'"
          :chart-data="osakaGraph1"
          :date="osakaGraph1[4]"
          :items="osakaItems1"
          :unit="$t('人')"
          :url="$t('./data/summary.csv')"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <osaka-chart-1
          :title="$t('軽症中等症病床使用率')"
          :title-id="'osaka-chart1-2'"
          :chart-id="'osaka-chart1-2'"
          :chart-data="osakaGraph2"
          :date="osakaGraph2[4]"
          :items="osakaItems2"
          :unit="$t('人')"
          :url="$t('./data/summary.csv')"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <osaka-chart-2
          :title="$t('直近1週間の人口10万人あたり新規陽性者数')"
          :title-id="'osaka-chart2-1'"
          :chart-id="'osaka-chart2-1'"
          :chart-data="osakaGraph3"
          :date="osakaGraph3[2]"
          :unit="$t('人')"
          :url="$t('./data/summary.csv')"
        />
      </v-col>
      <!-- 
      <v-col cols="12" md="6" class="DataCard">
        <data-table
          :title="$t('陽性者の属性')"
          :title-id="'attributes-of-confirmed-cases'"
          :chart-data="patientsTable"
          :chart-option="{}"
          :date="Data.patients.date"
          :info="sumInfoOfPatients"
          :unit="$t('人')"
          :url="$t('./data/patients.csv')"
        />
      </v-col>
      -->
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="$t('退院・解除済累計')"
          :title-id="'number-of-treated'"
          :chart-id="'time-bar-chart-inspections'"
          :chart-data="treatedGraph"
          :date="Data.treated_summary.date"
          :unit="$t('人')"
          :url="$t('./data/summary.csv')"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="$t('検査実施件数')"
          :title-id="'number-of-tested'"
          :chart-id="'time-bar-chart-inspections'"
          :chart-data="inspectionsGraph"
          :date="Data.inspections_summary.date"
          :unit="$t('件.tested')"
          :url="$t('./data/summary.csv')"
          :note="
            $t(
              '※11月15日までは再陽性検査数を除き、11月16日以降は再陽性検査数を含む'
            )
          "
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-stacked-bar-chart2
          :title="$t('感染経路不明者（リンク不明者）')"
          :title-id="'number-of-transmission-route'"
          :chart-id="'time-stacked-bar-chart2-transmission-route'"
          :chart-data="transmissionRouteGraph"
          :date="Data.transmission_route_summary.date"
          :items="transmissionRouteItems"
          :labels="transmissionRouteLabels"
          :unit="$t('人')"
          :url="$t('./data/summary.csv')"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="$t('発症日別による陽性者数の推移')"
          :title-id="'number-of-onset'"
          :chart-id="'time-bar-chart-onset'"
          :chart-data="onsetGraph"
          :date="Data.onset_summary.date"
          :unit="$t('件.reports')"
          :url="$t('./data/onset.csv')"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-stacked-bar-chart
          :title="$t('新型コロナ受診相談センターへの相談件数')"
          :title-id="'number-of-contacts２'"
          :chart-id="'time-stacked-bar-chart-inspections'"
          :chart-data="contacts2Graph"
          :date="Data.contacts2_summary.date"
          :items="contacts2Items"
          :labels="contacts2Labels"
          :note="
            $t(
              '※各自治体からの報告状況により数値が後日更新される場合があります'
            )
          "
          :unit="$t('件.reports')"
          :url="$t('./data/contacts2.csv')"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="$t('府民向け相談窓口への相談件数')"
          :title-id="'number-of-contacts1'"
          :chart-id="'time-bar-chart-inspections'"
          :chart-data="contactsGraph"
          :date="Data.contacts1_summary.date"
          :unit="$t('件.reports')"
          :url="$t('./data/contacts1.csv')"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script>
import PageHeader from '@/components/PageHeader.vue'
import TimeBarChart from '@/components/TimeBarChart.vue'
import TimeStackedBarChart from '@/components/TimeStackedBarChart.vue'
import TimeStackedBarChart2 from '@/components/TimeStackedBarChart2.vue'
import WhatsNew from '@/components/WhatsNew.vue'
import StaticInfo from '@/components/StaticInfo.vue'
import Data from '@/data/data.json'
import OsakaData1 from '@/data/osaka_data_1.json'
import OsakaData2 from '@/data/osaka_data_2.json'
import OsakaData3 from '@/data/osaka_data_3.json'
import formatGraph from '@/utils/formatGraph'
import formatTable from '@/utils/formatTable'
import formatConfirmedCases from '@/utils/formatConfirmedCases'
import News from '@/data/news.json'
import SvgCard from '@/components/SvgCard.vue'
import ConfirmedCasesTable from '@/components/ConfirmedCasesTable.vue'
import OsakaChart1 from '@/components/OsakaChart1.vue'
import OsakaChart2 from '@/components/OsakaChart2.vue'

export default {
  components: {
    PageHeader,
    TimeBarChart,
    WhatsNew,
    StaticInfo,
    SvgCard,
    ConfirmedCasesTable,
    TimeStackedBarChart,
    TimeStackedBarChart2,
    OsakaChart1,
    OsakaChart2
  },
  data() {
    // 感染者数グラフ
    const patientsGraph = formatGraph(Data.patients_summary.data)
    // 感染者数
    const patientsTable = formatTable(Data.patients.data)
    // 検査実施状況
    const inspectionsGraph = formatGraph(Data.inspections_summary.data)
    // 検査陽性者の状況
    const confirmedCases = formatConfirmedCases(Data.main_summary)
    // 感染経路不明者
    const transmissionRouteGraph = [
      Data.transmission_route_summary.data['感染経路不明者'],
      Data.transmission_route_summary.data['感染経路明確者']
    ]
    const transmissionRouteItems = [
      this.$t('リンク不明'),
      this.$t('リンク確認')
    ]
    const transmissionRouteLabels = Data.transmission_route_summary.labels
    // 府民向け相談窓口相談件数
    const contactsGraph = formatGraph(Data.contacts1_summary.data)
    // 新型コロナ受診相談センターへの相談件数
    const contacts2Graph = [
      Data.contacts2_summary.data['府管轄保健所'],
      Data.contacts2_summary.data['政令中核市保健所']
    ]
    const contacts2Items = [
      this.$t('府管轄保健所'),
      this.$t('政令中核市保健所')
    ]
    const contacts2Labels = Data.contacts2_summary.labels
    // 治療終了者数
    const treatedGraph = formatGraph(Data.treated_summary.data)
    // 発症日別による陽性者数の推移
    const onsetGraph = formatGraph(Data.onset_summary.data)

    const sumInfoOfPatients = {
      lText: patientsGraph[
        patientsGraph.length - 1
      ].cumulative.toLocaleString(),
      sText: this.$t('{date}の累計', {
        date: patientsGraph[patientsGraph.length - 1].label
      }),
      unit: '人'
    }

    // 陽性患者の属性 ヘッダー翻訳
    for (const header of patientsTable.headers) {
      header.text = this.$t(header.value)
    }

    const otherAges = ['10歳未満', '不明', '未就学児', '就学児', '調査中']

    // 陽性患者の属性 中身の翻訳
    for (const row of patientsTable.datasets) {
      row['居住地'] = this.$t(row['居住地'])
      row['性別'] = this.$t(row['性別'])
      row['退院・解除'] = this.$t(row['退院・解除'])

      if (otherAges.includes(row['年代'])) {
        row['年代'] = this.$t(row['年代'])
      } else {
        const age = row['年代'].slice(0, -1)
        row['年代'] = this.$t('{age}代', { age })
      }
    }

    // 重症病床使用率
    const osakaItems1 = ['重症病床確保数', '重症入院患者数', '重症病床使用率']
    const tmpData1 = OsakaData1.data.filter(
      d => new Date(d.date) >= new Date('2020-01-01')
    )
    const dateList1 = tmpData1.map(d => d.date)
    const denominator1 = tmpData1.map(d => d.denominator)
    const numerator1 = tmpData1.map(d => d.numerator)
    const percentage1 = tmpData1.map(d => d.percentage * 100)
    const updated1 = OsakaData1.date
    const osakaGraph1 = [
      dateList1,
      denominator1,
      numerator1,
      percentage1,
      updated1
    ]
    // 軽症中等症病床使用率
    const osakaItems2 = [
      '軽症中等症病床確保数',
      '軽症中等症入院患者数',
      '軽症中等症病床使用率'
    ]
    const tmpData2 = OsakaData2.data.filter(
      d => new Date(d.date) >= new Date('2020-01-01')
    )
    const dateList2 = tmpData2.map(d => d.date)
    const denominator2 = tmpData2.map(d => d.denominator)
    const numerator2 = tmpData2.map(d => d.numerator)
    const percentage2 = tmpData2.map(d => d.percentage * 100)
    const updated2 = OsakaData2.date
    const osakaGraph2 = [
      dateList2,
      denominator2,
      numerator2,
      percentage2,
      updated2
    ]
    // 軽症中等症病床使用率
    const tmpData3 = OsakaData3.data.filter(
      d => new Date(d.date) >= new Date('2020-01-01')
    )
    const dateList3 = tmpData3.map(d => d.date)
    const value3 = tmpData3.map(d => d.value)
    const updated3 = OsakaData3.date
    const osakaGraph3 = [dateList3, value3, updated3]

    const data = {
      Data,
      patientsTable,
      patientsGraph,
      inspectionsGraph,
      confirmedCases,
      transmissionRouteGraph,
      transmissionRouteItems,
      transmissionRouteLabels,
      contactsGraph,
      contacts2Graph,
      contacts2Items,
      contacts2Labels,
      treatedGraph,
      onsetGraph,
      sumInfoOfPatients,
      osakaGraph1,
      osakaItems1,
      osakaGraph2,
      osakaItems2,
      osakaGraph3,
      headerItem: {
        icon: 'mdi-chart-timeline-variant',
        title: this.$t('大阪府の最新感染動向'),
        date: Data.lastUpdate
      },
      newsItems: News.newsItems
    }
    return data
  },
  head() {
    return {
      title: this.$t('大阪府の最新感染動向')
    }
  }
}
</script>

<style lang="scss" scoped>
.MainPage {
  .DataBlock {
    margin: 20px -8px;
    .DataCard {
      @include largerThan($medium) {
        padding: 10px;
      }
      @include lessThan($small) {
        padding: 4px 8px;
      }
    }
  }
}
</style>

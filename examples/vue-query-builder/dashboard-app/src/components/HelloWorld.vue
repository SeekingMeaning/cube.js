<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <v-container>
    <v-row class="text-center">
      <query-builder :cubejs-api="cubejsApi" :query="innerQuery" style="width: 100%">
        <template
          v-slot:builder="{
          measures,
          setMeasures,
          availableMeasures,
          dimensions,
          setDimensions,
          availableDimensions,
          timeDimensions,
          setTimeDimensions,
          availableTimeDimensions,
          // availableFilters,
          filters,
          setFilters,
          // query
          }"
        >
          <v-row>
            <!-- @TODO  Добавить в документацию, что сетим measures и dimensions вот так: [obj.name:string, obj.name:string] -->
            <v-col cols="3">
              <v-select
                multiple
                label="Measures"
                outlined
                :value="measures.map(i => (i.name))"
                @change="setMeasures"
                :items="availableMeasures.map(i => (i.name))"
              />
            </v-col>
            <v-col cols="3">
              <v-select
                multiple
                label="Dimensions"
                outlined
                :value="dimensions.map(i => (i.name))"
                @change="setDimensions"
                :items="availableDimensions.map(i => (i.name))"
              />
            </v-col>
            <v-col cols="3">
              <v-select
                label="ChartType"
                outlined
                v-model="type"
                :items="['line', 'table']"
              />
            </v-col>
          </v-row>
          <v-row>
            <!-- @TODO Добавить в документацию, что сетим мы timeDimension вот так: [{dimension: $event:string, granularity: 'month', dateRange: 'This year'}] -->
            <v-col cols="3">
              <v-select
                label="Time Dimensions"
                :value="timeDimensions[0]['dimension']['name']"
                @change="setTimeDimensions([{dimension: $event, granularity: 'month', dateRange: 'This year'}])"
                :items="availableTimeDimensions.map(i => (i.name))"
              />
            </v-col>
            <v-col cols="3">
              <v-select
                label="Time Dimensions"
                v-model="innerQuery.timeDimensions[0]['granularity']"
                :items="timeDimensions[0]['dimension']['granularities'].map(obj => obj.name)"
              />
            </v-col>
            <v-col cols="3">
              <v-select
                label="Time Dimensions"
                v-model="innerQuery.timeDimensions[0]['dateRange']"
                :items="dateRange"
              />
            </v-col>
          </v-row>
          <FilterComponent :filters="filters" :setFilters="setFilters"></FilterComponent>
        </template>

        <template v-slot="{ resultSet }">
          <v-col cols="12" v-if="resultSet">
            <line-chart v-if="type === 'line'" :data="series(resultSet)"></line-chart>
            <Table v-if="type === 'table'" :data="resultSet"></Table>
          </v-col>
        </template>
      </query-builder>
    </v-row>
  </v-container>
</template>

<script>
import cubejs from '@cubejs-client/core'
import { QueryBuilder } from './vue/src'
import FilterComponent from './FilterComponent.vue'
import Table from './Table'

const API_URL = process.env.NODE_ENV === 'production' ? '' : 'http://localhost:4000'
const CUBEJS_TOKEN =
  'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE1OTQ2NjExMzQsImV4cCI6MTYyNjE5NzEzNH0._sWwksID3MLJxXmqNnECV_A3x7gUcVzSgn4szFox76s'
const cubejsApi = cubejs(CUBEJS_TOKEN, {
  apiUrl: `${API_URL}/cubejs-api/v1`
})

export default {
  name: 'HelloWorld',

  components: {
    QueryBuilder,
    FilterComponent,
    Table
  },
  data: () => {
    const innerQuery = {
      limit: 100,
      measures: [
        'Orders.count'
      ],
      timeDimensions: [
        {
          dimension: 'LineItems.createdAt',
          granularity: 'month',
          "dateRange": "This year"
        }
      ],
      filters: [
        {
          "dimension": "Orders.status",
          "operator": "notEquals",
          "values": [
            "completed"
          ]
        }
      ]
    }

    return {
      cubejsApi,
      innerMeasures: [],
      granularity: ['day', 'week', 'month', 'year'],
      dateRange: ['All time', 'Today', 'Yesterday', 'This week', 'This month', 'This quarter', 'This year'],
      type: 'line',
      innerQuery,
    }
  },
  methods: {
    series (resultSet) {
      const seriesNames = resultSet.seriesNames()
      const pivot = resultSet.chartPivot()
      const series = []
      seriesNames.forEach((e) => {
        const data = pivot.map(p => [p.x, p[e.key]])
        series.push({ name: e.key, data })
      })
      return series
    }
  }
}
</script>

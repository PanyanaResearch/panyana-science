<template>
  <v-layout row wrap>
    <v-flex>
      <v-card color="primary" max-width="700px" class="mx-auto">
        <v-card-text class="headline text-xs-center secondary--text">
          {{ type }} Material Boosts
        </v-card-text>
        <v-card-actions>
          <v-select
            v-model="slot"
            :items="typeOptions"
            :label="`Select an ${type.toLowerCase()} component`"
            color="accent"
            class="secondary--text mx-auto"
            style="max-width: 200px"
            hide-details
            dark
            must-sort
          />
        </v-card-actions>
        <v-data-table
          id="boostTable"
          :headers="baseHeaders.concat(headers[slot])"
          :items="materials"
          :pagination.sync="pagination"
          class="pa-3"
          hide-actions
          dark
        >
          <template slot="items" slot-scope="props">
            <td class="text-xs-right" :style="{ color: props.item.color }">
              {{ props.item.name }}
            </td>
            <td :style="getStyle(props.item.weight, 'weight')">
              {{ props.item.weight }}
            </td>
            <td
              v-for="(value, index) in props.item.boosts[type.toLowerCase()][slot]"
              :id="props.item.name + '_' + index"
              :key="value+'_'+index"
              :style="getStyle(value, 'statValue')"
            >
              {{ round(value * 100, 2) }}%
            </td>
          </template>
        </v-data-table>
      </v-card>
    </v-flex>
  </v-layout>
</template>
<script>
import chroma from 'chroma-js'
export default {
  name: `MaterialsChart`,
  props: {
    type: {
      type: String,
      default: ''
    },
    typeOptions: {
      type: Array,
      default: () => []
    },
    headers: {
      type: Object,
      default: () => {}
    }
  },
  head() {
    return {
      title: `${this.type} Materials`,
      meta: [
        {
          hid: 'og:title',
          name: 'og:title',
          content: `${this.type.toLowerCase()} Materials`
        },
        {
          hid: 'og:description',
          name: 'og:description',
          content: `Shows various materials effects on crafting ${this.type.toLowerCase()}s`
        }
      ]
    }
  },
  data() {
    return {
      pagination: {
        sortBy: 'weight',
        rowsPerPage: -1
      },
      materials: [],
      slot: this.typeOptions[0].value || null,
      baseHeaders: [
        { text: 'Name', value: 'name', width: '110px', align: 'right' },
        {
          text: 'Weight',
          value: 'weight',
          width: '50px',
          align: 'center',
          class: 'px-0'
        }
      ],
      scale: chroma
        .bezier(['lightyellow', 'orange', 'deeppink', 'darkred'])
        .scale(),
      max: {}
    }
  },
  mounted() {
    this.$api
      .get('/materials')
      .then(res => {
        this.materials = res.data
        this.materials.forEach(mat => {
          if (!this.max.weightMax) this.max.weightMax = mat.weight
          else this.max.weightMax = Math.max(this.max.weightMax, mat.weight)
          if (!this.max.weightMin) this.max.weightMin = mat.weight
          else this.max.weightMin = Math.min(this.max.weightMin, mat.weight)
          for (const type in mat.boosts[this.type.toLowerCase()]) {
            if (!this.max[type]) {
              this.max[type] = Math.max(
                ...Object.values(mat.boosts[this.type.toLowerCase()][type])
              )
            } else
              this.max[type] = Math.max(
                this.max[type],
                Math.max(
                  ...Object.values(mat.boosts[this.type.toLowerCase()][type])
                )
              )
          }
        })
      })
      .catch(err => {
        console.error(err)
      })
  },
  methods: {
    getStyle(value, type) {
      switch (type) {
        case 'statValue': {
          const color = this.scale
            .domain([0, this.max[this.slot]])(value)
            .hex()
          return {
            backgroundColor: color,
            color:
              chroma.contrast(this.$vuetify.theme.primary, color) > 4.5
                ? this.$vuetify.theme.primary
                : this.$vuetify.theme.secondary
          }
        }
        case 'weight': {
          const color = this.scale
            .domain([this.max.weightMin, this.max.weightMax])(value)
            .hex()
          return {
            backgroundColor: color,
            color:
              chroma.contrast(this.$vuetify.theme.primary, color) > 4.5
                ? this.$vuetify.theme.primary
                : this.$vuetify.theme.secondary,
            textAlign: 'center',
            borderRight: '5px solid black'
          }
        }
      }
    },
    round(value, decs) {
      return Number(Math.round(value + 'e' + decs) + 'e-' + decs)
    }
  }
}
</script>
<style lang="scss">
</style>
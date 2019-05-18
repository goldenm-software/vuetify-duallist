<template>
  <div>
    <template v-if="$vuetify === undefined">
      <p>{{ i18n.vuetifyNotFound }}</p>
    </template>

    <v-container v-else fluid>
      <v-layout row wrap>
        <v-flex xs12 md5>
          <v-layout row wrap>
            <v-flex xs12 class="pb-0">
              <h4 style="text-align: center; text-transform: uppercase; font-weight: bold;" class="pa-0 ma-0">{{ i18n.list1 }}</h4>
            </v-flex>

            <v-flex xs12 class="pt-0 pb-0">
              <v-text-field
                v-model="search.options"
                :dark="dark"
                :color="dark ? 'white--text' : 'primary'"
                :label="`${i18n.findIn} ${i18n.list1}`"
              />
            </v-flex>

            <v-flex xs12 style="height: 350px; overflow-y: scroll;" class="pa-0 mt-0">
              <v-list :dense="dense" :dark="dark">
                <template v-for="(position, i) in options">
                  <v-list-tile
                    v-if="validFilter(position, search.options)"
                    :key="items[position][keys.primary]"
                    avatar
                    ripple
                    :disabled="disabled"
                    :class="getColor(position)"
                    @click="toggleToSelected(position)"
                  >
                    <v-list-tile-content>
                      <h6 class="pa-0 ma-0">{{ items[position][keys.primary] }}</h6>
                    </v-list-tile-content>

                    <v-list-tile-action>
                      <small style="opacity: 0.7;">{{ items[position][keys.secondary] }}</small>
                    </v-list-tile-action>
                  </v-list-tile>

                  <v-divider
                    v-if="i + 1 < options.length && validFilter(position, search.options)"
                    :key="i"
                  />
                </template>
              </v-list>
            </v-flex>
          </v-layout>
        </v-flex>
        
        <v-flex xs12 md2 style="height: 100%; margin: auto; display: flex; flex-direction: column; align-items: center;">
          <v-btn :disabled="disabled" fab small ripple :color="dark ? 'white' : 'primary'" @click="toggleAll('selected')">
            <v-icon medium>{{ icons.nextAll }}</v-icon>
          </v-btn>

          <v-btn :disabled="disabled" fab small ripple :color="dark ? 'white' : 'primary'" @click="toggleToList('selected')">
            <v-icon medium>{{ icons.next }}</v-icon>
          </v-btn>

          <v-btn :disabled="disabled" fab small ripple :color="dark ? 'white' : 'primary'" @click="toggleToList('options')">
            <v-icon medium>{{ icons.prev }}</v-icon>
          </v-btn>

          <v-btn :disabled="disabled" fab small ripple :color="dark ? 'white' : 'primary'" @click="toggleAll('options')">
            <v-icon medium>{{ icons.prevAll }}</v-icon>
          </v-btn>
        </v-flex>

        <v-flex xs12 md5>
          <v-layout row wrap>
            <v-flex xs12 class="pb-0">
              <h4 style="text-align: center; text-transform: uppercase; font-weight: bold;" class="pa-0 ma-0">{{ i18n.list2 }}</h4>
            </v-flex>

            <v-flex xs12 class="pt-0 pb-0">
              <v-text-field
                v-model="search.selected"
                :dark="dark"
                :color="dark ? 'white--text' : 'primary'"
                :label="`${i18n.findIn} ${i18n.list2}`"
              />
            </v-flex>

            <v-flex xs12 style="height: 350px; overflow-y: scroll;" class="pa-0 mt-0">
              <v-list :dense="dense" :dark="dark">
                <template v-for="(position, i) in selected">
                  <v-list-tile
                    v-if="validFilter(position, search.selected)"
                    :key="items[position][keys.primary]"
                    avatar
                    ripple
                    :disabled="disabled"
                    :class="getColor(position, 'selected')"
                    @click="toggleToOptions(position)"
                  >
                    <v-list-tile-content>
                      <h6 class="pa-0 ma-0">{{ items[position][keys.primary] }}</h6>
                    </v-list-tile-content>

                    <v-list-tile-action>
                      <small style="opacity: 0.7;">{{ items[position][keys.secondary] }}</small>
                    </v-list-tile-action>
                  </v-list-tile>

                  <v-divider
                    v-if="i + 1 < selected.length && validFilter(position, search.selected)"
                    :key="i"
                  />
                </template>
              </v-list>
            </v-flex>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
export default {
  props: {
    items: {
      type: Array,
      required: true
    },
    keys: {
      type: Object,
      required: true
    },
    dense: {
      type: Boolean,
      default () {
        return false
      }
    },
    disabled: {
      type: Boolean,
      default () {
        return false
      }
    },
    dark: {
      type: Boolean,
      default () {
        return false
      }
    },
    preSelected: {
      type: Array,
      default () {
        return []
      }
    },
    i18n: {
      type: Object,
      default () {
        return {
          vuetifyNotFound: 'Vuetify is required',
          list1: 'Options',
          list2: 'Selected',
          findIn: 'Find in',
          noData: 'No records available'
        }
      }
    },
    icons: {
      type: Object,
      default () {
        return {
          next: 'mdi-chevron-right',
          nextAll: 'mdi-chevron-double-right',
          prev: 'mdi-chevron-left',
          prevAll: 'mdi-chevron-double-left'
        }
      }
    }
  },

  data () {
    return {
      options: [],
      selected: [],
      search: {
        options: '',
        selected: ''
      },
      ctrlSelected: false,
      shiftSelected: false,
      tempSelected: [],
      tempOptions: []
    }
  },

  watch: {
    items (newVal, oldVal) {
      this.options = []
      this.selected = []
      for (let i in newVal) {
        if (this.preSelected.includes(newVal[i])) {
          this.selected.push(parseInt(i))
        } else {
          this.options.push(parseInt(i))
        }
      }
    },

    selected (newVal, oldVal) {
      let selected = []

      for (let i in newVal) {
        selected.push(this.items[newVal[i]])
      }

      this.$emit('selected', selected)
    }
  },

  created () {
    for (let i in this.items) {
      if (this.preSelected.includes(this.items[i])) {
        this.selected.push(parseInt(i))
      } else {
        this.options.push(parseInt(i))
      }
    }

    document.addEventListener('keydown', this.validateKey)
    document.addEventListener('keyup', this.removeKey)
  },

  beforeDestroy () {
    document.removeEventListener('keydown', this.validateKey)
    document.removeEventListener('keyup', this.removeKey)
  },

  methods: {
    getColor (index, list = 'options') {
      if (this[(list === 'options' ? 'tempSelected' : 'tempOptions')].includes(parseInt(index))) {
        return 'primary white--text'
      } else {
        return ''
      }
    },

    validateKey (e) {
      if (e.keyCode === 16) {
        this.shiftSelected = true
      } else if (e.keyCode === 91) {
        this.ctrlSelected = true
      }
    },

    removeKey (e) {
      if ([16, 91].includes(e.keyCode)) {
        this.ctrlSelected = false
        this.shiftSelected = false
      }
    },

    validFilter (pos, search) {
      if (search === '') {
        return true
      }

      if (this.items[pos][this.keys.primary].toLowerCase().includes(search.toLowerCase())) {
        return true
      }

      if (this.items[pos][this.keys.secondary].toLowerCase().includes(search.toLowerCase())) {
        return true
      }
    },

    toggleToList (destination) {
      if (destination === 'selected') {
        for (let i in this.tempSelected) {
          let item = this.tempSelected[i]

          if (this.selected.indexOf(item) === -1) {
            this.selected.push(item)
          }

          let pos = this.options.indexOf(item)

          this.options.splice(pos, 1)
        }

        this.tempSelected = []
      } else {
        for (let i in this.tempOptions) {
          let item = this.tempOptions[i]

          if (this.options.indexOf(item) === -1) {
            this.options.push(item)
          }

          let pos = this.selected.indexOf(item)

          this.selected.splice(pos, 1)
        }

        this.tempOptions = []
      }

      this.sort()
    },

    toggleToSelected (index) {
      index = parseInt(index)

      if (this.tempSelected.includes(index)) {
        let pos = this.tempSelected.indexOf(index)
        this.tempSelected.splice(pos, 1)
      } else if (!this.shiftSelected && !this.ctrlSelected) {
        this.tempSelected = []
        this.tempSelected.push(index)
      } else if (this.shiftSelected && this.tempSelected.length === 0) {
        this.tempSelected.push(index)
      } else if (this.shiftSelected && this.tempSelected.length > 0) {
        let flag = []

        let first = this.tempSelected[0]

        if (index > first) {
          for (let i = first; i <= index; i++) {
            flag.push(i)
          }

          this.tempSelected = flag
        } else {
          for (let i = index; i <= first; i++) {
            flag.push(i)
          }

          this.tempSelected = flag
        }
      } else if (this.ctrlSelected && this.tempSelected.length === 0) {
        this.tempSelected.push(index)
      } else if (this.ctrlSelected && this.tempSelected.length > 0) {
        this.tempSelected.push(index)
      }
    },

    toggleToOptions (index) {
      index = parseInt(index)

      if (this.tempOptions.includes(index)) {
        let pos = this.tempOptions.indexOf(index)
        this.tempOptions.splice(pos, 1)
      } else if (!this.shiftSelected && !this.ctrlSelected) {
        this.tempOptions = []
        this.tempOptions.push(index)
      } else if (this.shiftSelected && this.tempOptions.length === 0) {
        this.tempOptions.push(index)
      } else if (this.shiftSelected && this.tempOptions.length > 0) {
        let flag = []

        let first = this.tempOptions[0]

        if (index > first) {
          for (let i = first; i <= index; i++) {
            flag.push(i)
          }

          this.tempOptions = flag
        } else {
          for (let i = index; i <= first; i++) {
            flag.push(i)
          }

          this.tempOptions = flag
        }
      } else if (this.ctrlSelected && this.tempOptions.length === 0) {
        this.tempOptions.push(index)
      } else if (this.ctrlSelected && this.tempOptions.length > 0) {
        this.tempOptions.push(index)
      }
    },

    toggleAll (destination) {
      let origin = destination === 'selected' ? 'options' : 'selected'
      
      for (let i in this[origin]) {
        this[destination].push(this[origin][i])
      }

      this[origin] = []

      this.sort()
    },

    sort () {
      if (this.selected.length > 0) {
        this.selected.sort((a, b) => {
          try {
            return this.items[a][this.keys.primary].localeCompare(this.items[b][this.keys.primary])
          } catch (e) {}
        })
      }

      if (this.options.length > 0) {
        this.options.sort((a, b) => {
          try {
            return this.items[a][this.keys.primary].localeCompare(this.items[b][this.keys.primary])
          } catch (e) {}
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .dual-table {
    width: 100%;
    background: rgba(0,0,0,0.2);

    .dual-table-row {
      background: #fff;

      td {
        padding: 5px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        transition: all 200ms ease;

        .dual-table-subtitle {
          font-size: 12px;
          color: rgba(0,0,0,0.5);
        }

        &:hover {
          background: var(--v-primary-base);
          color: #fff;

          .dual-table-subtitle {
            color: #fff !important;
          }
        }
      }
    }
  }
</style>
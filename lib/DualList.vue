<template>
  <div>
    <template v-if="$vuetify === undefined">
      <p>{{ i18n.vuetifyNotFound }}</p>
    </template>

    <v-container v-else fluid>
      <v-layout row wrap pa-2>
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
                <template v-for="(item, i) in options">
                  <v-list-item
                    v-if="validFilter(item, search.options)"
                    :key="`option-${i}`"
                    ripple
                    :disabled="disabled"
                    :class="getColor(item, 'options')"
                    @click="toggleToSelected(item)"
                  >
                    <v-list-item-content>
                      <h6 class="pa-0 ma-0">{{ item[itemText] }}</h6>
                    </v-list-item-content>

                    <v-list-item-action v-if="itemAlt !== ''">
                      <small style="opacity: 0.7;">{{ item[itemAlt] }}</small>
                    </v-list-item-action>
                  </v-list-item>

                  <v-divider
                    v-if="i + 1 < options.length && validFilter(item, search.options)"
                    :key="i"
                  />
                </template>
              </v-list>
            </v-flex>
          </v-layout>
        </v-flex>
        
        <v-flex xs12 md2 style="height: 100%; margin: auto; display: flex; flex-direction: column; align-items: center;">
          <v-btn :disabled="disabled" :text="dark" fab small :color="dark ? 'white' : 'primary'" @click="toggleAll('selected')">
            <v-icon medium>{{ icons.nextAll }}</v-icon>
          </v-btn>

          <v-btn :disabled="disabled" :text="dark" fab small :color="dark ? 'white' : 'primary'" @click="toggleToList('selected')">
            <v-icon medium>{{ icons.next }}</v-icon>
          </v-btn>

          <v-btn :disabled="disabled" :text="dark" fab small :color="dark ? 'white' : 'primary'" @click="toggleToList('options')">
            <v-icon medium>{{ icons.prev }}</v-icon>
          </v-btn>

          <v-btn :disabled="disabled" :text="dark" fab small :color="dark ? 'white' : 'primary'" @click="toggleAll('options')">
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
                <template v-for="(item, i) in selectedInternal">
                  <v-list-item
                    v-if="validFilter(item, search.selected)"
                    :key="`selected-${i}`"
                    ripple
                    :disabled="disabled"
                    :class="getColor(item, 'selected')"
                    @click="toggleToOptions(item)"
                  >
                    <v-list-item-content>
                      <h6 class="pa-0 ma-0">{{ item[itemText] }}</h6>
                    </v-list-item-content>

                    <v-list-item-action v-if="itemAlt !== ''">
                      <small style="opacity: 0.7;">{{ item[itemAlt] }}</small>
                    </v-list-item-action>
                  </v-list-item>

                  <v-divider
                    v-if="i + 1 < selected.length && validFilter(item, search.selected)"
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
  model: {
    prop: 'selected',
    event: 'change'
  },

  props: {
    selected: {
      type: Array
    },

    items: {
      type: Array,
      required: false,
      default () {
        return []
      }
    },

    itemText: {
      type: String,
      required: false,
      default () {
        return 'text'
      }
    },

    itemAlt: {
      type: String,
      required: false,
      default () {
        return ''
      }
    },

    itemValue: {
      type: String,
      required: false,
      default () {
        return ''
      }
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
      loaded: false,
      search: {
        options: '',
        selected: ''
      },

      temporal: {
        options: [],
        selected: []
      },

      options: [],
      selectedInternal: [],

      ctrlSelected: false,
      shiftSelected: false
    }
  },

  watch: {
    /*
     * =================================
     *  Update available options
     * =================================
    */
    items (newVal, oldVal) {
      this.options = []
      for (let i in newVal) {
        if (!this.selectedInterval.includes(newVal[i])) {
          this.options.push(Object.assign({}, newVal[i]))
        }
      }
    },

    /*
     * =================================
     *  Broadcast to parent
     * =================================
    */
    selectedInternal (newVal, oldVal) {
      this.$emit('change', newVal.map((item, index, arr) => {
        if (this.itemValue === '') {
          return item
        } else {
          return item[this.itemValue]
        }
      }))
    },

    // /*
    //  * =================================
    //  *  Update selected items
    //  * =================================
    // */
    // selected (newVal, oldVal) {
    //   this.selectedInternal = []

    //   const items = this.options.filter((item, index, arr) => {
    //     if (this.itemValue === '') {
    //       return item
    //     } else {
    //       if (newVal.includes(item[this.itemValue])) {
    //         return item
    //       }
    //     }
    //   })

    //   for (let i in newVal) {
    //     if (options.includes(newVal[i])) {
    //       this.selectedInternal.push(Object.assign({}, newVal[i]))
    //       this.options.splice(this.options.indexOf(items.indexOf(newVal[i])), 1)
    //     }
    //   }
    // }
  },

  computed: {
    /*
     * =================================
     *  Get dark state
     * =================================
    */
    dark () {
      return this.$vuetify.theme.dark
    }
  },

  created () {
    this.options = Object.values(Object.assign({}, this.items))

    document.addEventListener('keydown', this.validateKey)
    document.addEventListener('keyup', this.removeKey)
  },

  beforeDestroy () {
    document.removeEventListener('keydown', this.validateKey)
    document.removeEventListener('keyup', this.removeKey)
  },

  methods: {
    /*
     * =================================
     *  Get active color
     *  @param item Object
     *  @param list String
     * =================================
    */
    getColor (item, list = 'options') {
      if (this.temporal[(list === 'options' ? 'selected' : 'options')].includes(item)) {
        return 'primary white--text'
      } else {
        return ''
      }
    },

    /*
     * =================================
     *  Key selector
     *  @param e Event
     * =================================
    */
    validateKey (e) {
      if (e.keyCode === 16) {
        this.shiftSelected = true
      } else if (e.keyCode === 91) {
        this.ctrlSelected = true
      }
    },

    /*
     * =================================
     *  Remove key selector
     *  @param e Event
     * =================================
    */
    removeKey (e) {
      if ([16, 91].includes(e.keyCode)) {
        this.ctrlSelected = false
        this.shiftSelected = false
      }
    },

    /*
     * =================================
     *  Filter validator
     *  @param item Object
     *  @param search String
     * =================================
    */
    validFilter (item, search) {
      // If search variable is empty
      if (search === '') {
        return true
      }

      // If item text (primary) contain search value
      if (item[this.itemText].toLowerCase().includes(search.toLowerCase())) {
        return true
      }

      // If item alt (secondary) contain search value
      if (item[this.itemAlt].toLowerCase().includes(search.toLowerCase())) {
        return true
      }
    },

    /*
     * =================================
     *  Toggle temporal options to final list
     *  @param destination String
     * =================================
    */
    toggleToList (destination) {
      // Validates destination list
      if (destination === 'selected') {
        // Iterate over selected temporal list
        for (let i in this.temporal.selected) {
          // Get index of temporal item from selected list
          const validator = this.selectedInternal.indexOf(this.temporal.selected[i])

          // Get index of temporal item from options list
          const index = this.options.indexOf(this.temporal.selected[i])

          // Validates if that item exists in final selected list
          if (validator === -1) {
            this.selectedInternal.push(Object.assign({}, this.options[index]))
          }

          // Remove that index in options list
          this.options.splice(index, 1)
        }
      } else {
        // Iterate over options temporal list
        for (let i in this.temporal.options) {
          // Get index of temporal item from options list
          const validator = this.options.indexOf(this.temporal.options[i])

          // Validates if that item exists in final options list
          if (index === -1) {
            this.options.push(this.temporal.options[i])
          }

          // Get index of temporal item from options list
          const index = this.selectedInternal.indexOf(this.temporal.options[i])

          // Remove that index in options list
          this.selectedInternal.splice(index, 1)
        }
      }
    },

    /*
     * =================================
     *  Temporal select item to selected temporal list
     *  @param item Object
     * =================================
    */
    toggleToSelected (item) {
      // Validates if that item exists in selected temporal list
      if (this.temporal.selected.includes(item)) {
        // Get position of that item in temporal selected
        const index = this.temporal.selected.indexOf(item)

        // Delete that item from selected temporal list
        this.temporal.selected.splice(index, 1)
      } else if (!this.shiftSelected && !this.ctrlSelected) { // Validates if shift and control is not selected
        // Clear selected temporal list
        this.temporal.selected = []

        // Push that item into empty selected temporal list
        this.temporal.selected.push(item)
      } else if (this.shiftSelected) { // Validates if Shift is pressed
        // Validates if selected temporal list is empty
        if (this.temporal.selected.length === 0) {
          this.temporal.selected.push(item) // Push that item into selected temporal list
        } else { // If that list is not empty
          const firstIndex = this.options.indexOf(this.temporal.selected[0]) // Get first index from options list

          const currentIndex = this.options.indexOf(item) // Get current index from options list

          // Initialize required variables
          let origin = null, destination = null, temporal = []

          // Validates what is more greater
          if (currentIndex > firstIndex) { // If current index is greater than first index
            origin = firstIndex
            destination = currentIndex
          } else { // If current index is less than first indexx
            origin = currentIndex
            destination = firstIndex
          }

          // Iterate over origin and destination
          for (let i = origin; i <= destination; i++) {
            // Push options into temporal variable
            temporal.push(this.options[i])
          }

          // Assign temporal variable to selected temporal list
          this.temporal.selected = Object.values(Object.assign({}, temporal))

          // Clear temporal variable
          temporal = []
        }

      } else if (this.ctrlSelected) { // Validates if control is pressed
        this.temporal.selected.push(item) // Push that item into selected temporal list
      }
    },

    /*
     * =================================
     *  Temporal select item to options temporal list
     *  @param item Object
     * =================================
    */
    toggleToOptions (item) {
      // Validates if that item exists in options temporal list
      if (this.temporal.options.includes(item)) {
        // Get position of that item in temporal options
        const index = this.temporal.options.indexOf(item)

        // Delete that item from options temporal list
        this.temporal.options.splice(index, 1)
      } else if (!this.shiftoptions && !this.ctrloptions) { // Validates if shift and control is not options
        // Clear options temporal list
        this.temporal.options = []

        // Push that item into empty options temporal list
        this.temporal.options.push(item)
      } else if (this.shiftoptions) { // Validates if Shift is pressed
        // Validates if options temporal list is empty
        if (this.temporal.options.length === 0) {
          this.temporal.options.push(item) // Push that item into options temporal list
        } else { // If that list is not empty
          const firstIndex = this.options.indexOf(this.temporal.options[0]) // Get first index from options list

          const currentIndex = this.options.indexOf(item) // Get current index from options list

          // Initialize required variables
          let origin = null, destination = null, temporal = []

          // Validates what is more greater
          if (currentIndex > firstIndex) { // If current index is greater than first index
            origin = firstIndex
            destination = currentIndex
          } else { // If current index is less than first indexx
            origin = currentIndex
            destination = firstIndex
          }

          // Iterate over origin and destination
          for (let i = origin; i <= destination; i++) {
            // Push options into temporal variable
            temporal.push(this.options[i])
          }

          // Assign temporal variable to options temporal list
          this.temporal.options = Object.values(Object.assign({}, temporal))

          // Clear temporal variable
          temporal = []
        }

      } else if (this.ctrloptions) { // Validates if control is pressed
        this.temporal.options.push(item) // Push that item into options temporal list
      }
    },

    /*
     * =================================
     *  Toggle all elements between lists
     *  @param destination String
     * =================================
    */
    toggleAll (destination) {
      let origin = destination === 'selected' ? 'options' : 'selectedInternal'
      
      destination = destination === 'selected' ? 'selectedInternal' : 'options' 

      this[destination] = Object.values(Object.assign({}, this[origin]))
      
      this[origin] = []
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
        align-options: center;
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
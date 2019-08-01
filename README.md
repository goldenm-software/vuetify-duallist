# vuetify-dual-list

vuetify-dual-list is a Vuetify component for VueJS.

## Installation

Use the package manager [yarn](https://yarnpkg.com/en/docs/getting-started) to install vuetify-dual-list.

```bash
yarn add vuetify-dual-list
```

## Usage

```vue
<template>
  <dual-list :icons="icons" :items="items" v-model="selected" item-text="value" item-value="key" :dense="dense" />
</template>

<script>
import DualList from 'vuetify-dual-list'

export default {
  components: {
    DualList
  },

  data () {
    return {
      items: [], // Required
      i18n: { // Optional
        vuetifyNotFound: '',
        list1: '',
        list2: '',
        findIn: '',
        noData: ''
      },
      icons: { // Optional
        prevAll: '',
        prev: '',
        next: '',
        nextAll: ''
      },
      selected: [] // Required - NOTE: Actually we doesn't support all functions related to v-model, it means, if you update "selected", the component didn't update automaticly
      dense: false // Optional
    }
  }
}
</script>
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
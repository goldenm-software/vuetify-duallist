# vuetify-dual-liast

vuetify-dual-list is a Vuetify component for VueJS.

## Installation

Use the package manager [yarn](https://yarnpkg.com/en/docs/getting-started) to install vuetify-dual-list.

```bash
yarn add vuetify-dual-list
```

## Usage

```vue
<template>
  <dual-list :items="items" :keys="keys" :i18n="i18n" :dense="dense" :dark="dark" @selected="attachItems" />
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
      keys: { primary: '', secondary: '' }, // Required
      i18n: { // Optional
        vuetifyNotFound: '',
        list1: '',
        list2: '',
        findIn: '',
        noData: ''
      },
      dense: false, // Optional
      dark: false // Optional
    }
  },

  methods: {
    attachItems (newItems) {
      // ...
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
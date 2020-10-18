<template>
  <v-container>
    <client-only>
      <CoordinateField
        v-model="coordinate"
        clearable
        dense
        label="Coordinate"
        outlined
      />
    </client-only>
  </v-container>
</template>

<script>
import CoordinateField from '~/components/vuetify/CoordinateField'

export default {
  components: {
    CoordinateField
  },
  data() {
    return {
      coordinate: '-6.229728,106.6894312'
    }
  }
}
</script>

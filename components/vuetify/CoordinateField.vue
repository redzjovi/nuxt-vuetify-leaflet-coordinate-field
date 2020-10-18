<template>
  <div>
    <v-text-field
      @change="textChange($event)"
      @click:prepend-inner="textPrependInnerClick($event)"
      v-bind="$attrs"
      v-model="text.value"
      autocomplete="off"
      prepend-inner-icon="mdi-map-marker"
    >
    </v-text-field>

    <v-dialog
      :fullscreen="$vuetify.breakpoint.xsOnly"
      v-model="dialog"
      max-width="600"
      scrollable
    >
      <v-card>
        <l-map
          ref="lmap"
          @drag="mapDrag($event)"
          :center="map.center"
          :options="map.options"
          :zoom="18"
          style="height: 600px;"
        >
          <l-tile-layer :url="map.tileLayer.url"></l-tile-layer>
          <l-control position="topleft">
            <v-autocomplete
              @change="mapSearchChange($event)"
              :items="map.search.items"
              :loading="map.search.loading"
              :search-input.sync="map.search.s"
              background-color="white"
              clearable
              dense
              hide-no-data
              item-text="label"
              label="Search"
              return-object
              solo
            >
            </v-autocomplete>
          </l-control>
          <l-control-fullscreen v-if="$vuetify.breakpoint.smAndUp">
          </l-control-fullscreen>
          <l-marker :lat-lng="[map.marker.latitude, map.marker.longitude]">
          </l-marker>
          <l-control position="bottomleft">
            <v-btn @click="mapConfirmClick()" color="primary">
              Confirm
            </v-btn>
          </l-control>
          <l-control-zoom position="bottomright"></l-control-zoom>
          <l-control position="bottomright">
            <v-btn @click="mapGpsClick()" fab x-small>
              <v-icon>mdi-crosshairs-gps</v-icon>
            </v-btn>
          </l-control>
        </l-map>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      default: '0,0',
      type: String
    }
  },
  data() {
    return {
      dialog: false,
      map: {
        center: [0, 0],
        marker: {
          latitude: 0,
          longitude: 0
        },
        options: {
          zoomControl: false
        },
        search: {
          debounce: null,
          items: [],
          loading: false,
          s: null
        },
        tileLayer: {
          url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
        }
      },
      text: {
        value: this.value
      }
    }
  },
  watch: {
    'map.search.s': function(value) {
      this.map.search.loading = true
      clearTimeout(this.map.search.debounce)
      this.map.search.debounce = setTimeout(() => {
        this.mapSearchItemsFetch(value)
        this.map.search.loading = false
      }, 1000)
    },
    value: function(value) {
      const values = value ? value.split(',') : []
      const latitude = parseFloat(values[0]) || 0
      const longitude = parseFloat(values[1]) || 0
      this.map.center = [latitude, longitude]
      this.map.marker.latitude = latitude
      this.map.marker.longitude = longitude
      this.text.value = value
    }
  },
  mounted() {
    if (this.value) {
      const values = this.value.split(',')
      const latitude = parseFloat(values[0]) || 0
      const longitude = parseFloat(values[1]) || 0
      this.map.center = [latitude, longitude]
      this.map.marker.latitude = latitude
      this.map.marker.longitude = longitude
    }
  },
  methods: {
    mapConfirmClick() {
      this.dialog = false
      this.$emit(
        'input',
        [this.map.marker.latitude, this.map.marker.longitude].join(',')
      )
    },
    mapDrag($event) {
      const center = $event.target.getCenter()
      this.map.marker.latitude = center.lat
      this.map.marker.longitude = center.lng
    },
    mapGpsClick() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(position => {
          this.map.center = [
            position.coords.latitude,
            position.coords.longitude
          ]
          this.map.marker.latitude = position.coords.latitude
          this.map.marker.longitude = position.coords.longitude
        })
      } else {
        console.log('Geolocation is not supported by this browser.')
      }
    },
    mapSearchChange(payload) {
      if (payload) {
        this.map.center = [payload.y, payload.x]
        this.map.marker.latitude = payload.y
        this.map.marker.longitude = payload.x
      }
    },
    async mapSearchItemsFetch(value) {
      let results = []

      if (value) {
        results = await this.$leafletGeosearch.provider.search({
          query: value
        })
      }

      this.map.search.items = results
    },
    textChange(value) {
      this.$emit('input', value)
    },
    async textPrependInnerClick() {
      this.dialog = true
      await this.$nextTick()
      this.$refs.lmap.mapObject.invalidateSize()
    }
  }
}
</script>

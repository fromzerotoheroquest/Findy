<template>
  <v-row class="adjustment" justify="center">
    <v-col cols="12" class="mt-5" align="center">
      <NuxtLink to="/search" class="">
        <LogoFindy class="logo-search  " />
      </NuxtLink>
    </v-col>
    <v-col cols="12" xs="12" sm="12" md="10">
      <v-col cols="12" sm="8" md="8" lg="6" class="mx-auto">
        <v-text-field
          v-model="place"
          label="search for places"
          type="text"
          filled
          rounded
          outlined
          prepend-inner-icon="mdi-magnify"
          color="#af429a"
          class=""
          dense
          @keyup="geosearch () "
        />
        <div class="faded-effect">
          <div v-for="(location, idx) in geoResults " :key="idx" class="px-5">
            <a
              class="suggestions"
              @click="showRealTime(location.y,location.x), showForecast(location.y,location.x), showChoice(location.label)"
            >
              {{ location.label }}
            </a>
          </div>
        </div>
      </v-col>
      <v-col cols="12" class="px-0">
        <LeafletMap v-if="forecastResults" :coordinates="coordinates" class="map" />
        <LeafletMap v-else class="map" />
      </v-col>
    </v-col>

    <!-- to be deleted -->

    <!-- <v-col v-if="forecastResults">
      <pre>
      {{ realTimeResults }}</pre> <br>
      <pre>Rain probability {{ forecastResults.list[10].pop }}
      </pre>
    </pre>
    </v-col> -->

    <v-col cols="12" xs="12" sm="4" md="3">
      <RealTime v-if="forecastResults" :realtime="realTimeResults" :selectedplace="selectedPlaceShort" class="mt-5" />
    </v-col>
    <v-col cols="12" xs="12" sm="8" md="7">
      <TabComponent
        v-if="forecastResults"
        :forecast="forecastResults.list"
        :selectedplace="selectedPlaceFull"
        class="mt-5"
      />
    </v-col>
  </v-row>
</template>

<script>
import { OpenStreetMapProvider } from 'leaflet-geosearch'
import API from '@/services/API'
import RealTime from '~/components/RealTime.vue'
import LeafletMap from '@/components/LeafletMap.vue'
import TabComponent from '@/components/TabComponent.vue'
import LogoFindy from '~/components/LogoFindy.vue'

export default {
  name: 'SearchPage',
  components: {
    RealTime,
    LeafletMap,
    TabComponent,
    LogoFindy
  },
  data () {
    return {
      provider: null,
      place: '',
      realTimeResults: null,
      forecastResults: null,
      geoResults: [],
      // geoCoords: '',
      selectedPlaceShort: '',
      selectedPlaceFull: '',
      coordinates: {
        latitude: '',
        longitude: ''
      }
    }
  },
  mounted () {
    this.provider = new OpenStreetMapProvider()
  },
  methods: {
    async showRealTime (lat, lon) {
      if (this.place.length > 0) {
        this.realTimeResults = await API.searchRealTime(lat, lon)
        this.coordinates.latitude = lat
        this.coordinates.longitude = lon
        this.geoResults = []
        this.place = ''
      }
    },
    async showForecast (lat, lon) {
      if (this.place.length > 0) {
        this.forecastResults = await API.searchForecast(lat, lon)
        this.coordinates.latitude = lat
        this.coordinates.longitude = lon
        console.log(`Result of the request ${this.forecastResults}`)
        this.geoResults = []
        this.place = ''
      }
    },
    async geosearch () {
      this.geoResults = await this.provider.search({ query: this.place })
    },
    // asignCoord (lat, long) {
    //   this.geoCoords = lat.toString() + ',' + long.toString()
    //   console.log(`Geosearch result: ${this.geoResults[0]}`)
    // },
    showChoice (suggestion) {
      this.selectedPlaceFull = suggestion
      this.selectedPlaceShort = this.shorten(suggestion)
    },
    shorten (sth) {
      const chunk = sth.split(',')
      const LastChunkIndex = chunk.length - 1
      return [chunk[0], chunk[LastChunkIndex]].toString()
    }
  }
}
</script>

<style scoped>
.suggestions {
  color: #af429a;
}

.faded-effect {
  background: linear-gradient(to top, #fff, #af429a);
  background-clip: text;
  -webkit-text-fill-color: transparent;
  max-height: 150px;
}

.map {
  border: 1px solid #af429a;
}

.logo-search {
  width: 250px;
}

.adjustment {
  flex: 0 0 auto;
}
</style>

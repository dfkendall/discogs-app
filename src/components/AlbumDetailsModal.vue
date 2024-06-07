<template>
  <v-card class="pa-1" min-height="500" max-height="100vh">
    <v-row class="d-flex justify-end" style="max-height: 60px">
      <v-btn class="mr-3 mt-3" height="30" @click="close">
        <v-icon size="25">mdi-close</v-icon>
      </v-btn>
    </v-row>
    <v-skeleton-loader
      v-if="loading"
      class="mx-auto"
      type="image, article"
    ></v-skeleton-loader>
    <div v-else>
      <v-row>
        <v-col lg="6" sm="12">
          <v-card-title class="text-center font-weight-bold"><a class="text-decoration-none" target="_blank" :href="additionalAlbumDetails.uri">{{title}}</a></v-card-title>
          <v-card-text class="text-center"><a class="text-decoration-none" target="_blank" :href="additionalArtistDetails.uri">{{subtitle}}</a></v-card-text>
          <div class="d-flex justify-content-center">
            <v-img width="300" height="250" :src="image"></v-img>
          </div>
        </v-col>
        <v-col>
          <v-card-text>Genre: {{genre}}</v-card-text>
          <v-card-text class="mt-n4">{{`${format} | ${description}`}}</v-card-text>
          <v-card-text>
            <p class="text-decoration-underline font-weight-bold">Tracklist:</p>
            <v-list v-for="(track, index) in additionalAlbumDetails.tracklist" :key="track.id">
              <v-list-item>{{`${index + 1}. ${track.title} ${track.duration}`}}</v-list-item>
            </v-list>
          </v-card-text>
          <v-card-text><v-btn class="text-capitalize" style="letter-spacing: 0px; color: #000000 !important" color="#FFFFFF" @click="setAdditionalAlbumDetails" :href="additionalAlbumDetails.uri">View on Discogs</v-btn></v-card-text>
        </v-col>
      </v-row>
      <v-row class="mx-3">
        <v-card-text v-if="additionalAlbums.length > 1" class="mb-n3">More {{subtitle}} in the collection:</v-card-text>
      </v-row>
      <v-row  v-if="additionalAlbums.length > 1" class="mx-3">
        <!-- Need v-if here to not include album shown -->
        <v-col v-for="album in additionalAlbums" :key="album.id" lg="3" sm="12">
          <v-dialog max-width="1000" transition="false">
          <template v-slot:activator="{ props: activatorProps }">
          <v-card color="#666666" class="pa-3" v-bind="activatorProps">
            <div class="text-center">
              <img width="100" contain :src="album.basic_information.cover_image">
              
            </div>
            <v-card-title class="text-center font-weight-bold">{{album.basic_information.title}}</v-card-title>
            <v-card-text class="text-center">{{album.basic_information.artists[0].name}}</v-card-text>
          </v-card>
          </template>
           <template v-slot:default="{ isActive }">
          <album-details-modal 
          :title="album.basic_information.title" 
          :subtitle="album.basic_information.artists[0].name" 
          :image="album.basic_information.cover_image" 
          :genre="album.basic_information.styles[0]" 
          :format="album.basic_information.formats[0].name" 
          :description="album.basic_information.formats[0].descriptions[0]" 
          :year="album.basic_information.year"
          :additionalAlbums="getAdditionalAlbums(album)"
          :resourceURL="album.basic_information.resource_url"
          @close-event="isActive.value = false" />
        </template>
      </v-dialog>
        </v-col>
      </v-row>
    </div>
  </v-card>
</template>

<script setup>
//Imports
import axios from 'axios';
import { ref, onMounted } from 'vue'

//Refs
const additionalAlbumDetails = ref(null)
const additionalArtistDetails = ref(null)
const loading = ref(true)

//Functions
const setAdditionalAlbumDetails = async () => {
   loading.value = true
  await axios.get(props.resourceURL).then(response => {
    additionalAlbumDetails.value = response.data
  })
  await axios.get(additionalAlbumDetails.value.artists[0].resource_url).then(response => {
    additionalArtistDetails.value = response.data
    loading.value = false;
  })
}
const close = () => {
  emit('closeEvent')
}
const getAdditionalAlbums = (value) => {
  var results = props.albumArray.filter(function (item) { return value.basic_information.title != props.title && item.basic_information.artists[0].name === value.basic_information.artists[0].name; });
  return results
}

//Lifecycle
onMounted(() => {
  setAdditionalAlbumDetails()
})

//Props
const props = defineProps({ title: String, subtitle: String, albumArray: Array, image: String, genre: String, format: String, description: String, year: String, additionalAlbums: Array, resourceURL: String })

//Emits
const emit = defineEmits(['closeEvent'])
</script>

<style scoped lang="scss">
  .v-list {
    padding: 0px !important;
    margin-bottom: -15px;
  }
  .v-list-item {
    padding-left: 0px !important;
  }

  a {
    color: #FFFFFF !important;
    &:hover {
      text-decoration: underline !important;
    }
  }
</style>
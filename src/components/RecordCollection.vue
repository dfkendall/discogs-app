<template>
  <v-container>
    <v-row
      class="mx-auto"
      max-width="1200"
      justify="center"
    >
      <v-col cols="12">
      <div class="text-center">
        <h1 class="my-5" style="color: #FFFFFF">Derek's Discogs Collection</h1>
      </div>
      </v-col>
    <v-col cols="12" lg="9" md="12">
    <v-row>
      <v-col cols="12" class="px-0 pb-3">
        <v-card
          class="py-4"
          color="#1f2833"
          rounded="lg"
        >
          <v-card-text>
            <h2 class="text-h5 font-weight-bold text-center mb-5" style="color: #FFFFFF">Explore</h2>
            <v-row>
              <v-col lg="6" sm="12" cols="12">
                <v-select
                  @click="showSearchResultsInfo = false"
                  :items="searchParameters"
                  density="compact"
                  hide-details
                  bg-color="#f2f2f2"
                  item-color="#45a29e"
                  v-model="selectedSearchParameter"
                ></v-select>
              </v-col>
              <v-col lg="6" sm="12" cols="12">
                <v-select
                  @click="showSearchResultsInfo = false"
                  :items="selectedSearchParameterOptions"
                  density="compact"
                  hide-details
                  bg-color="#f2f2f2"
                  item-color="#45a29e"
                  :model-value="selectedSearchValue"
                  v-model="selectedSearchKeyword"
                  ></v-select>
              </v-col>
            </v-row>
            <v-row justify="center" class="mt-10">
              <v-btn color="#66fcf1" style="letter-spacing: 0px;" class="text-capitalize mr-3" @click="searchAlbums" :disabled="!isSearchEnabled">Search</v-btn>
              <v-btn color="#ffffff" style="letter-spacing: 0px;" class="text-capitalize ml-3" @click="removeFilter" :disabled="!albumsFiltered">Clear</v-btn> 
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    </v-col>
    <v-col cols="12" lg="9" md="12">
    <!-- Need to lazy load these -->
    <p v-if="showSearchResultsInfo" class="my-5">{{`Showing ${filteredAlbumArray.length} result(s) for "${selectedSearchParameter} - ${selectedSearchKeyword}"`}}</p>
    <v-row v-for="record in filteredAlbumArray" :key="record.id"> 
      <v-dialog max-width="1000" transition="false">
        <template v-slot:activator="{ props: activatorProps }">
          <album-results-item 
          v-bind="activatorProps" 
          :title="record.basic_information.title" 
          :subtitle="record.basic_information.artists[0].name" 
          :image="record.basic_information.cover_image" 
          :genre="getGenres(record.basic_information.styles)" 
          :format="record.basic_information.formats[0].name" 
          :description="record.basic_information.formats[0].descriptions[0]" 
          :year="record.basic_information.year"
          @close-event="isActive.value = false" />
        </template>
        <template v-slot:default="{ isActive }">
          <album-details-modal 
          :title="record.basic_information.title" 
          :subtitle="record.basic_information.artists[0].name" 
          :image="record.basic_information.cover_image" 
          :genre="getGenres(record.basic_information.styles)" 
          :format="record.basic_information.formats[0].name" 
          :description="record.basic_information.formats[0].descriptions[0]" 
          :year="record.basic_information.year"
          :additionalAlbums="getAdditionalAlbums(record)"
          :resourceURL="record.basic_information.resource_url"
          @close-event="isActive.value = false" 
          :albumArray="albumArray"/>
        </template>
      </v-dialog>
    </v-row>
    </v-col>
    <v-dialog
      v-model="loading"
      width="auto"
    >
      <v-card
        max-width="400"
        color="#0b0c10"
        prepend-icon="mdi-update"
        text="The API is waking up from its slumber. Please be patient!"
        title="Service is waking up"
      >
      </v-card>
    </v-dialog>
    <v-dialog
      v-model="apiError"
      width="auto"
    >
      <v-card
        max-width="400"
        color="#0b0c10"
        prepend-icon="mdi-update"
        text="There has been an error. Please try again later."
        title="Unexpected Error"
      >
      </v-card>
    </v-dialog>
    </v-row>
  </v-container>
</template>

<script setup>
//Imports
import axios from 'axios';
import AlbumDetailsModal from "@/components/AlbumDetailsModal.vue"
import AlbumResultsItem from "@/components/AlbumResultsItem.vue"
import { ref, watch, computed, onMounted } from "vue"

//Lifecycle
onMounted(() => {
  callService()
})

//Refs
const searchParameters = ref(['Genre', 'Pressing Year', 'Artist'])
const albumArray = ref([])
const filteredAlbumArray = ref([])
const selectedSearchParameter = ref(null)
const selectedSearchKeyword = ref(null)
const selectedSearchParameterOptions = ref([])
const albumsFiltered = ref(false)
const loading = ref(true)
const apiError = ref(false)
const showSearchResultsInfo = ref(false)

//Functions
const getDropdownValues = () => {
  selectedSearchKeyword.value = null
  let property = getObjectProperty(selectedSearchParameter.value)
  selectedSearchParameterOptions.value = property
}
const getGenres = (value) => {
  let genre = null
  for (var i = 0; i < value.length; i++) {
    if (value.length == 1) {
      genre = value[i]
    } else {
      genre = [i] == 1 ? value[i] : genre + '/' + value[i]
    }
  }
  return genre
}
const removeFilter = () => {
  showSearchResultsInfo.value = false;
  selectedSearchKeyword.value = null 
  selectedSearchParameter.value = null
  albumsFiltered.value = false;
  filteredAlbumArray.value = albumArray.value
}
const getAdditionalAlbums = (value) => {
  var results = albumArray.value.filter(function (item) { return item.basic_information.title != value.basic_information.title && item.basic_information.artists[0].name === value.basic_information.artists[0].name; });
  return results
}
const getObjectProperty = (param) => {
  switch (param) {
     case 'Genre': 
        return [...new Set(albumArray.value.map(item => item.basic_information.styles[0]))];
      case 'Pressing Year': 
        let years = [...new Set(albumArray.value.map(function (item) { if (item.basic_information.year.toString().length === 4) { return item.basic_information.year}}))];
        // let years = [...new Set(albumArray.value.map(item => item.basic_information.year))];
        return years.sort(function(a, b){return a-b});
      case 'Condition': 
        return [...new Set(albumArray.value.map(item => item.basic_information.notes[0].value))];
      case 'Artist': 
        return [...new Set(albumArray.value.map(item => item.basic_information.artists[0].name))];
  }
}
const searchAlbums = () => {
  showSearchResultsInfo.value = true;
  albumsFiltered.value = true;
  switch (selectedSearchParameter.value) {
      case 'Genre': 
        filteredAlbumArray.value = [...new Set(albumArray.value.filter(function (item) { return item.basic_information.styles[0] === selectedSearchKeyword.value; }))];
        break;
      case 'Pressing Year': 
        filteredAlbumArray.value = [...new Set(albumArray.value.filter(function (item) { return item.basic_information.year === selectedSearchKeyword.value; }))];
        break;
      case 'Artist': 
       filteredAlbumArray.value = [...new Set(albumArray.value.filter(function (item) { return item.basic_information.artists[0].name === selectedSearchKeyword.value; }))];
       break;
  }
}
const callService = () => {
    axios.get('https://discogs-collection-api.onrender.com/getRecordCollection').then(response => {
    albumArray.value = response.data.releases
    filteredAlbumArray.value = response.data.releases
  }).catch(function (error) {
    apiError.value = true
  })
   loading.value = false;
}

//Computed properties
const isSearchEnabled = computed(() => selectedSearchParameter.value && selectedSearchKeyword.value)

//Watchers
watch(selectedSearchParameter, () => {
  getDropdownValues()
})
</script>

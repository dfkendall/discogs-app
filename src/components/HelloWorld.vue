<template>
  <v-container class="fill-height">
    <v-responsive
      class="fill-height mx-auto"
      max-width="1200"
    >
    <div class="text-center">
      <h1 class="my-5" style="color: #404040">Derek's Record Collection</h1>
    </div>
    <v-row>
      <v-col cols="12">
        <v-card
          class="py-4"
          color="#404040"
          rounded="lg"
        >
          <v-card-text>
            <h2 class="text-h5 font-weight-bold text-center mb-5">Explore</h2>
            <!-- <v-text-field
              :loading="loading"
              append-inner-icon="mdi-magnify"
              density="compact"
              label="Search for an album"
              variant="solo"
              hide-details
              single-line
              @click:append-inner="searchAlbums"
              bg-color="#f2f2f2"
            ></v-text-field>
            <div class="text-center my-5"><p>- OR -</p></div> -->
            <v-row>
              <v-col cols="6">
                <v-select
                  :items="searchParameters"
                  density="compact"
                  hide-details
                  bg-color="#f2f2f2"
                  item-color="#FF0033"
                  v-model="selectedSearchParameter"
                ></v-select>
              </v-col>
              <v-col cols="6">
                <v-select
                  :items="selectedSearchParameterOptions"
                  density="compact"
                  hide-details
                  bg-color="#f2f2f2"
                  :model-value="selectedSearchValue"
                  v-model="selectedSearchKeyword"
                  ></v-select>
              </v-col>
            </v-row>
            <v-row justify="center" class="mt-10">
              <v-btn color="#f2b749" style="letter-spacing: 0px;" class="text-capitalize mr-3" @click="searchAlbums" :disabled="!isSearchEnabled">Search</v-btn>
              <v-btn color="#ffffff" style="letter-spacing: 0px;" class="text-capitalize ml-3" @click="removeFilter" :disabled="!albumsFiltered">Clear</v-btn> 
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <!-- Need to lazy load these -->
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
    </v-responsive>
  </v-container>
</template>

<script setup>
//Imports
import testData from '@/assets/testdata.json'
import AlbumDetailsModal from "@/components/AlbumDetailsModal.vue"
import AlbumResultsItem from "@/components/AlbumResultsItem.vue"
import { ref, watch, computed } from "vue"

//Refs
const searchParameters = ref(['Genre', 'Release Year', 'Artist'])
const albumArray = ref(testData.data.releases)
const filteredAlbumArray = ref(testData.data.releases)
const selectedSearchParameter = ref(null)
const selectedSearchKeyword = ref(null)
const selectedSearchParameterOptions = ref([])
const albumsFiltered = ref(false)

//Functions
const getDropdownValues = () => {
  selectedSearchKeyword.value = null
  let property = getObjectProperty(selectedSearchParameter.value)
  selectedSearchParameterOptions.value = property
}
const getGenres = (value) => {
  let genre = null
  for (var i = 0; i < value.length; i++) {
    console.log(value[i], "value")
    if (value.length == 1) {
      genre = value[i]
    } else {
      genre = [i] == 1 ? value[i] : genre + '/' + value[i]
    }
  }
  return genre
}
const removeFilter = () => {
  selectedSearchKeyword.value = null 
  selectedSearchParameter.value = null
  albumsFiltered.value = false;
  filteredAlbumArray.value = testData.data.releases
}
const getAdditionalAlbums = (value) => {
  var results = albumArray.value.filter(function (item) { return item.basic_information.title != value.basic_information.title && item.basic_information.artists[0].name === value.basic_information.artists[0].name; });
  return results
}
const getObjectProperty = (param) => {
  switch (param) {
     case 'Genre': 
        return [...new Set(albumArray.value.map(item => item.basic_information.styles[0]))];
      case 'Release Year': 
        return [...new Set(albumArray.value.map(item => item.basic_information.year))];
      case 'Condition': 
        return [...new Set(albumArray.value.map(item => item.basic_information.notes[0].value))];
      case 'Artist': 
        return [...new Set(albumArray.value.map(item => item.basic_information.artists[0].name))];
  }
}
const searchAlbums = () => {
  albumsFiltered.value = true;
  switch (selectedSearchParameter.value) {
      case 'Genre': 
        filteredAlbumArray.value = [...new Set(albumArray.value.filter(function (item) { return item.basic_information.styles[0] === selectedSearchKeyword.value; }))];
        break;
      case 'Release Year': 
        filteredAlbumArray.value = [...new Set(albumArray.value.filter(function (item) { return item.basic_information.year === selectedSearchKeyword.value; }))];
        break;
      case 'Artist': 
       filteredAlbumArray.value = [...new Set(albumArray.value.filter(function (item) { return item.basic_information.artists[0].name === selectedSearchKeyword.value; }))];
       break;
  }
}

//Computed properties
const isSearchEnabled = computed(() => selectedSearchParameter.value && selectedSearchKeyword.value)

//Watchers
watch(selectedSearchParameter, () => {
  getDropdownValues()
})

console.log(testData, "This is the test data")
</script>

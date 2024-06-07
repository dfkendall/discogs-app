<template>
   <v-card class="mb-3" width="100%" color="#45a29e">
        <v-row class="pa-8">
          <v-col cols="12" lg="2" md="12" :align="desktop ? 'start' : 'center'">
            <v-img :width="desktop ? '100' : '200'" :src="image"></v-img>
          </v-col>
          <v-col cols="12" lg="4" md="12" :align="desktop ? 'start' : 'center'">
            <v-card-text class="text-h5 font-weight-bold">{{title}}<span>{{` (${year})`}}</span></v-card-text>
            <v-card-text class="mt-n6 text-body-1">{{subtitle}}</v-card-text>
          </v-col>
          <v-col :class="!desktop ? 'mt-n10' : ''" cols="12" lg="3" md="12" :align="desktop ? 'start' : 'center'">
            <v-card-text>Genre: {{genre}}</v-card-text>
          </v-col>
          <v-col :class="!desktop ? 'mt-n10' : ''" cols="12" lg="3" md="12" :align="desktop ? 'start' : 'center'">
            <v-card-text>{{`${format} | ${description}`}}</v-card-text>
          </v-col>
        </v-row>
      </v-card>
</template>

<script setup>
import { ref, watch } from "vue";
import { useDisplay } from "vuetify"
const props = defineProps({ title: String, subtitle: String, image: String, genre: String, format: String, description: String, year: String })
const { lgAndUp } = useDisplay()
const desktop = ref(true)

watch(
  lgAndUp,
  (val) => {
    desktop.value = val ? true : false;
  },
  {
    immediate: true,
  }
);
</script>
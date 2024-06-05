<template>
   <v-card class="mb-3" width="100%">
        <v-row class="pa-8">
          <v-col cols="2">
            <v-img width="100" :src="image"></v-img>
          </v-col>
          <v-col cols="4">
            <v-card-text class="text-h5 font-weight-bold">{{title}}<span>{{` (${year})`}}</span></v-card-text>
            <v-card-text class="mt-n6 text-body-1">{{subtitle}}</v-card-text>
          </v-col>
          <v-col cols="3">
            <v-card-text>Genre: {{genre}}</v-card-text>
          </v-col>
          <v-col cols="3">
            <v-card-text>{{`${format} | ${description}`}}</v-card-text>
          </v-col>
        </v-row>
      </v-card>
</template>

<script setup>
const props = defineProps({ title: String, subtitle: String, image: String, genre: String, format: String, description: String, year: String })
</script>
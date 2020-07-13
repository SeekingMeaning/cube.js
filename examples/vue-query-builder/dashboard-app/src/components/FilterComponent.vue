<template>
  <v-row>
    <v-row>
      <template>
        <v-dialog v-model="dialog" persistent max-width="600px">
          <template v-slot:activator="{ on, attrs }">
            <v-col cols="2">
              <v-btn
                color="primary"
                dark
                v-bind="attrs"
                v-on="on"
              >
                Add filter
              </v-btn>
            </v-col>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">Adding filter</span>
            </v-card-title>

            <v-container>
              <v-row>

              </v-row>
            </v-container>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="dialog = false">Close</v-btn>
              <v-btn color="blue darken-1" text @click="dialog = false">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </template>
    </v-row>
    <v-row v-for="(filter, index) in filters" :key="index">
      <v-alert
        dense
        color="info"
      >
        {{index}} : {{filter}}
      </v-alert>
    </v-row>
  </v-row>
</template>

<script>
  export default {
    name: 'FilterComponent',
    props: ['filters'],
    data: () => ({
      dialog: false,
    }),
  }
</script>


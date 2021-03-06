<template>
  <v-container>
    <input
      ref="file"
      type="file"
      class="d-none"
      @change="uploadPhoto"
    >
    <div
      class="detail-header"
    >
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-card
            width="52"
            height="52"
            outlined
            class="detail-avartar"
            v-on="on"
            @click="$refs.file.click()"
          >
            <v-img
              v-if="coverPhoto"
              :src="coverPhoto"
              height="100%"
              width="100%"
            />
            <v-icon
              v-else
              class="mx-auto"
              size="48"
            >
              mdi-ferry
            </v-icon>
          </v-card>
        </template>
        <span>Upload/Change Image</span>
      </v-tooltip>
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <h3
            class="display-2 title-ellipsis"
            v-on="on"
          >
            {{ editedItem.name }}
          </h3>
        </template>
        <span>{{ editedItem.name }}</span>
      </v-tooltip>
      <div class="detail-status">
        <v-badge
          bottom
          bordered
          overlap
          color="orange"
          :value="smff"
        >
          <template v-slot:badge>
            <v-icon dark>
              mdi-star
            </v-icon>
          </template>
          <span>
            <v-icon
              v-if="editedItem.active"
              color="success"
              size="30"
            >
              mdi-shield-check
            </v-icon>
            <v-icon
              v-else
              color="error"
              size="30"
            >
              mdi-shield-off
            </v-icon>
          </span>
        </v-badge>
        <v-switch
          v-model="editedItem.active"
          label="Coverage"
          @click.stop="toggleStatus"
        />
        <v-switch
          v-model="smff"
          label="Capabilities"
          @click.stop="toggleSMFF"
        />
      </div>
    </div>
    <base-material-tabs
      v-model="activeTab"
      background-color="transparent"
      centered
      color="secondary"
      icons-and-text
      show-arrows
    >
      <v-tab
        v-for="(tab, i) in tabs"
        :key="i"
        :to="tab.to"
      >
        {{ tab.title }}
        <v-icon v-text="tab.icon" />
      </v-tab>
    </base-material-tabs>
    <router-view />
    <base-material-snackbar
      v-model="snackbar"
      :color="snackbarColor"
      bottom
      right
      :type="null"
    >
      {{ snackbarText }}
    </base-material-snackbar>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import { snackBar } from '@/mixins/snackBar'

  export default {
    mixins: [snackBar],
    data: () => ({
      coverPhoto: null,
      activeTab: 0,
      editedItem: {},
      msgBox: false,
      smff: null,
    }),
    computed: {
      tabs () {
        return [
          {
            title: 'General',
            icon: 'mdi-information',
            to: '/vessels/' + this.$route.params.id + '/general',
          },
          {
            title: 'Construction',
            icon: 'mdi-ruler-square',
            to: '/vessels/' + this.$route.params.id + '/construction',
          },
          {
            title: 'Fire Plans',
            icon: 'mdi-fire-extinguisher',
            to: '/vessels/' + this.$route.params.id + '/fire-plans',
          },
          {
            title: 'Drawings',
            icon: 'mdi-file-cad',
            to: '/vessels/' + this.$route.params.id + '/drawings',
          },
          {
            title: 'RACs',
            icon: 'mdi-phone-in-talk',
            to: '/vessels/' + this.$route.params.id + '/racs',
          },
          {
            title: 'Capabilities',
            icon: 'mdi-pier-crane',
            to: '/vessels/' + this.$route.params.id + '/smff',
          },
          {
            title: 'AIS Data',
            icon: 'mdi-map-marker',
            to: '/vessels/' + this.$route.params.id + '/ais',
          },
          {
            title: 'Notes',
            icon: 'mdi-note',
            to: '/vessels/' + this.$route.params.id + '/notes',
          },
          {
            title: 'Models',
            icon: 'mdi-laptop',
            to: '/vessels/' + this.$route.params.id + '/models',
          },
        ]
      },
    },
    watch: {
      $route (to, from) {
        // react to route changes...
        if (to.params.id !== from.params.id) {
          this.getDataFromApi()
          this.getCover()
        }
      },
    },
    mounted () {
      this.getDataFromApi()
      this.getCover()
    },
    methods: {
      getCover () {
        axios.get('vessels/' + this.$route.params.id + '/getPhoto')
          .then(res => {
            if (res.data) {
              this.coverPhoto = res.data + '?' + Math.random().toString(36).substring(7)
            }
          })
      },
      uploadPhoto (event) {
        const formData = new FormData()
        formData.append('file', event.target.files[0])
        axios.post(
          'vessels/' + this.$route.params.id + '/setPhoto',
          formData,
          {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          },
        ).then(res => {
          this.getCover()
        })
      },
      getDataFromApi () {
        axios.get('vessels/' + this.$route.params.id)
          .then(res => {
            this.editedItem = res.data.data[0]
          })
        axios.get('vessels/' + this.$route.params.id + '/smff')
          .then(res => {
            this.smff = res.data.smff
          })
      },
      toggleStatus () {
        axios.post('vessels/' + this.$route.params.id + '/toggleStatus')
          .then(res => {
            this.showSnackBar(res.data.message, 'success')
            this.getDataFromApi()
          })
      },
      toggleSMFF () {
        if (this.smff) {
          this.loading = true
          axios.delete('vessels/' + this.$route.params.id + '/smff')
            .then(res => {
              this.showSnackBar(res.data.message, 'success')
              this.getDataFromApi()
            })
        } else {
          axios.post('vessels/' + this.$route.params.id + '/smff/create')
            .then(res => {
              this.showSnackBar(res.data.message, 'success')
              this.getDataFromApi()
            })
        }
      },
    },
  }
</script>

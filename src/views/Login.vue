<template>
<v-container fluid row fill-height grid-list-lg class="pt-0">
  <v-layout align-center justify-center wrap>
    <v-flex xs12>
      <v-alert :value="loggedIn" color="success">
        You are currently logged in. <v-btn depressed flat small @click.stop="logOut">Click here to log out.</v-btn>
      </v-alert>
    </v-flex>

    <v-flex xs12 >
      <v-card flat>
        <v-toolbar color="primary" dark flat>
          <v-toolbar-title>Database login</v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
        <v-card-text>
          <v-form>
            <v-select v-model="database.protocol" :items="['http','https']" label="Protocol"></v-select>
            <v-text-field v-model="database.host" label="Database host" placeholder="couchdb.example.com" type="text"></v-text-field>
            <v-text-field v-model="database.port" label="Database port" placeholder="6984" type="text"></v-text-field>
            <v-text-field v-model="database.database" label="Database name" placeholder="OpenInventory" type="text"></v-text-field>
            <v-switch v-model="database.remember" label="Remember database details?" hide-details></v-switch>

            <br />
            <br />

            <v-text-field v-model="user.username" label="Username" type="text"></v-text-field>
            <v-text-field v-model="user.password" label="Password" type="password"></v-text-field>
            <v-switch v-model="user.remember" label="Remember username and password?" :disabled="!database.remember" hide-details></v-switch>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn depressed small @click.stop="skip" color="">Run in local mode</v-btn>
          <v-spacer></v-spacer>
          <v-btn depressed @click.stop="login" color="primary">Login</v-btn>
        </v-card-actions>
      </v-card>
    </v-flex>
  </v-layout>

  <v-dialog v-model="progressModal" width="500" :persistent="true">
    <v-card>
      <v-card-text>
        Initial replication in progress...
        <v-progress-linear indeterminate color="white"></v-progress-linear>
      </v-card-text>
    </v-card>
  </v-dialog>
</v-container>
</template>

<script>
import {
  ConfigService
} from '../services/Config.js'
import DatabaseService from '../services/Database.js'

export default {
  data: () => ({
    database: ConfigService.getLocalConfig('database') || {},
    user: ConfigService.getLocalConfig('user') || {},
    loggedIn: ConfigService.getLocalConfig('user') && ConfigService.getLocalConfig('database'),
    progressModal: false
  }),
  watch: {
    'database.remember' (to, from) { // Make sure we stop skpping when the modal closes
      if (!to) {
        this.user.remember = false
      }
    }
  },
  methods: {
    login: function () {
      ConfigService.setLocalConfig(this.database.remember, 'database', this.database)
      ConfigService.setLocalConfig(this.user.remember, 'user', this.user)
      ConfigService.setLocalConfig(true, 'localMode', false) // Disable local mode if we ever log in

      this.progressModal = true
      DatabaseService(true, () => { // Force DB service recreation.
        this.progressModal = false
        this.submit()
      })
    },
    skip: function () {
      ConfigService.setLocalConfig(true, 'localMode', true)
      this.$toasted.show('Now in local mode')
      this.submit()
    },
    submit: function () {
      if (this.$router.currentRoute.path === '/login/' || this.$router.currentRoute.path === '/login') {
        this.$router.push('/')
      }
    },
    /**
     * Log out of local database. Destroy local data. Remove configuration.
     */
    logOut: function () {
      ConfigService.setLocalConfig(true, 'database', null)
      ConfigService.setLocalConfig(true, 'user', null)
      ConfigService.setLocalConfig(true, 'localMode', null)
      DatabaseService(true).getDatabase().destroy().then(() => {
        DatabaseService(true)
        this.$router.push('/login/')
      }).catch((error) => {
        this.$toasted.error(`Error ${error}`)
      })
    }
  }
}
</script>

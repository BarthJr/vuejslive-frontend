<template>
  <div class="instructors">
    <v-dialog v-model="dialog" persistent max-width="400px">
      <v-card>
        <v-card-title>
          {{ formTitle }}
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col class="align-center justify-space-between" cols="12">
                <v-row align="center">
                  <v-avatar size="40px" class="mr-4">
                    <img src="//ssl.gstatic.com/s2/oz/images/sge/grey_silhouette.png" alt="">
                  </v-avatar>
                  <v-text-field class="mr-3" v-model="editedItem.name" placeholder="Name"></v-text-field>
                </v-row>
              </v-col>
              <v-col cols="12">
                <v-text-field prepend-icon="mail" v-model="editedItem.email" placeholder="Email"></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-btn color="error" :disabled="awaiting" @click="close">Cancel</v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary" :loading="awaiting" @click="save">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-data-table
      :headers="headers"
      :items="instructors"
      class="elevation-1"

    >
      <template v-slot:item.action="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        >
          edit
        </v-icon>
        <v-icon
          small
          @click="deleteItem(item)"
        >
          delete
        </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>
    </v-data-table>
    <v-btn
      bottom
      color="primary"
      dark
      fab
      fixed
      right
      @click="dialog = !dialog"
    >
      <v-icon>add</v-icon>
    </v-btn>
  </div>
</template>


<script>
import axios from '../plugins/axios'
const url = 'api/instructors/'
export default {
  data: () => ({
    awaiting: false,
    dialog: false,
    headers: [
      { text: 'Id', align: 'left', value: 'id' },
      { text: 'Name', align: 'left', value: 'name' },
      { text: 'Email', value: 'email' },
      { text: 'Actions', value: 'action', sortable: false },
    ],
    instructors: [],
    editedIndex: -1,
    editedItem: {
      name: '',
      email: '',
    },
    defaultItem: {
      name: '',
      email: '',
    },
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
  },

  created () {
    this.initialize()
  },

  methods: {
    async load () {
      this.instructors = await axios.get(url).then(res => res.data)
    },
    initialize () {
      this.instructors = []
    },

    editItem (item) {
      this.editedIndex = this.instructors.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    async deleteItem (item) {
      const index = this.instructors.indexOf(item)
      if (confirm('Are you sure you want to delete this item?') && this.instructors.splice(index, 1)) {
        const { id } = item
        try {
          await axios.delete(`${url}${id}/`)
          this.load()
        } catch (e) {
          console.error(e)
        }

      }
    },

    close () {
      this.dialog = false
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      }, 300)
    },

    async save () {
      this.awaiting = true
      if (this.editedIndex > -1) {
        Object.assign(this.instructors[this.editedIndex], this.editedItem)
        const { id } = this.editedItem
        try {
          await axios.patch(`${url}${id}/`, this.editedItem)
          this.close()
          this.load()
        } catch (e) {
          console.error(e)
        } finally {
          this.awaiting = false
        }
      } else {
        try {
          await axios.post(url, this.editedItem)
          this.instructors.push(this.editedItem)
          this.close()
          this.load()
        } catch (e) {
          console.error(e)
        } finally {
          this.awaiting = false
        }
      }
      this.close()
    },
  },
  mounted () {
    this.load()
  },
}
</script>

<style scoped>
</style>

<template>
  <v-data-table
      dense
      :headers="headers"
      :items="got_workhours"
      disable-sort
      locale="ru-Ru"
      mobile-breakpoint="300"
      class="elevation-1"
      :search="search"
  >

    <template v-slot:top>
      <v-toolbar            flat        >
        <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details
            dense
            outlined
            clearable
            background-color="#555555"
        ></v-text-field>
        <v-spacer></v-spacer>
        <v-dialog
            v-model="dialog"
            max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
                color="primary"
                dark
                class="mb-2"
                v-bind="attrs"
                v-on="on"
                x-small
                plain
            >
              <v-icon>mdi-account-plus</v-icon>
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="overline">{{ formTitle }}</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                      cols="12"
                      sm="6"
                      md="4"
                  >
                    <v-select
                        dense
                        label="ФИО"
                        :items="available_users"
                        item-text="fio"
                        item-value="id"
                    ></v-select>
                  </v-col>
                  <v-col
                      cols="12"
                      sm="6"
                      md="4"
                  >
                    <v-text-field
                        dense
                        v-model="editedItem.hours"
                        label="Часы"
                        type="number"
                    ></v-text-field>
                  </v-col>
                  <v-col
                      cols="12"
                      sm="6"
                      md="4"
                  >
                    <v-text-field
                        dense
                        v-model="editedItem.change"
                        label="Замещение"
                        type="number"
                    ></v-text-field>
                  </v-col>
                  <v-col
                      cols="12"
                      sm="6"
                      md="4"
                  >
                    <v-text-field
                        dense
                        v-model="editedItem.shifts"
                        label="Смены"
                        type="number"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                  color="blue darken-1"
                  text
                  @click="close"
              >
                Закрыть
              </v-btn>
              <v-btn
                  color="blue darken-1"
                  text
                  @click="save"
              >
                Сохранить
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-center overline">Вы уверены что хотите удалить сотрудника?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Нет</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">Да</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <!--<v-icon
          small
          @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>-->
    </template>
    <template v-slot:no-data>
      <v-btn
          color="primary"
          @click="initialize"
      >
        Обновить
      </v-btn>
    </template>
  </v-data-table>
</template>

<script>
export default {
  data: () => ({
    available_users: null,
    search: '',
    updateComplete: null,
    dialog: false,
    dialogDelete: false,
    got_workhours: [],
    headers: [
      { text: 'ФИО', align: 'start', sortable: true, value: 'fio' },
      { text: 'Ч', align: 'end', sortable: false, value: 'hours' },
      { text: 'З', align: 'end', sortable: false, value: 'change' },
      { text: 'См', align: 'end', sortable: false, value: 'shifts' },
      { text: '', align: 'end', value: 'actions', sortable: false },
    ],
    editedIndex: -1,
    editedItem: {
      name: '',
      surname: '',
      hours: '',
      change: '',
      shifts: ''
    },
    defaultItem: {
      name: '',
      surname: '',
      hours: '',
      change: '',
      shifts: ''
    },
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'Запись часов' : 'Изменение данных'
    },
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    },
  },

  created () {
    this.initialize()
  },

  methods: {
    async getavailableusers(){
      this.available_users = await this.$api.post('getavailableusers',{})
      console.log(this.available_users)
    },
    async get_workhours() {
      this.got_workhours = await this.$api.post('getworkhours',{"month" : 4, "year": 2021})
      console.log(this.got_workhours)
    },
    async updateWorkhours(){
      console.log(this.editedItem)
      //const updateComplete = await this.$api.post('updateworkhours', this.editedItem)
      //console.log(updateComplete)
    },
    initialize () {
      this.get_workhours()
      this.getavailableusers()
    },

    editItem (item) {
      this.editedIndex = this.got_workhours.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem (item) {
      this.editedIndex = this.got_workhours.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    deleteItemConfirm () {
      this.got_workhours.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save () {
      if (this.editedIndex > -1) {
        Object.assign(this.got_workhours[this.editedIndex], this.editedItem)
        //this.editedItem['sender'] = this.$storage.state.user.id
        this.updateWorkhours()
        console.log(JSON.stringify(this.editedItem))
      } else {
        this.got_workhours.push(this.editedItem)

        console.log(this.editedItem)
      }
      this.close()
    },
  },
name: "Workhours"
}
</script>

<style scoped>

</style>
<template>
  <v-container class="col-xl-4 offset-xl-4">
    <v-text-field
        v-if="this.workingUsers.length"
                  class="mb-5"
                  v-model="search"

                  label="Поиск"
                  single-line
                  hide-details
                  dense
                  clearable
    ></v-text-field>
    <v-data-table
        :headers="headers"
        :items="workingUsers"
        :items-per-page="15"
        :search="search"
        hide-default-header
        class="elevation-1"
        dense
        mobile-breakpoint="300"
        v-if="this.workingUsers.length"
        @click:row="row_click"
        :footer-props="{
          itemsPerPageAllText: '',
          itemsPerPageText: '',
          itemsPerPageOptions: [15],
          showCurrentPage: true
        }"
    >
    </v-data-table>
    <div class="text-center">
      <v-dialog
        v-model="editZpDialog"
        width="400"
        overlay-opacity="0.95"
        overlay-color="#111111"
      >
        <v-card>
          <v-card-title>
            Надбавки
          </v-card-title>
          <v-form>
            <v-container>
              <v-row
                  justify="space-around"
                  align="center"
              >
                <v-col
                    cols="6"
                    xs="6"
                    sm="6"
                    md="6"
                >
                  <v-text-field
                      clearable
                      v-model="user_data.dop"
                      label="Дополнительно"
                  ></v-text-field>
                </v-col>
                <v-col
                    cols="6"
                    xs="6"
                    sm="6"
                    md="6"
                >
                  <v-text-field
                      clearable
                      v-model="user_data.correction"
                      label="Коррекция"
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-form>
          <v-divider></v-divider>
          <v-card-actions>
            <v-btn color="success" text @click="updateExtra(user_data.id, user_data.dop, user_data.correction, user_data.period_month, user_data.period_year)">
              Записать
            </v-btn>
            <v-spacer></v-spacer>
              <v-btn
                  color="warning"
                  text
                  @click="editZpDialog = false"
              >
                Отменить
              </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog
          v-if="this.user_data && this.user_data.surname"
          v-model="zp_dialog"
          width="500"
          overlay-opacity="0.95"
          overlay-color="#111111"
      >
        <v-card>
          <v-card-title
              v-if="this.user_data">
            {{ user_data.surname + " " + user_data.name}}
          </v-card-title>
          <v-card-subtitle>{{ user_data.role_id_ru}}, {{user_data.month_ru_small}}, {{user_data.period_year}}</v-card-subtitle>
          <v-card-title
              v-if="!this.user_data">
            Данных нет
          </v-card-title>
          <v-card-text
              v-if="this.user_data">
            <v-simple-table style="font-family: monospace; font-size: 10px" class="mb-5"  dense>
              <tbody>
              <tr v-if="user_data.hours">
                <td style="width: 60%">Норма часов</td>
                <td style="text-align: right">{{ user_data.hours}} / {{ user_data.month_standard}}</td>
              </tr>
              <tr v-if="user_data.change">
                <td style="width: 60%">Замещения</td>
                <td style="text-align: right">{{ user_data.change}}</td>
              </tr>
              <tr v-if="user_data.shifts">
                <td style="width: 60%">Смены</td>
                <td style="text-align: right">{{ user_data.shifts}}</td>
              </tr>
              <tr v-if="user_data.salary">
                <td>Оклад</td>
                <td style="text-align: right">{{ user_data.salary}}</td>
              </tr>

              </tbody>
            </v-simple-table>
            <v-simple-table style="font-family: monospace; font-size: 10px" class="my-5"  dense>
              <tbody>

              <tr v-if="user_data.ktu">
                <td style="width: 60%">КТУ</td>
                <td style="text-align: right">{{ user_data.ktu}}</td>
              </tr>
              <tr v-if="user_data.nachisleno">
                <td>Начислено</td>
                <td style="text-align: right">{{ user_data.nachisleno}}</td>
              </tr>
              <tr v-if="user_data.accrualbonus">
                <td>Бонус</td>
                <td style="text-align: right">{{ user_data.accrualbonus}}</td>
              </tr>
              <tr v-if="user_data.shifts">
                <td>Смены</td>
                <td style="text-align: right">{{ user_data.shifts}}</td>
              </tr>
              <tr v-if="user_data.change">
                <td>Замещения</td>
                <td style="text-align: right">{{ user_data.change}}</td>
              </tr>
              <tr v-if="user_data.bonus">
                <td>Поощрения</td>
                <td style="text-align: right">{{ user_data.bonus}}</td>
              </tr>
              <tr v-if="user_data.correction">
                <td>Коррекция</td>
                <td style="text-align: right">{{ user_data.correction }}</td>
              </tr>
              <tr v-if="user_data.dop">
                <td>Доп</td>
                <td style="text-align: right">
                  {{ user_data.dop }}
                </td>
              </tr>
              <tr>
                <td style="width: 60%">ИТОГО</td>
                <td style="text-align: right">{{+user_data.nachisleno + user_data.change + +user_data.correction + +user_data.dop + user_data.accrualbonus + user_data.bonus}}</td>
              </tr>
              </tbody>
            </v-simple-table>
            <!-------------->
            <v-simple-table style="font-family: monospace; font-size: 10px" class="mt-5"  dense>
              <tbody>

              <tr v-if="user_data.fine">
                <td>Штрафы</td>
                <td style="text-align: right">{{ user_data.fine}}</td>
              </tr>

              <tr v-if="user_data.ndfl">
                <td>НДФЛ</td>
                <td style="text-align: right">{{ user_data.ndfl}}</td>
              </tr>
              <tr v-if="user_data.aliments">
                <td>Алименты</td>
                <td style="text-align: right">{{ user_data.aliments}}</td>
              </tr>
              <tr v-if="user_data.bank">
                <td>На карту</td>
                <td style="text-align: right">{{ user_data.bank }}</td>
              </tr>
              <tr>
                <td style="width: 60%">Налоги итого</td>
                <td style="text-align: right">{{user_data.taxes}}</td>
              </tr>
              <tr v-if="user_data.credit">
                <td>Авансы</td>
                <td style="text-align: right">{{ user_data.credit}}</td>
              </tr>
              <tr>
                <td>ИТОГО</td>
                <td style="text-align: right">{{user_data.fine + user_data.credit + user_data.aliments + user_data.ndfl + user_data.bank}}</td>
              </tr>

              </tbody>
            </v-simple-table>
          </v-card-text>


          <v-divider></v-divider>

          <v-card-actions>


            <v-btn color="success" text @click="editZpDialog = true">
              Редактировать
            </v-btn>
            <v-spacer></v-spacer>
            <v-btn
                color="primary"
                text
                @click="zp_dialog = false
                user_data = []"
            >
              Закрыть
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
    <!------->
  </v-container>
</template>

<script>
export default {
  name: "Main.vue",
  data() {
    return {
      zp_dialog: false,
      editZpDialog: false,
      user_data: [],
      workingUsers: [],
      search: '',
      headers: [
        { text: 'Сотрудник', value: 'fio' },
        { text: 'С', value: 'ktu_sum' },
        //{ text: 'Ш', value: 'ktu_amount' },
        //{ text: 'Стр', value: 'ktu_lines' },
        //{ text: 'Ч', value: 'ktu_documents' }
      ]
    }
  },
  methods: {
    async updateExtra(userid, dop, correction, month, year){
      //user_data.id, user_data.dop, user_data.period_month, user_data.period_year
      console.log("UPDATING ITEMS: USER ID:" + userid + ", DOP: " + dop + ", CORRECTION: " + correction)
      let response = await this.$api.post('updatecorrection', {'selected_user': userid, 'dop': dop, 'correction': correction, 'month': month, 'year': year})
      console.log(response.callback)
      this.editZpDialog = false
    }
    ,
    async get_individual_data(selected_user){
      this.user_data = await this.$api.post('individualzp',{
        "month": this.$storage.state.calMonth,
        "year" : this.$storage.state.calYear,
        "selected_user" : selected_user
      })
      console.log(this.user_data)
    },
    row_click(item){
      this.zp_dialog = true
      this.get_individual_data(item.id)
      //alert(item.fio + " " + item.birthday)
    },
    async get_working_users() {
      console.log('get_ktu func from Ktu')
      this.workingUsers = await this.$api.post('getworkingusers',{"report_date" : "2021-05-01", "year": this.$storage.state.calYear, "month": this.$storage.state.calMonth})
      console.log(this.workingUsers)
    }
  },
  mounted() {
    this.get_working_users()
    },
  beforeMount(){

  },
}

</script>

<style scoped>

</style>
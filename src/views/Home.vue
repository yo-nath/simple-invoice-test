<template>
  <v-container fluid>
    <v-layout wrap>
      <v-flex align-center xs12 class="mb-2" style="display: flex;">
        <v-btn color="info" v-on:click="openModal">New Invoice</v-btn>
        <date-picker
          v-model="dateModel"
          :confirm="true"
          range
          lang="en"
          :shortcuts="false"
          placeholder="Filter by date"
          @change="updateRange"
          @clear="updateRange"
         />
      </v-flex>
      <v-flex xs12>
        <v-data-table
          :headers="headers"
          :items="filteredData"
          :pagination.sync="paginationSetting"
          :rows-per-page-items="[10, 25, 50, 100]"
        >
          <template v-slot:items="props">
            <tr v-bind:class="{'blocked-row': props.item.blocked}">
              <td>{{ props.item.invoiceName }}</td>
              <td>{{ props.item.date | moment }}</td>
              <td>${{ props.item.amount }}</td>
              <td class="layout px-0">
                <v-btn color="warning" @click="editInvoice(props.item)">Edit</v-btn>
                <v-btn color="error" @click="deleteInvoice(props.item)">Delete</v-btn>
              </td>
            </tr>
          </template>
          <template v-slot:footer>
            <td colspan="2">
              <strong>Total</strong>
            </td>
            <td colspan="2">
              <strong>${{ totalPrice }}</strong>
            </td>
          </template>
        </v-data-table>
      </v-flex>
    </v-layout>
    <Modal :title="modalTitle" :show="showModal">
      <template v-slot:fields>
        <v-layout wrap>
          <v-flex xs12 sm12 md12>
            <v-text-field v-model="fields.invoiceName" label="Invoice Name"></v-text-field>
          </v-flex>
          <v-flex xs12 sm12 md12>
            <v-text-field v-model="fields.amount" label="Amount"></v-text-field>
          </v-flex>
          <v-flex xs12 sm12 md12>
            <v-menu
              ref="menu"
              v-model="menu"
              :close-on-content-click="false"
              :return-value.sync="fields.date"
              transition="scale-transition"
              full-width
              min-width="290px"
            >
              <template v-slot:activator="{ on }">
                <v-text-field
                  v-model="formattedDate"
                  label="Date"
                  readonly
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker v-model="fields.date" no-title scrollable>
                <v-spacer></v-spacer>
                <v-btn flat color="primary" @click="menu = false">Cancel</v-btn>
                <v-btn flat color="primary" @click="$refs.menu.save(fields.date)">OK</v-btn>
              </v-date-picker>
            </v-menu>
          </v-flex>
        </v-layout>
      </template>
      <template v-slot:actions>
        <v-btn color="blue darken-1" flat @click="closeModal">Cancel</v-btn>
        <v-btn color="blue darken-1" flat @click="saveInvoice">Save</v-btn>
      </template>
    </Modal>
    <ConfirmDialog ref="confirm" />
    <v-snackbar
      v-model="snackbar.show"
      :color="snackbar.color"
      :timeout="snackbar.timeout"
      :bottom="true"
    >
      {{ snackbar.text }}
      <v-btn
        dark
        flat
        @click="snackbar.show = false"
      >
        Close
      </v-btn>
    </v-snackbar>
  </v-container>
</template>

<script>
  import Modal from '../components/Modal';
  import ConfirmDialog from '../components/Confirm';
  import faker from 'faker';
  import DatePicker from 'vue2-datepicker'
  import moment from 'moment';
  export default {
    name: 'App',
    components: {
      Modal,
      ConfirmDialog,
      DatePicker
    },
    data () {
      return {
        headers: [
          { text: 'Invoice Name', value: 'invoiceName'},
          { text: 'Date', value: 'date' },
          { text: 'Amount', value: 'amount'},
          { text: 'Actions', value: 'actions'}
        ],
        paginationSetting: {
          rowsPerPage: 10,
          sortBy: ''
        },
        data: [],
        showModal: false,
        fields: {
          invoiceName: '',
          date: '',
          amount: ''
        },
        menu: '',
        editedIndex: -1,
        snackbar: {
          show: false,
          color: '',
          timeout: 5000,
          text: ''
        },
        dateRange: {start: undefined, end: undefined},
        dateModel: []
      }
    },

    methods: {
      openModal() {
        this.showModal = true;
      },

      closeModal() {
        this.showModal = false;
        setTimeout(() => {
          this.fields = {
            invoiceName: '',
            date: '',
            amount: ''
          };
          this.editedIndex = -1;
        }, 300);
      },

      saveInvoice() {
        this.$store.dispatch('loading', true);
        if (this.editedIndex === -1) {
          setTimeout(() => {
            this.data.unshift(this.fields);
            this.$store.dispatch('loading', false);
            this.closeModal();
            this.showSnackbar('success', 'Invoice created successfully');
          }, 2500);   
        } else {
          setTimeout(() => {
            Object.assign(this.data[this.editedIndex], this.fields);
            this.$store.dispatch('loading', false);
            this.closeModal();
            this.showSnackbar('success', 'Invoice edited successfully');
          }, 2500);   
        }
      },

      editInvoice(item) {
        this.editedIndex = this.data.indexOf(item);
        this.fields = Object.assign({}, item);
        this.fields.date = moment(this.fields.date).format('YYYY-MM-DD');
        this.openModal();
      },

      deleteInvoice(item) {
        const index = this.data.indexOf(item);
        this.$refs.confirm.open('Delete Invoice', 'Are you sure?', { color: 'red' }).then((confirm) => {
          if (confirm) {
            this.data.splice(index, 1);
            this.showSnackbar('success', 'Invoice deleted successfully');
          }
        });
      },

      showSnackbar(color, text) {
        this.snackbar.color = color;
        this.snackbar.text = text;
        this.snackbar.show = true;
      },

      updateRange(date) {
        if (date == null) {
          this.dateRange = {start: undefined, end: undefined};
        } else {
          this.dateRange = {start: moment(date[0]).format('MM/DD/YYYY'), end: moment(date[1]).format('MM/DD/YYYY')};
        }
      }
    },

    mounted() {
      this.$store.dispatch('loading', true);
      setTimeout(() => {
        let data = [];
        for (let x = 0; x < 10; x++) {
          data[x] = {
            invoiceName: faker.name.firstName() + "'s Invoice",
            amount: faker.random.number(),
            date: faker.date.recent()
          }
        }
        this.data = data;
        this.$store.dispatch('loading', false);
      }, 3000);
    },

    computed: {
      totalPrice() {
        return this.filteredData.reduce((acc, obj) => {
          return Number(acc) + Number(obj['amount']);
        }, 0);
      },

      modalTitle() {
        return this.editedIndex === -1 ? 'New Invoice' : 'Edit Invoice'
      },

      formattedDate() {
        return this.fields.date ? moment(this.fields.date).format('MM/DD/YYYY') : '';
      },

      filteredData() {
        if (this.dateRange.start != null && this.dateRange.end != null) {
          return this.data.filter((obj) => {
            return moment(obj.date).isBetween(this.dateRange.start, this.dateRange.end);
          });
        } else {
          return this.data;
        }
      }
    },

    filters: {
      moment(date) {
        return moment(date).format('MM/DD/YYYY');
      }
    }

  }
</script>
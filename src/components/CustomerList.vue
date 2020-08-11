
<template>
  <main>
    <br/>
    <v-card
    class="mx-auto"
    max-width="400" style="background-color: burlywood;"
  >
  
    <v-card-text class="text--primary">
      <div>
            <v-container fluid ma-0 pa-0 fill-height>
                <v-layout column align-center>
                  <blockquote>
                    &#8220;Welcome {{validUserName}}!&#8221;
                    <footer>
                      <small>
                        <em>&mdash;Eagle Financial Services, your Midwest Financial Services Partner.</em>
                      </small>
                    </footer>
                  </blockquote>

                </v-layout>
          </v-container>
      </div>

      
    </v-card-text>
  </v-card>

       <v-layout column align-center>
       <v-flex xs6 sm8 md7>
       <v-alert v-if="showMsg === 'new'"
                dismissible
        :value="true"
        type="success"
      >
        New customer has been added.
      </v-alert>
      <v-alert v-if="showMsg === 'update'" dismissible
        :value="true"
        type="success"
      >
        Customer information has been updated.
      </v-alert>
         <v-alert v-if="showMsg === 'deleted'" dismissible
        :value="true"
        type="success"
      >
        Selected Customer has been deleted.
      </v-alert>

       </v-flex>
       </v-layout>
      <br/>
             <v-card>
               <v-card-title style="display: flex; justify-content: center;background: darksalmon;"><h3 style="font-size: x-large;">Customer List</h3></v-card-title>
                  <v-card-text class="text--primary">
                    <div>
                          <v-container fluid grid-list-md fill-height>
                              <v-layout column align-center>
                                <v-flex md6>
                                  <v-data-table
                                  :headers="headers"
                                  :items="customers"
                                  hide-actions
                                  class=" overflow"
                                  fixed
                                  style="max-height: 300px; display:flex; flex-direction:column;">
                                  xs6 sm8 md7
                                    <template slot="items" slot-scope="props"  >
                                      <td>{{ props.item.cust_number }}</td>
                                      <td>{{ props.item.name }}</td>
                                      <td nowrap="true">{{ props.item.address }}</td>
                                      <td nowrap="true">{{ props.item.city }}</td>
                                      <td nowrap="true">{{ props.item.state }}</td>
                                      <td nowrap="true">{{ props.item.zipcode }}</td>
                                      <td nowrap="true">{{ props.item.email }}</td>
                                      <td nowrap="true">{{ props.item.cell_phone }}</td>
                                      <td nowrap="true">
                                        <v-icon @click="updateCustomer(props.item)">edit</v-icon>
                                      </td>
                                      <td nowrap="true">
                                        <v-icon @click="deleteCustomer(props.item)">delete</v-icon>
                                      </td>
                            
                                    </template>

                                  </v-data-table>
                                </v-flex>

                              </v-layout>
                          </v-container>
                    </div>
                  </v-card-text>
              </v-card>
    <div style="display: flex;
    justify-content: center;">
      <v-btn class="blue white--text" style="" @click="addNewCustomer">Add Customer</v-btn>
    </div>
  </main>
</template>


<script>

  import router from '../router';
  import {APIService} from '../http/APIService';
  const apiService = new APIService();

  export default {
    name: "CustomerList",
    data: () => ({
      customers: [],
      validUserName: "Guest",
      customerSize: 0,
      showMsg: '',
      headers: [
        {
          text: 'Customer Number',
          align: 'left',
          sortable: false,

        },
        {text: 'Name', sortable: false, align: 'left',},
        {text: 'Address', sortable: false, align: 'left',},
        {text: 'City', sortable: false, align: 'left',},
        {text: 'State', sortable: false, align: 'left',},
        {text: 'ZipCode', sortable: false, align: 'left',},
        {text: 'Email', sortable: false, align: 'left',},
        {text: 'Phone', sortable: false, align: 'left',},
        {text: 'Update', sortable: false, align: 'left',},
        {text: 'Delete', sortable: false, align: 'left',}
      ],

    }),
    mounted() {
      this.getCustomers();
      this.showMessages();
    },
    methods: {
      showMessages(){
        console.log(this.$route.params.msg)
         if (this.$route.params.msg) {
           this.showMsg = this.$route.params.msg;
         }
      },
      getCustomers() {
        apiService.getCustomerList().then(response => {
          this.customers = response.data.data;
          this.customerSize = this.customers.length;
          if (localStorage.getItem("isAuthenticates")
            && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
            this.validUserName = JSON.parse(localStorage.getItem("log_user"));
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      },
      addNewCustomer() {
        if (localStorage.getItem("isAuthenticates")
          && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
          router.push('/customer-create');
        } else {
          router.push("/auth");
        }
      },
      updateCustomer(customer) {
        router.push('/customer-create/' + customer.pk);
      },
      deleteCustomer(customer) {
        let isDelete = confirm("Are you sure want to delete this customer??");
        if(isDelete) {
          apiService.deleteCustomer(customer.pk).then(response => {
          if (response.status === 204) {
            
            this.showMsg = 'deleted';
            this.$router.go();
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
        }
      }
    }
  };
</script>
<style >
.overflow .v-table__overflow {
  overflow:unset !important;
}
</style>

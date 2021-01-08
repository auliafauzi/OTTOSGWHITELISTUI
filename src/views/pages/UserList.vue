<template>
  <div class="animated fadeIn">
    <!-- <div>{{this.currentDate | dateParse('YYYY-MM-DD') | dateFormat('MMMM D, YYYY') }}</div> -->

    <div class = "text-left">
      <b-table show-empty stacked="md" responsive="sm" :items="dataUser" :fields="fields2">
               <template v-slot:cell(status)="row">
                </template>

               <template v-slot:cell(Actions)="row">

                 <b-button variant="warning" size="sm" @click="editModal(row.item.Company)" class="mr-1">
                   EDIT
                 </b-button>
                <b-button variant="danger" size="sm" @click="deleteConfirm(row.item.Company)" class="mr-1">
                  DELETE
                </b-button>

              </template>
             </b-table>
             <b-button variant="primary" size="sm" @click="addUserModal()" class="mr-1">
               Add New User
             </b-button>

             <b-modal id="deleteModal"
                     v-model="deleteModal"
                     ok-title="Ok" @ok="deleteCompany()">
                     Do you Really want to Delete this Company from List ?
             </b-modal>
             <b-modal id="AddUserModal" title="Add New User"
                     v-model="AddUserModal"
                     ok-title="Ok" @ok="addUser()">
                     <b-form-input type="text"
                             placeholder="Name"
                             v-model="Name"></b-form-input><p/>
                     <b-form-input type="text"
                             placeholder="Email"
                             v-model="Email"></b-form-input><p/>
                     <b-form-input type="text"
                             placeholder="Company"
                             v-model="Company"></b-form-input><p/>
                     <b-form-input type="text"
                             placeholder="Role"
                             v-model="Role"></b-form-input><p/>
                     <b-form-input type="text"
                             placeholder="User Name"
                             v-model="UserName"></b-form-input><p/>
                     <b-form-input type="text"
                             placeholder="Password"
                             v-model="Password"></b-form-input><p/>
             </b-modal>
             <b-modal id="editCompanyModal"
                     v-model="editModalToogle"
                     ok-title="Ok" @ok="editCompany()">Edit Company {{compNameToEdit}}
                     <b-form-input type="text"
                             placeholder="New Company Name"
                             v-model="companyNewName"></b-form-input>
             </b-modal>

    </div>

  <!-- </b-card> -->
</div>
</template>

<script>
import axios from "axios";
import {getToken} from '../../utils';
export default {
  data: () => ({
    dataUser: [],
    totalUser: '',
    fields2 :[
      { key: 'username', label: 'User Name'},
      { key: 'email', label: 'Email' },
      { key: 'role', label: 'Role' },
      { key: 'company', label: 'Company' },
      { key: 'active', label: 'Active Status' }
    ],
    UserName: '',
    Name:'',
    Email:'',
    Role: '',
    Company: '',
    Password:'',
    deleteModal : false,
    AddUserModal : false,
    editModalToogle : false,
    compNameToDelete : '',
    compNameToEdit: '',
    companyToAdd : '',
    companyNewName: '',
    currentDate : new Date(),
    token : ''
  }),
  mounted() {
    this.getData()
  },
  methods: {
    // getData() {
    //   this.token = getToken()
    //   axios({
    //     method: "get",
    //     url: `https://antares.pede.id/sg-web-service/v1/users?page=1&size=5`,
    //     // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
    //     headers: {Authorization: `Bearer ${this.token}`},
    //     withCredentials: true // True otherwise I receive another error
    //   }).then(response => {
    //     if (response.status === 200) {
    //       this.dataUser = response.data.data.user;
    //       this.totalUser = response.data.data.total;        }
    //   })
    // },
    getData() {
      console.log("here")
      console.log(getToken())
      this.token = getToken()
      var headers = {
        'Content-Type': 'application/json',
        // 'Authorization': `Bearer ${this.state.login.access_token}`
        // 'Authorization': 'Bearer ' + getToken()
        'Authorization': `Bearer ${this.token}`
        // 'Authorization': getToken()
      }
      console.log(headers)
      axios.get("https://antares.pede.id/sg-web-service/v1/users?page=1&size=5",{
            headers
            },
          // }
        ).then(response => {
          if (response.status === 200) {
                console.log(response.data);
                this.totalUser = response.data.data.total;
                this.dataUser = response.data.data.user;
              }
            })
    },
    deleteConfirm(compName){
      this.deleteModal = true
      this.compNameToDelete = compName
    },
    editModal(OldCompName){
      this.editModalToogle = true
      this.compNameToEdit = OldCompName
    },
    addUserModal(){
      this.AddUserModal = true
    },
    editCompany(){
      console.log(this.compNameToEdit)
      axios({
        method: "post",
        url: `http://0.0.0.0:3000/api/v1/editCompany/${this.compNameToEdit}/${this.companyNewName}`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
          this.logdata = response.data;
          console.log(response.data)
          // this.getData()
          setTimeout(() => {this.getData()}, 300);
          // this.logfilename = logfilename;
          // this.$root.$emit('bv::show::modal', this.logModal.id)
        }
        this.compNameToEdit = ''
        this.companyNewName = ''
      })
    },
    deleteCompany(){
      axios({
        method: "post",
        url: `http://0.0.0.0:3000/api/v1/delCompany/${this.compNameToDelete}`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
          this.logdata = response.data;
          // this.getData()
          setTimeout(() => {this.getData()}, 300);
          // this.logfilename = logfilename;
          // this.$root.$emit('bv::show::modal', this.logModal.id)
        }
        this.compNameToDelete = ''
      })
    },
    addUser(){
      console.log(this.companyToAdd)
      axios({
        method: "post",
        url: `http://0.0.0.0:3000/api/v1/addCompany/${this.companyToAdd}`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
          this.logdata = response.data;
          console.log(response.data)
          // this.getData()
          setTimeout(() => {this.getData()}, 300);
          // this.logfilename = logfilename;
          // this.$root.$emit('bv::show::modal', this.logModal.id)
        }
        this.companyToAdd = ''
      })
    },
    // info(item, logfile, index, button) {
    //     this.infoModal.title = `Row index: ${index}`
    //     this.infoModal.content = JSON.stringify(item, null, 2)
    //     this.infoModal.logfilename = logfile
    //     this.$root.$emit('bv::show::modal', this.infoModal.id, button)
    //   },
    resetLogfoModal() {
        this.logModal.title = ''
        this.logModal.content = ''
        this.logdata = ''
      },
    resetInfofoModal() {
        this.infoModal.title = ''
        this.infoModal.content = ''
        this.logdata = ''
      },
    getLogFile(logfilename) {
      axios({
        method: "get",
        url: `http://0.0.0.0:3000/api/v1/openLog/${logfilename}`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
          this.logdata = response.data;
          this.logfilename = logfilename;
          this.$root.$emit('bv::show::modal', this.logModal.id)
        }
      })
    },
    getJobInfo(confFilename) {
      axios({
        method: "get",
        url: `http://0.0.0.0:3000/api/v1/openInfo/${confFilename}`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
          this.infodata = response.data;
          // this.logfilename = logfilename;
          this.$root.$emit('bv::show::modal', this.infoModal.id)
        }
      })
    }
  }
};
</script>

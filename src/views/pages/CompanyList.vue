<template>
  <div class="animated fadeIn">
    <!-- <div>{{this.currentDate | dateParse('YYYY-MM-DD') | dateFormat('MMMM D, YYYY') }}</div> -->
    <!-- <div>
      <ejs-datepicker :value="currentDate" :format="dateFormat"></ejs-datepicker>
    </div> -->
  <!-- <b-card> -->
    <!-- <div class="justify-content-center row my-1"> -->
    <template v-if="errorAlert">
      <b-row>
        <b-col sm="12">
          <b-alert show variant="danger">{{errorMessage}}</b-alert>
        </b-col>
      </b-row>
    </template>

    <div class = "text-left">
      <b-table show-empty stacked="md" responsive="sm" :items="dataCompany" :fields="fields2">
               <template v-slot:cell(status)="row">
                 <!-- <div v-if="row.item.status == this.currentDate">
                   active
                 </div> -->
                 <!-- <div v-else>
                   {{row.item.status}}
                 </div> -->
                 <!-- <div class="green">
                   ACTIVE
                 </div>  -->
                 <!-- {{row.item.status}} -->
               </template>
               <template v-slot:cell(Actions)="row">

                 <b-button variant="warning" size="sm" @click="editModal(row.item.Company)" class="mr-1">
                   EDIT
                 </b-button>
                <b-button variant="danger" size="sm" @click="deleteConfirm(row.item.Company)" class="mr-1">
                  DELETE
                </b-button>

                <!-- <p/>
                <b-button variant="primary" size="sm" @click="getJobInfo(row.item.jobname)" class="mr-2">
                  Open Job Info
                </b-button> -->

              </template>
             </b-table>
             <b-button variant="primary" size="sm" @click="addCompanyModal()" class="mr-1">
               Add New Company
             </b-button>

             <b-modal id="deleteModal"
                     v-model="deleteModal"
                     ok-title="Ok" @ok="deleteCompany()">
                     Do you Really want to Delete this Company from List ?
             </b-modal>
             <b-modal id="AddCompanyModal"
                     v-model="AddCompanyModal"
                     ok-title="Ok" @ok="addCompany()">
                     <b-form-input type="text"
                             placeholder="Company Name"
                             v-model="companyToAdd"></b-form-input>
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
export default {
  data: () => ({
    dataCompany: [],
    fields :[
      { key: 'jobname'},
      { key: 'status', label: 'Status' },
      { key: 'error', label: 'Error' },
      { key: 'logfile', label: 'Logfile' }
      // ,
      // { key: 'datenow', label: 'datenow' }
      // { key: 'logfilename', label: 'Logfilename', thClass: 'd-none' },
    ],
    fields2 :[
      { key: 'Company', label: 'Company Name'},
      { key: 'Records', label: 'Records' },
      { key: 'Created Date', label: 'Created Date' },
      // { key: 'Stat', label: 'Status' },
      { key: 'Actions', label: 'Actions' }
    ],
    deleteModal : false,
    AddCompanyModal : false,
    editModalToogle : false,
    compNameToDelete : '',
    compNameToEdit: '',
    companyToAdd : '',
    companyNewName: '',
    currentDate : new Date(),
    errorAlert: false,
    errorMessage: ''
  }),
  mounted() {
    this.getData()
    // axios({
    //   method: "get",
    //   url: `http://0.0.0.0:3000/api/v1/CompanyList`,
    //   // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
    //   withCredentials: true // True otherwise I receive another error
    // }).then(response => {
    //   if (response.status === 200) {
    //     // const databuku = response.data;
    //     this.dataCompany = response.data.CompanyList;
    //     console.log(this.dataCompany)
    //   }
    // })
  },
  methods: {
    datefunc() {
      this.currentDate = new Date();
    },
    getData() {
      axios({
        method: "get",
        url: `http://0.0.0.0:3000/api/v1/CompanyList`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
          // const databuku = response.data;
          this.dataCompany = response.data.CompanyList;
          console.log(this.dataCompany)
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
    addCompanyModal(){
      this.AddCompanyModal = true
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
    addCompany(){
      console.log(this.companyToAdd)
      if (this.companyToAdd === '') {
        console.log("Company Name Cannot Empty");
        this.errorMessage = "Company Name Cannot Empty";
        this.errorAlert = true;
        this.AddCompanyModal = false;
        setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
      } else {
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
      }
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

<template>
  <div style="overflow: scroll;">
  <div class="animated fadeIn">

  <!-- <b-card> -->
    <!-- <div class="justify-content-center row my-1"> -->
    <template v-if="errorAlert">
      <b-row>
        <b-col sm="12">
          <b-alert show variant="danger">{{errorMessage}}</b-alert>
        </b-col>
      </b-row>
    </template>
    <template v-if="SuccessAlert">
      <b-row>
        <b-col sm="12">
          <b-alert show variant="success">{{successMessage}}</b-alert>
        </b-col>
      </b-row>
    </template>

    <div class = "text-left">
      <div>
      <b-table
        class="my-table"
        head-row-variant="transparent"
        :sticky-header="this.DynamicHeight"
        variant="success"
        show-empty
        responsive="sm"
        :items="dataCompany"
        :fields="fields"
        @row-clicked="companyDetailModal" >
      </b-table>

       <b-pagination
       v-model="currentPage"
       :total-rows="rows()"
       :per-page="perPage"
       limit=6
       aria-controls="my-table"
       size="sm"
       @change="loadPage"
       ></b-pagination>

      </div>

             <b-button variant="primary" size="sm" @click="addCompanyModal()" class="mr-1">
               Add New Company
             </b-button>

             <b-modal id="deleteModal"
                     v-model="deleteModal"
                     ok-title="Ok" @ok="deleteCompany()">
                     Do you really want to delete this Company from List ?
             </b-modal>

             <b-modal id="companyDetail" v-model="DetailToggle" ok-title="Ok">

               <template v-if="errorAlertinDetail">
                 <b-row>
                   <b-col sm="12">
                     <b-alert show variant="danger">{{errorMessage}}</b-alert>
                   </b-col>
                 </b-row>
               </template>

              <template v-if="this.ImagePath !== ''">
               <img class="logo-mini" :src="this.ImagePath" alt="logo"
               height="150px"
               width="150px"/></p></template>
               <b-button variant="outline-primary" size="sm" @click="changeLogo(DataDetail.id)" class="mr-1">
                 Change Logo
               </b-button>
               <div class="mt-3">Company:</div>
               <div class="mt-3"> <pre> {{DataDetail.name}}</pre> </div>
               <b-button variant="outline-primary" size="sm" @click="editModal(DataDetail.name, DataDetail.id)" class="mr-1">
                 Change Name
               </b-button>
               <div class="mt-3">Created Date:</div>
               <div class="mt-3"><pre> {{DataDetail.createdDateString}}</pre> </div>
               <div class="mt-3">Updated Date: <strong></strong></div>
               <div class="mt-3"><pre> {{DataDetail.updateDateString}}<strong></strong></pre> </div>
               <div class="mt-3">Created By : <strong></strong></div>
               <div class="mt-3"><pre> {{DataDetail.createdBy}}<strong></strong></pre> </div>
               <div class="mt-3">Updated By: <strong></strong></div>
               <div class="mt-3"><pre> {{DataDetail.updatedBy}}<strong></strong></pre> </div>
             </b-modal>

             <b-modal id="AddCompanyModal"
                v-model="addModalToggle"
                ok-title="Ok" @ok="addCompany" @hide="resetModal">
               <template v-if="errorAlertinAdd">
                 <b-row>
                   <b-col sm="12">
                     <b-alert show variant="danger">{{errorMessage}}</b-alert>
                   </b-col>
                 </b-row>
               </template>

               <b-form-input type="text"
                       placeholder="Company Name"
                       v-model="companyToAdd"
                       aria-describedby="input-live-feedback"
                       trim
                       >
               </b-form-input>
               <!-- <b-form-invalid-feedback id="input-live-feedback">
                 Company Name Tidak Boleh Kosong
               </b-form-invalid-feedback> -->
             </b-modal>

             <b-modal id="editCompanyModal"
               v-model="editModalToogle"
               ok-title="Ok"
               @ok="editCompany"
               @hide="resetModal"
               >Edit Company Name :  {{compNameToEdit}}
               <template v-if="errorAlertinEdit">
                 <b-row>
                   <b-col sm="12">
                     <b-alert show variant="danger">{{errorMessage}}</b-alert>
                   </b-col>
                 </b-row>
               </template>
               <b-form-input type="text"
                       placeholder="New Company Name"
                       v-model="companyNewName">
               </b-form-input>
             </b-modal>

             <b-modal id="uploadLogoModal"
                     v-model="UploadLogoToggle"
                     @hide="resetModal"
                     ok-title="Ok" @ok="uploadLogo">Change logo for :  {{compNameToEdit}}
                     <label>
                       <input type="file" id="file" accept=".png,.jpg,.jpeg" ref="file" v-on:change="handleFileUpload()"/>
                     </label>
                     <template v-if="errorAlertinUploadLogo">
                       <b-row>
                         <b-col sm="12">
                           <b-alert show variant="danger">{{errorMessage}}</b-alert>
                         </b-col>
                       </b-row>
                     </template>

             </b-modal>

    </div>

  <!-- </b-card> -->
</div>
</div>
</template>

<script>
import axios from "axios";
import {getToken, getUserDataInSession2, BASE_URL} from '../../utils';
export default {
  data: () => ({
    dataCompany: [],
    DataDetail: {},
    fields :[
      { key: 'name', label: 'Company Name' },
      { key: 'totalEmployee', label: 'Records' },
      // { key: 'createdDate', label: 'Created Date' },
      // { key: 'updateDate', label: 'Updated Date' },
      // { key: 'createdBy', label: 'Created By' },
      // { key: 'updatedBy', label: 'Updated By' },
      // { key: 'Actions', label: 'Actions' },
      // { key: 'Actions', label: 'Actions2' },
      // { key: 'Actions', label: 'Actions3' },
      // { key: 'Actions', label: 'Actions4' }
    ],
    fields2 :[
      { key: 'Company', label: 'Company Name'},
      { key: 'Records', label: 'Records' },
      { key: 'Created Date', label: 'Created Date' },
      // { key: 'Stat', label: 'Status' },
      { key: 'Actions', label: 'Actions' }
    ],
    deleteModal : false,
    addModalToggle : false,
    editModalToogle : false,
    DetailToggle: false,
    UploadLogoToggle: false,
    SuccessAlert: false,
    successMessage : '',
    compNameToDelete : '',
    compNameToEdit: '',
    companyToAdd : '',
    companyNewName: '',
    CompanyId: '',
    ImagePath : '',
    currentDate : new Date(),
    errorAlert: false,
    errorAlertinDetail: false,
    errorAlertinEdit: false,
    errorAlertinAdd: false,
    errorAlertinUploadLogo : false,
    errorMessage: '',
    perPage : 10,
    currentPage : 1,
    TotalData : 0,
    DynamicHeight : '',
    HomeLocation : '/ottosg_webportal'
  }),
  mounted() {
    this.getData()
    this.dynamicHeight()
  },
  created() {
  window.addEventListener("resize", this.myEventHandler);
  },
  destroyed() {
    window.removeEventListener("resize", this.myEventHandler);
  },
  methods: {
    datefunc() {
      this.currentDate = new Date();
    },
    myEventHandler(e) {
    // your code for handling resize...
    this.dynamicHeight()
    },
    dynamicHeight(){
      var TableHeight = window.innerHeight - 230
      this.DynamicHeight =  TableHeight.toString().concat("px")
    },
    // nameState() {
    //   return this.companyToAdd.length > 0 ? true : false
    // },
    getData() {
      // this.userInfo = getUserDataInSession2("UserInfo")
      this.token = getToken()
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      // console.log(`${BASE_URL}/companies?page=${this.currentPage}&size=${this.perPage}`)
      axios.get(`${BASE_URL}/companies?page=${this.currentPage}&size=${this.perPage}`,{
            headers
            },
          // }
        ).then(response => {
          if (response.status === 200) {
            if (response.data.code === 401) {
              this.errorAlert = true;
              this.errorMessage = response.data.message;
              setTimeout(() => {this.errorMessage= '',console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
            } else {
                this.dataCompany = response.data.data.user;
                this.TotalData = Number(response.data.data.total)
                // console.log(response.data.message + "HERE1")
                // setTimeout(() => {this.errorMessage= '',console.log(response.data.message + "HERE"),window.location.href = this.HomeLocation}, 2000);
            }
          }
        })
    },
    deleteConfirm(compName){
      this.deleteModal = true
      this.compNameToDelete = compName
    },
    editModal(OldCompName, id){
      this.editModalToogle = true
      this.compNameToEdit = OldCompName
      this.CompanyId = id
    },
    companyDetailModal(Data){
      this.DetailToggle = true
      this.DataDetail = Data
      // let c = this.DataDetail.updateDate.substring(0,10)
      this.DataDetail.updateDateString = this.DataDetail.updateDate.substring(0,10)
      this.DataDetail.createdDateString = this.DataDetail.createdDate.substring(0,10)

      // this.DataDetail.updateDateNew = this.DataDetail.updateDate
      this.ImagePath = Data.filepath
    },
    resetModal(){
      this.companyName = ''
      this.companyToAdd = ''
      this.companyNewName = ''
    },
    addCompanyModal(){
      this.addModalToggle = true
    },
    changeLogo(id){
      this.UploadLogoToggle = true
      this.CompanyId = id
    },
    editCompany(bvModalEvt){
      if (this.companyNewName === '') {
        bvModalEvt.preventDefault()
        this.errorMessage = "Company Name Tidak Boleh Kosong";
        this.errorAlertinEdit = true;
        this.editModalToogle = true;
        // this.DetailToggle = false;
        setTimeout(() => {this.errorAlertinEdit = false, this.errorMessage = ''}, 3000);
      } else {
        var headers = {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.token}`
        };
        axios.put("${BASE_URL}/company",{
          name: this.companyNewName,
          id: Number(this.CompanyId)
        }, {headers}).then(response => {
          if (response.status === 200) {
            if (response.data.code !== 401 && response.data.code !== 200) {
              this.editModalToogle = true;
              this.errorMessage = response.data.message;
              this.errorAlertinEdit = true ;
              setTimeout(() => {this.errorAlertinEdit = false, this.errorMessage = '' }, 5000);
            } else if (response.status === 200 && response.data.code === 401) {
              this.editModalToogle = true;
              this.errorMessage = response.data.message;
              this.errorAlertinEdit = true ;
              setTimeout(() => {this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
            }
            else {
              this.logdata = response.data;
              this.DetailToggle = false
              this.successMessage = response.data.status
              this.SuccessAlert = true
              this.getData()
              setTimeout(() => {this.successMessage = '', this.SuccessAlert = false}, 3000);
            }
          }
          this.resetModal()
        }).catch((error) => {
          if (error.response !== undefined) {
            this.DetailToggle = false
            this.errorMessage = error.response
            this.errorAlert = true;
            setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
          } else {
            this.DetailToggle = false
            this.errorMessage = error.response
            this.errorAlert = true;
            setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
          }
          this.resetModal()
        });
      }
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
          setTimeout(() => {this.getData()}, 300);
        } else {
          this.errorMessage = "Internal Server Error";
          this.errorAlert = true;
          setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
        }
        this.compNameToDelete = ''
      }).catch((error) => {
        if (error.response !== undefined) {
          this.errorMessage = "Internal Server Error";
          this.errorAlert = true;
          setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
        } else {
          this.errorMessage = "Internal Server Error";
          this.errorAlert = true;
          setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
        }
      });
    },
    addCompany(bvModalEvt){
      if (this.companyToAdd === '') {
        this.errorMessage = "Company Name Tidak Boleh Kosong";
        // this.$bvModal.hide('AddCompanyModal')
        // this.$bvModal.show('AddCompanyModal')
        bvModalEvt.preventDefault()
        this.errorAlertinAdd = true;
        this.addModalToggle = true;
        setTimeout(() => {this.errorAlertinAdd = false, this.errorMessage = ''}, 5000);
      } else {
        var headers = {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.token}`
        };
        axios.post("${BASE_URL}/company",{
          name: this.companyToAdd,
        }, {headers}).then(response => {
          if (response.status === 200) {
            if (response.data.code !== 401 && response.data.code !== 200) {
              this.errorMessage = response.data.message;
              this.errorAlert = true ;
              setTimeout(() => {this.errorAlert = false, this.errorMessage = '' }, 5000);
            } else if (response.status === 200 && response.data.code === 401) {
              this.errorMessage = response.data.message;
              this.errorAlert = true ;
              setTimeout(() => {this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
            }
            else {
              this.logdata = response.data;
              this.successMessage = response.data.status
              this.SuccessAlert = true
              this.getData()
              setTimeout(() => {this.successMessage = '', this.SuccessAlert = false}, 3000);
            }
          }
          this.resetModal()
        }).catch((error) => {
          if (error.response !== undefined) {
            this.DetailToggle = false
            this.errorMessage = error.response
            this.errorAlert = true;
            setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
          } else {
            this.DetailToggle = false
            this.errorMessage = error.response
            this.errorAlert = true;
            setTimeout(() => {this.errorAlert = false, this.errorMessage = ''}, 3000);
          }
          this.resetModal()
        });
      }
    },

    resetLogfoModal() {
        this.logModal.title = ''
        this.logModal.content = ''
        this.logdata = ''
      },
    rows() {
      return Number(this.TotalData)
     },
     loadPage(pageNum) {
       // this.getTrades(pageNum);
       this.currentPage = pageNum;
       this.getData()
     },
     handleFileUpload() {
       this.file = this.$refs.file.files[0];
     },
     uploadLogo(evt) {
       if (this.file === '' || this.file === undefined) {
         evt.preventDefault();
         // this.$refs.modalRestore.show();
         this.errorMessage = 'File tidak boleh kosong';
         this.errorAlertinUploadLogo = true;
         setTimeout(() => {this.errorAlertinUploadLogo = false, this.file = ''}, 5000);
       }
       else {
         this.errorRestore = false;
         this.submitFile();
       }
     },
     submitFile() {
       this.token = getToken()
       const formData = new FormData();
       formData.append('file', this.file);
       formData.append('company-id', this.CompanyId);

       var headers = {
         'Content-Type': 'multipart/form-data',
         'Authorization': `Bearer ${this.token}`
       };
       axios.put(`${BASE_URL}/company/image`,
       formData,{ headers}).then((response) => {
         if (response.status === 200) {
           if (response.data.status === "Success") {
             this.DetailToggle = false
             this.successMessage = response.data.status
             this.SuccessAlert = true;
             setTimeout(() => {this.SuccessAlert = false, this.file = ''}, 5000);
           } else if (response.data.status === false) {
             this.errorMessage = response.data.message;
             this.errorAlertinDetail = true;
             setTimeout(() => {this.errorAlertinDetail = false, this.file = ''}, 5000);
           } else {
             this.errorMessage = response.data.message;
             this.errorAlertinDetail = true;
             setTimeout(() => {this.errorAlertinDetail = false, this.file = ''}, 5000);
           }
         } else {
           this.errorMessage = response.data.message;
           this.errorAlertinDetail = true;
           setTimeout(() => {this.errorAlertinDetail = false, this.file = ''}, 5000);
         }
         this.getData()
       }).catch((error) => {
         if (error.response !== undefined) {
           this.success = false;
           // setAlert(this, error.response);
         }
       });
     },


  }
};
</script>

<style scoped>
.my-table {
  position: sticky;
  cursor: pointer;
}
.red_color {
  color: red;
}
/deep/ .my-table th {
  position: sticky;
  /* background-color: #f4f7ff; */

  background-color: #007bff;
  color: white;
  font-size:  16px;
  font-weight: 500;
}

</style>

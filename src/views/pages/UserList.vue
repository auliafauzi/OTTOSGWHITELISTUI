<template>
  <div style="overflow: scroll;">
  <div class="animated fadeIn">
    <template v-if="alertToggle">
      <b-row>
        <b-col sm="12">
          <b-alert show variant="danger">{{alertMassage}}</b-alert>
        </b-col>
      </b-row>
    </template>
    <template v-if="successToggle">
      <b-row>
        <b-col sm="12">
          <b-alert show variant="success">{{successMassage}}</b-alert>
        </b-col>
      </b-row>
    </template>

    <div class = "text-left">
      <b-table class="my-table"
               head-row-variant="transparent"
               :sticky-header="this.DynamicHeight"
               variant="success"
               show-empty
               responsive="sm"
               :items="dataUser"
               :fields="fields2"
               @row-clicked="editUserModal"  >
               <template v-slot:cell(ActiveStatus)="row">
                 <div v-if="row.item.active===true">Active</div>
                 <div class="danger" v-if="row.item.active===false" text-variant="white">Non-Active</div>
                </template>

               <!-- <template v-slot:cell(Actions)="row">
                 <b-button variant="primary" size="sm" @click="editUserModal(row.item.id, row.item.username,row.item.name, row.item.email)" class="mr-1">
                   EDIT
                 </b-button>
              </template> -->
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

             <b-button variant="primary" size="sm" @click="addUserModal()" class="mr-1">
               Add New User
             </b-button>

             <b-modal id="deleteModal"
                     v-model="deleteModal"
                     ok-title="Ok" @ok="deleteCompany()">
                     Do you Really want to Delete this Company from List ?
             </b-modal>
             <b-modal checkFormValidity id="AddUserModal" title="Add New User"
                     v-model="AddUserModalToggle"
                     ok-title="Ok" @ok="handleOk" @cancel="resetModal">

                     <template v-if="alertAddModalToggle">
                       <b-row>
                         <b-col sm="12">
                           <b-alert show variant="danger">{{alertMassage}}</b-alert>
                         </b-col>
                       </b-row>
                     </template>
                    <div class="mt-3">Name: <strong></strong></div>
                    <b-form-input type="text"
                       placeholder="Name"
                       v-model="Name">
                     </b-form-input><p/>
                     <div class="mt-3">Email: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="Email"
                             v-model="Email"></b-form-input><p/>
                     <div class="mt-3">Role: <strong></strong></div>
                      <b-form-select v-model="roleSelected" :options="roleOptions"></b-form-select>
                      <!-- <div class="mt-3">Selected: <strong>{{ roleSelected }}</strong></div> -->
                      <!-- <b-form-input type="text"
                              placeholder="Role"
                              v-model="Role"></b-form-input><p/> -->
                     <!-- <b-form-input type="text"
                             placeholder="Company"
                             v-model="Company"></b-form-input><p/> -->
                    <div v-if="roleSelected===2" class="mt-3">Company: <strong></strong></div>
                    <b-form-select v-if="roleSelected===2" v-model="companySelected" :options="companyOptions"></b-form-select>
                    <!-- <div class="mt-3">Selected: <strong>{{ companySelected }}</strong></div> -->
                    <div class="mt-3">Username: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="User Name"
                             v-model="UserName"></b-form-input><p/>
                    <div class="mt-3">Password: <strong></strong></div>
                     <b-form-input type="password"
                             placeholder="Password"
                             v-model="Password"></b-form-input><p/>
             </b-modal>

             <b-modal checkFormValidity id="EditUserModal"
                      title="Edit User"
                      v-model="editModalToogle"
                      ok-title="Ok"
                      @ok="editUser"
                      @cancel="resetModal"
                      >
                     <b>User name :
                     {{UserName}}</b>
                     <template v-if="alertAddModalToggle">
                       <b-row>
                         <b-col sm="12">
                           <b-alert show variant="danger">{{alertMassage}}</b-alert>
                         </b-col>
                       </b-row>
                     </template>
                    <div class="mt-3">Name: <strong></strong></div>
                    <b-form-input type="text"
                       placeholder="Name"
                       v-model="Name">
                     </b-form-input><p/>
                     <div class="mt-3">Email: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="Email"
                             v-model="Email"></b-form-input><p/>
                    <div class="mt-3">Username: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="User Name"
                             disabled
                             v-model="UserName"></b-form-input><p/>
                   <div class="mt-3">Active Status: <strong></strong></div>
                   <b-form-select v-model="Active" :options="ActiveOptions"></b-form-select>
             </b-modal>

             <!-- <b-modal id="editCompanyModal"
                     v-model="editModalToogle"
                     ok-title="Ok" @ok="editCompany()">Edit Company {{compNameToEdit}}
                     <b-form-input type="text"
                             placeholder="New Company Name"
                             v-model="companyNewName"></b-form-input>
             </b-modal> -->

    </div>

  <!-- </b-card> -->
</div>
</div>
</template>

<script>
import axios from "axios";
import {getToken, BASE_URL} from '../../utils';
export default {
  data: () => ({
    dataUser: [],
    dataCompany: [],
    dataRole: [],
    TotalData: '',
    DataDetail: {},
    fields2 :[
      // { key: 'id', label: 'User ID'},
      { key: 'username', label: 'User Name'},
      { key: 'email', label: 'Email' },
      { key: 'role', label: 'Role' },
      { key: 'company', label: 'Company' },
      { key: 'ActiveStatus', label: 'Status' },
      // { key: 'Actions', label: 'Actions' }
    ],
    companyOptions : [
      { value: null, text: 'Please Select Company ', disabled: true  },
    ],
    roleOptions : [
      { value: null, text: 'Please Select Role ', disabled: true  },
    ],
    ActiveOptions : [
      { value: true, text: 'Active' },
      { value: false, text: 'Non-Active' },
    ],
    keyValueOption : {},
    companySelected : '',
    roleSelected : '',
    UserName: '',
    Name:'',
    Email:'',
    Role: '',
    Company: '',
    Password:'',
    UserId : '',
    Active: '',
    deleteModal : false,
    AddUserModalToggle : false,
    editModalToogle : false,
    alertToggle: false,
    alertAddModalToggle: false,
    successToggle: false,
    successMassage: '',
    alertMassage:'',
    compNameToDelete : '',
    userToEdit: '',
    companyToAdd : '',
    companyNewName: '',
    currentDate : new Date(),
    token : '',
    perPage: 10,
    currentPage: 1,
    DynamicHeight : '',
    HomeLocation : '/ottosg_webportal'
  }),
  mounted() {
    // this.resetOptions()
    this.getData()
    this.getCompaniesAndRolesList()
    this.dynamicHeight()
  },
  created() {
  window.addEventListener("resize", this.myEventHandler);
  },
  destroyed() {
    window.removeEventListener("resize", this.myEventHandler);
  },
  methods: {
    // getData() {
    //   this.token = getToken()
    //   axios({
    //     method: "get",
    //     url: `${BASE_URL}/users?page=1&size=5`,
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
      this.token = getToken()
      var headers = {
        'Content-Type': 'application/json',
        // 'Authorization': `Bearer ${this.state.login.access_token}`
        // 'Authorization': 'Bearer ' + getToken()
        'Authorization': `Bearer ${this.token}`
        // 'Authorization': getToken()
      }
      axios.get(`${BASE_URL}/users?page=`+this.currentPage+`&size=`+this.perPage,{
            headers
            },
        ).then(response => {
          if (response.status === 200 && response.data.code === 200) {
            this.TotalData = response.data.data.total;
            this.dataUser = response.data.data.user;
          } else if (response.status === 200 && response.data.code === 401) {
            this.alertToggle = true;
            this.alertMassage = response.data.message;
            setTimeout(() => {this.alertMassage= '',console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
          }
        });
    },
    getCompaniesAndRolesList(){
      this.token = getToken()
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      axios.get(`${BASE_URL}/companies`,{
            headers
            },
        ).then(response => {
          if (response.status === 200) {
                this.dataCompany = response.data.data;
                this.optionsFunc(this.dataCompany,this.companyOptions)
              }
            });
      axios.get(`${BASE_URL}/roles`,{
            headers
            },
        ).then(response => {
          if (response.status === 200) {
            // console.log(response.data.data)
                this.dataCompany = response.data.data;
                this.optionsFunc(this.dataCompany, this.roleOptions)
              }
            })
    },
    optionsFunc(data, target){
      for (let i= 0; i < data.length; i++) {
        this.keyValueOption["value"] = data[i]["id"]
        this.keyValueOption["text"] = data[i]["name"]
        target.push(this.keyValueOption) //append keyvalue obj to options array
        this.keyValueOption = {} //reset keyvalue Object
      }
      // console.log(target)
    },
    myEventHandler(e) {
    // your code for handling resize...
    this.dynamicHeight()
    },
    dynamicHeight(){
      var TableHeight = window.innerHeight - 210
      this.DynamicHeight =  TableHeight.toString().concat("px")
    },
    deleteConfirm(compName){
      this.deleteModal = true
      this.compNameToDelete = compName
    },
    editUserModal(data){
      // console.log(data)
      this.UserId = data.id
      this.UserName = data.username
      this.Name = data.name
      this.Email = data.email
      this.Active = data.active
      this.editModalToogle = true
    },
    addUserModal(){
      this.AddUserModalToggle = true
    },
    checkFormValidity() {
      const valid = this.$refs.form.checkValidity()
      this.nameState = valid
      return valid
    },
    resetModal(){
      console.log("resetmodal")
      this.roleSelected = ''
      this.Name = ''
      this.UserName = ''
      this.Email = ''
      this.Password = ''
      this.companySelected = ''
    },
    resetOptions(){
      this.companyOptions = []
      this.roleOptions = []
    },
    handleOk(bvModalEvt) {
      // console.log("Hanle Ok")
      if (this.roleSelected === 1) {
        this.companySelected = 0
      } else {};
      if (this.roleSelected === '' || this.Name === '' || this.UserName === '' || this.Email === '' || this.Password === '' || this.companySelected === ''){
        this.alertMassage = "Data tidak boleh kosong"
        this.alertAddModalToggle = true ;
        setTimeout(() => {this.alertAddModalToggle = false, this.alertMassage = '' }, 5000);
        bvModalEvt.preventDefault() // Prevent modal from closing
      } else {
        this.addUser(bvModalEvt) // Trigger submit handler
      }
    },
    addUser(bvModalEvt){
      // console.log("add User")
      this.alertAddModalToggle = false;
      if (Number(this.roleSelected) === 1) {
        this.companySelected = 0
      }
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      };
      // console.log("roleSelected " +Number(this.roleSelected))
      // console.log("NAME " + this.Name)
      // console.log("UserName " + this.UserName)
      // console.log("Email " + this.Email)
      // console.log("Password " + this.Password)
      // console.log("companySelected " + Number(this.companySelected))
        axios.post("${BASE_URL}/user",{
                roleId: Number(this.roleSelected),
                name: this.Name,
                username: this.UserName,
                email: this.Email,
                password: this.Password,
                companyId: Number(this.companySelected)
              }, {headers}
          ).then(response => {
          if (response.status === 200) {
            if (response.data.code !== 401 && response.data.code !== 200) {
              this.alertMassage = response.data.message;
              this.alertAddModalToggle = true ;
              // bvModalEvt.preventDefault()
              this.AddUserModalToggle = true;
              setTimeout(() => {this.alertAddModalToggle = false, this.alertMassage = '' }, 5000);
            } else if (response.status === 200 && response.data.code === 401) {
              this.resetModal()
              this.alertMassage = response.data.message
              this.alertToggle = true
              setTimeout(() => {this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
            }
            else {
              // this.logdata = response.data;
              this.resetModal()
              this.resetOptions()
              this.successToggle = true
              this.successMassage = response.data.status
              // console.log(response.data)
              setTimeout(() => {this.successToggle = false, this.successMassage = '' ,this.getCompaniesAndRolesList(),this.getData()}, 3000);
            }
          }
          // this.resetModal()
        })
    },
    resetLogfoModal() {
      this.logModal.title = ''
      this.logModal.content = ''
      this.logdata = ''
    },
    editUser(){
      this.alertAddModalToggle = false;
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      };
        axios.put("${BASE_URL}/user",{
                userId: Number(this.UserId),
                name: this.Name,
                username: this.UserName,
                email: this.Email,
                active: this.Active
              }, {headers}
          ).then(response => {
          if (response.status === 200) {
            if (response.data.code !== 401 && response.data.code !== 200) {
              this.alertMassage = response.data.message;
              this.alertToggle = true ;
              setTimeout(() => {this.alertToggle = false, this.alertMassage = '' }, 5000);
            } else if (response.status === 200 && response.data.code === 401) {
              this.alertMassage = response.data.message
              this.alertToggle = true
              setTimeout(() => {this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
            }
            else {
              this.logdata = response.data;
              this.successToggle = true
              this.successMassage = response.data.status
              setTimeout(() => { this.successToggle = false, this.successMassage = '' ,this.resetOptions(),this.getData()}, 3000);
            }
          }
          this.resetModal()
        })
    },
    loadPage(pageNum) {
      this.currentPage = pageNum;
      this.getData()
    },
    rows() {
      return Number(this.TotalData)
    },

    // resetInfofoModal() {
    //     this.infoModal.title = ''
    //     this.infoModal.content = ''
    //     this.logdata = ''
    //   },
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

<template>
  <div>
    <div class = "flex">
      <div class="small-box search-box">Search: <strong></strong>
        <b-form-input class = "small-form" size = "sm" type="text" placeholder="Search" v-model="Query"></b-form-input>
      </div>
      <div class="small-box search-box">Date From: <strong></strong>
        <b-form-datepicker class = "small-form" size = "sm" id="example-datepicker" v-model="DateForm">
        </b-form-datepicker><p/>
      </div>
      <div class="small-box search-box">Date To: <strong></strong>
        <b-form-datepicker class = "small-form" size = "sm" id="datepicker2" v-model="DateTo" :date-disabled-fn="dateDisabled">
        </b-form-datepicker><p/>
      </div>
      <div class="small-box search-box">
        <b-button
         class = "button-search" variant = "primary" @click="getData" size = "sm" >Filter
        </b-button>
      </div>
    </div>

  <div style="overflow: scroll;">
  <div class="animated fadeIn">
    <!-- <div>{{this.currentDate | dateParse('YYYY-MM-DD') | dateFormat('MMMM D, YYYY') }}</div> -->

    <div class = "text-left">

      <template v-if="SuccessToogle">
        <b-row>
          <b-col sm="12">
            <b-alert show variant="success">{{SuccessMassage}}</b-alert>
          </b-col>
        </b-row>
      </template>
      <template v-if="AlertToggle">
        <b-row>
          <b-col sm="12">
            <b-alert show variant="danger">{{AlertMassage}}</b-alert>
          </b-col>
        </b-row>
      </template>

      <div>
      <b-table
        class="my-table"
        head-row-variant="transparent"
        :sticky-header="this.DynamicHeight"
        variant="success"
        show-empty
        responsive="sm"
        :items="dataUser"
        :fields="fields2"
        @row-clicked="employeeDetailModal" >

        <template v-slot:cell(resignDate2)="row">
          {{row.item.resignDate.substring(0,10)}}
        </template>



         <!-- <template v-slot:cell(Actions)="row">
           <b-button variant="warning" size="sm" @click="editModal(row.item.id, row.item.name, row.item.email, row.item.phoneNumber, row.item.nikKaryawan)" class="mr-1">
             EDIT
           </b-button>
           <p/>
          <b-button variant="danger" size="sm" @click="deleteConfirm(row.item.Company)" class="mr-1">
            DELETE
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

      </div>

             <b-button variant="primary" size="sm" @click="addEmployeeModal()" class="mr-1">
               Add New Employee
             </b-button>

             <b-modal id="deleteModal"
                     v-model="deleteModal"
                     ok-title="Ok" @ok="deleteEmployee()">
                     Do you Really want to Delete this Company from List ?
             </b-modal>
             <b-modal id="AddEmployeeModal" title="Add New Employee"
                     v-model="AddEmployeeModal"
                     ok-title="Ok" @ok="addEmployee()" @cancel="resetModal('company')"
                     >
                     <template v-if="AlertinAddToggle">
                       <b-row>
                         <b-col sm="12">
                           <b-alert show variant="danger">{{AlertMassage}}</b-alert>
                         </b-col>
                       </b-row>
                     </template>

                     <b> Company : </b>
                     <b v-if="SessionRole !=='System Administrator'"> {{this.Company}} </b>
                     <b-form-select v-if="SessionRole==='System Administrator'" v-model="companySelected" :options="companyOptions"></b-form-select>


                     <div class="mt-3">Nama: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="Nama"
                             v-model="Name"></b-form-input><p/>
                     <div class="mt-3">Email: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="Email"
                             v-model="Email"></b-form-input><p/>
                     <div class="mt-3">Phone Number: <strong></strong></div>
                     <b-form-input type="number"
                             placeholder="Phone Number"
                             v-model="PhoneNumber"></b-form-input><p/>
                     <div class="mt-3">ID Karyawan: <strong></strong></div>
                     <b-form-input type="text"
                             placeholder="ID Karyawan"
                             v-model="NikKaryawan"></b-form-input><p/>
                     <div class="mt-3">No KTP: <strong></strong></div>
                     <b-form-input type="number"
                             placeholder="No KTP"
                             v-model="NoKtp"></b-form-input><p/>
                     <!-- <div class="mt-3">Status: <strong></strong></div>
                     <b-form-select v-model="Active" :options="ActiveOptions"></b-form-select> -->
             </b-modal>
             <b-modal id="editEmployeeModal"
                     v-model="EditModalToogle"
                     size = "lg"
                     ok-title="Ok" @ok="editEmployee" @cancel="resetModal()">
                     <b> Company : {{DataDetail.companyName}}</b><p/>
                     <template v-if="AlertinEditToggle">
                       <b-row>
                         <b-col sm="12">
                           <b-alert show variant="danger">{{AlertMassage}}</b-alert>
                         </b-col>
                       </b-row>
                     </template>

                     <div class = "flex">
                     <div class="small-box">ID Karyawan: <strong></strong>
                     <b-form-input class = "small-form"
                             type="text"
                             disabled
                             v-model="this.NikKaryawan"></b-form-input><p/></div>
                     <div class="small-box">Nama: <strong></strong>
                     <b-form-input class = "small-form"
                             type="text"
                             v-model="Name"></b-form-input></div>
                     </div>

                     <div class = "flex">
                     <div class="small-box">Email: <strong></strong>
                     <b-form-input class = "small-form"
                             type="text"
                             disabled
                             v-model="this.Email"></b-form-input><p/></div>
                     <div class="small-box">Registered OttoSG: <strong></strong>
                     <b-form-input class = "small-form"
                             type="text"
                             disabled
                             v-model="RegisterOttoSGFlag"></b-form-input><p/></div>
                     </div>

                     <div class = "flex">
                     <div class = "small-box"> No HP Registered: <strong></strong>
                     <div class = "flex">
                     <b-form-input class = "ottosg_phone_box"
                             type="text"
                             disabled
                             placeholder="-"
                             v-model="PhoneNumberRegister"></b-form-input><p/>
                     <b-button v-b-popover.hover.top="'No hp yang terdaftar akan di reset, anda yakin ingin mereset?'"
                      class = "button-reset" variant = "primary" @click="openResetModal" >Reset
                     </b-button>
                   </div></div>
                     <div class="small-box">No HP from HR : <strong></strong>
                     <b-form-input class = "small-form"
                             type="text"
                             placeholder="Phone Number"
                             v-model="PhoneNumber"></b-form-input><p/></div>
                     </div>

                     <div class = "flex">
                     <div class="small-box">No KTP: <strong></strong>
                     <b-form-input class = "small-form"
                             type="text"
                             v-model="NoKtp"></b-form-input><p/></div>
                     </div>

                     <div class = "flex">
                     <div class="small-box">Status: <strong></strong>
                       <b-form-select class = "small-form" v-model="Active" :options="ActiveOptions">
                         <template #first>
                          <b-form-select-option :value="null" disabled>-- Please select an option --</b-form-select-option>
                        </template>
                       </b-form-select>
                     </div>
                     <div v-if="this.Active == 'resign'" class="small-box">Resign Date: <strong></strong>
                       <b-form-datepicker class = "small-form" id="datepicker-resignDate" v-model="ResignDate" :date-disabled-fn="dateDisabled" locale="en">
                       </b-form-datepicker>
                     </div>
                     </div>

                     <!-- <div class="small-box">Status: <strong></strong></div>
                     <b-form-select v-model="Active" :options="ActiveOptions">
                       <template #first>
                        <b-form-select-option :value="null" disabled>- Please select an option -</b-form-select-option>
                      </template>
                     </b-form-select> -->
             </b-modal>


             <b-modal id="EmployeeDetail" size = "lg" v-model="DetailToggle" ok-title="Ok">
               <template v-if="errorAlertinDetail">
                 <b-row>
                   <b-col sm="12">
                     <b-alert show variant="danger">{{errorMessage}}</b-alert>
                   </b-col>
                 </b-row>
               </template>
               <img class="logo-mini" src="@/assets/images/avatar-3.jpg" alt="logo"
               height="100px"
               width="100px"/></p>
               <b-button variant="outline-primary" size="sm" @click="bridgingEdit(DataDetail)" class="mr-1">
                 Edit this Employee
               </b-button>

               <div class = "flex">
               <div class="small-box">Nama: <strong></strong>
                 <div class="small-form"> <pre> {{DataDetail.name}}</pre> </div>
               </div>
               <div class="small-box">No HP: <strong></strong>
                 <div class="small-form"><pre> {{DataDetail.phoneNumber}}</pre> </div>
               </div>
               </div>

               <div class = "flex">
               <div class="small-box">No KTP: <strong></strong>
                 <div class="small-form"> <pre> {{DataDetail.noKtp}}</pre> </div>
               </div>
               <div class="small-box">Status: <strong></strong>
                 <div class="small-form"><pre> {{DataDetail.status}}</pre> </div>
               </div>
               </div>

               <div class = "flex">
               <div class="small-box">Created Date: <strong></strong>
                 <div class="small-form"> <pre> {{DataDetail.createdDateString}}</pre> </div>
               </div>
               <div class="small-box">Updated Date:<strong></strong>
                 <div class="small-form"><pre> {{DataDetail.updatedDateString}}</pre> </div>
               </div>
               </div>

               <div class = "flex">
               <div class="small-box">Created By :  <strong></strong>
                 <div class="small-form"> <pre> {{DataDetail.createdBy}}</pre> </div>
               </div>
               <div class="small-box">Updated By: <strong></strong>
                 <div class="small-form"><pre> {{DataDetail.updatedBy}}</pre> </div>
               </div>
               </div>
               <div class = "flex">
               <div class="small-box">Registered OttoSG :  <strong></strong>
                 <div class="small-form"> <pre> {{DataDetail.RegisterOttoSGFlag}}</pre> </div>
               </div>
               </div>



               <!-- <div class="mt-3">Created By : <strong></strong></div>
               <div class="mt-3"><pre> {{DataDetail.createdBy}}<strong></strong></pre> </div>
               <div class="mt-3">Updated By: <strong></strong></div>
               <div class="mt-3"><pre> {{DataDetail.updatedBy}}<strong></strong></pre> </div> -->

             </b-modal>

             <b-modal id="editStatusModal"
                     v-model="EditStatusModalToogle"
                     ok-title="Ok" @ok="editEmployee" @cancel="resetModal()"
                     @hidden="resetModal()">
                     <div class="mt-3">Status: <strong></strong></div>
                     <b-form-select v-model="Active" :options="ActiveOptions"></b-form-select>
             </b-modal>

             <b-modal id="ottoSGResetModal"
                     v-model="OttoSGResetModalToogle"
                     ok-title="Ok" @ok="resetLinking"
                     >Anda yakin ingin me-reset no HP karyawan ini ?
             </b-modal>

    </div>

</div>
</div>
</div>
</template>

<script>
import axios from "axios";
import {getToken, getUserDataInSession2, BASE_URL} from '../../utils';
export default {
  data: () => ({
    dataUser: [],
    DataDetail: {},
    totalUser: '',
    SessionRole : '',
    fields3 :[
      { key: 'nikKaryawan', label: 'ID Karyawan' },
      { key: 'name', label: 'Name'},
      { key: 'email', label: 'Email' },
      { key: 'phoneNumber', label: 'Phone Number' },
      { key: 'noKtp', label: 'KTP' },
      { key: 'status', label: 'Status' },
      { key: 'Actions', label: 'Actions' },
      { key: 'createdDate', label: 'Created Date' },
      { key: 'updatedDate', label: 'Update Date' },
      { key: 'createdBy', label: 'Created By' },
      { key: 'updatedBy', label: 'Updated By' },
      { key: 'Actions', label: 'Actions' },
      { key: 'Actions2', label: 'Actions3' },
      { key: 'Actions3', label: 'Actions4' },
      { key: 'Actions4', label: 'Actions5' }
    ],
    fields2 :[
      { key: 'nikKaryawan', label: 'ID Karyawan' , tdClass: 'columnClass'},
      { key: 'name', label: 'Nama', tdClass: 'columnClass'},
      { key: 'email', label: 'Email' , tdClass: 'columnClass'},
      { key: 'companyName', label: 'Company' , tdClass: 'columnClass'},
      { key: 'phoneNumber', label: 'No HP (From HR)' , tdClass: 'columnClass'},
      { key: 'phoneNumberRegister', label: 'No HP Registered' , tdClass: 'columnClass'},
      { key: 'status', label: 'Status' , tdClass: 'columnClass'},
      // { key: 'resignDate', label: 'Resign Date' , tdClass: 'columnClass'},
      { key: 'resignDate2', label: 'Resign Date' , tdClass: 'columnClass'},

    ],
    ActiveOptions : [
      // {value: null, text: 'Please Select Status ', disabled: true },
      {value: 'active', text: 'Active'},
      {value: 'block', text: 'Block'},
      {value: 'resign', text: 'Resign'}
    ],
    companyOptions : [
      { value: null, text: 'Please Select Company ', disabled: true  },
    ],
    keyValueOption : {},
    companySelected :'',
    Id :'',
    UserName: '',
    Name:'',
    Email:'',
    Role: '',
    NoKtp: '',
    NikKaryawan : '',
    Company: '',
    PhoneNumber: '',
    PhoneNumberRegister : '',
    Password:'',
    RegisterOttoSGFlag : '',
    ResignDate : null,
    Active : null,
    deleteModal : false,
    OttoSGResetModalToogle : false,
    AddEmployeeModal : false,
    EditModalToogle : false,
    DetailToggle : false,
    EditStatusModalToogle: false,
    errorAlertinDetail: false,
    AlertToggle: false,
    AlertinAddToggle: false,
    AlertinEditToggle : false,
    SuccessToogle : false,
    SuccessMassage : '',
    AlertMassage: '',
    DateForm : '',
    DateTo : '',
    Company : '',
    CompanyId: '',
    currentDate : new Date(),
    token : '',
    Query: '',
    userInfo : [],
    perPage : 10,
    currentPage : 1,
    TotalData : 0,
    // CompanyId: 0,
    DynamicHeight : '',
    HomeLocation : '/ottosg_webportal'
  }),
  mounted() {
    this.CompanyId = getUserDataInSession2('userCompanyId')
    if (getUserDataInSession2('userCompanyName') != null && getUserDataInSession2('userRole') != null){
      this.Company = getUserDataInSession2('userCompanyName').replace(/\"/gi, '')
      this.SessionRole = getUserDataInSession2('userRole').replace(/\"/gi, '')
    }
    this.getData()
    this.getCompaniesList()
    this.dynamicHeight()
    // console.log(this.SessionRole)
  },
  created() {
  window.addEventListener("resize", this.myEventHandler);
  },
  destroyed() {
    window.removeEventListener("resize", this.myEventHandler);
  },
  methods: {
    getData() {
      this.userInfo = getUserDataInSession2("UserInfo")
      this.token = getToken()
      // console.log("Bearer " + this.token)
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      // axios.get(`${BASE_URL}/employees?page=${this.currentPage}&size=${this.perPage}&company-id=${this.CompanyId}&name=${this.Query}&start-date=${this.DateForm}&finish-date=${this.DateTo}&nik=${this.Query}&phone-number=${this.Query}`,
      axios.get(`${BASE_URL}/employees?page=${this.currentPage}&size=${this.perPage}&company-id=${this.CompanyId}&name=${this.Query}&start-date=${this.DateForm}&finish-date=${this.DateTo}&nik=${this.Query}&phone-number=${this.Query}&phone-number-register=${this.Query}&status=${this.Query}&company-name=${this.Query}&resign-date=${this.Query}&email=${this.Query}`,
        {headers},
        ).then(response => {
          if (response.status === 200 && response.data.code === 200) {
            this.TotalData = Number(response.data.data.total)
            this.dataUser = response.data.data.user;
          } else if (response.status === 200 && response.data.code === 401) {
            this.AlertToggle = true;
            this.AlertMassage = response.data.message;
            setTimeout(() => {this.AlertToggle = false ,this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
          } else {
            this.AlertMassage = "Server is Down"
            this.AlertToggle = true
          }
        })
    },
    getCompaniesList(){
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
    },
    myEventHandler(e) {
      // your code for handling resize...
      this.dynamicHeight()
    },
    dynamicHeight(){
      var TableHeight = window.innerHeight - 270
      this.DynamicHeight =  TableHeight.toString().concat("px")
    },
    deleteConfirm(compName){
      this.deleteModal = true
      this.compNameToDelete = compName
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
    bridgingEdit(DataDetail){
      if (DataDetail.status.toLowerCase() === 'block'){
        this.errorAlertinDetail = true
        this.errorMessage = 'Karyawan sudah diblock'
        setTimeout(() => {this.errorAlertinDetail = false, this.errorMessage = ''}, 3000)
      } else if (DataDetail.status.toLowerCase() === 'resign') {
        this.errorAlertinDetail = true
        this.errorMessage = 'Karyawan sudah resign'
        setTimeout(() => {this.errorAlertinDetail = false, this.errorMessage = ''}, 3000)
      } else {
        this.editModal(DataDetail)
      }
    },
    editModal(DataDetail){
      // console.log(DataDetail)
      this.DetailToggle = false
      this.EditModalToogle = true
      this.Id = DataDetail.id
      this.Name = DataDetail.name
      this.Email = DataDetail.email
      this.PhoneNumber = DataDetail.phoneNumber
      this.PhoneNumberRegister = DataDetail.phoneNumberRegister
      this.NikKaryawan = DataDetail.nikKaryawan
      this.NoKtp = DataDetail.noKtp
      this.companyId = DataDetail.idCompany
      // if (DataDetail.PhoneNumberRegister === "" || DataDetail.PhoneNumberRegister === undefined) {
      //   this.RegisterOttoSGFlag = "False"
      // } else {
      //   this.RegisterOttoSGFlag = "True"
      // }
      // console.log("companyid : " + this.companyId)
    },
    resetModal(exception){
      this.Id = ''
      this.Name = ''
      this.UserName = ''
      this.Email = ''
      this.PhoneNumber = ''
      this.NikKaryawan = ''
      this.NoKtp = ''
      this.Active = null
      if (exception !== 'company') {
        this.Company = ""
        this.companySelected = ''
      }
    },
    addEmployeeModal(){
      this.AddEmployeeModal = true
      if (this.SessionRole==='System Administrator'){
        this.companySelected = ''
      }
    },
    employeeDetailModal(Data){
      this.DetailToggle = true
      this.DataDetail = Data

      if (this.DataDetail.updatedDate !== undefined) {
        this.DataDetail.updatedDateString = String(Data.updatedDate).substring(0,10)
      }
      if (this.DataDetail.updatedDate === null) {
        this.DataDetail.updatedDateString = ''
      }
      if (this.DataDetail.createdDate !== undefined) {
        this.DataDetail.createdDateString = String(Data.createdDate).substring(0,10)
      }
      if (this.DataDetail.createdDate === null) {
        this.DataDetail.createdDateString = ''
      }
      console.log("DataDetail.phoneNumberRegister :" + Data.phoneNumberRegister)
      if (this.DataDetail.phoneNumberRegister == "") {
        this.DataDetail.RegisterOttoSGFlag = "False"
        this.RegisterOttoSGFlag = "False"
      } else {
        this.DataDetail.RegisterOttoSGFlag = "True"
        this.RegisterOttoSGFlag = "True"
      }
    },
    openResetModal(){
      this.OttoSGResetModalToogle = true
    },
    rows() {
      // console.log("ROWS: " +Number(this.TotalData/this.perPage).toFixed(0))
      return Number(this.TotalData)
    },
    loadPage(pageNum) {
      this.currentPage = pageNum;
      this.getData()
    },
    changeStatus(){
      this.EditStatusModalToogle = true
    },
    dateDisabled(ymd, date) {
        var currentDate = new Date().toJSON().slice(0,10).replace(/-/g,'-');
        var currentDate2 = new Date();

        return date > currentDate2
    },
    deleteEmployee(){

    },
    editEmployee(evt){
      // console.log("ACTIVE: " + this.Active)
      // console.log(this.PhoneNumber + this.DataDetail.phoneNumber + this.Active + this.DataDetail.status)
      this.token = getToken();
      let company_now = Number(getUserDataInSession2("userCompanyId"));
      if (company_now != 0) {
        this.companyId = getUserDataInSession2("userCompanyId");
      }
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      if (this.Active === null) {
        // console.log("kesini")
        evt.preventDefault()
        this.EditModalToogle = true
        this.AlertMassage = "Status belum diubah"
        this.AlertinEditToggle = true
        setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
      } else if (this.PhoneNumber === null || this.PhoneNumber === ""){
        evt.preventDefault()
        this.EditModalToogle = true
        this.AlertMassage = "No HP tidak Boleh Kosong"
        this.AlertinEditToggle = true
        setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
      } else if (this.PhoneNumber === this.DataDetail.phoneNumber && this.Active === this.DataDetail.status && this.Name == this.DataDetail.name && this.NoKtp == this.DataDetail.noKtp ) {
        console.log("1")
        console.log(this.Name)
        console.log(this.DataDetail.name)
        evt.preventDefault()
        this.AlertMassage = "Tidak ada data yang diubah"
        this.AlertinEditToggle = true
        setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
      } else {
        console.log("2")
        axios.put(`${BASE_URL}/employee`,{
          id : Number(this.Id),
          name: this.Name,
          email: this.Email,
          identityId : this.NoKtp,
          employeeId: this.NikKaryawan,
          msisdn: this.PhoneNumber,
          resignDate: this.ResignDate,
          status: this.Active,
          companyId: Number(this.companyId)
              }, {headers}
            // }
          ).then(response => {
            if (response.status === 200 && response.data.code === 200) {
              this.SuccessMassage = response.data.status
              this.SuccessToogle = true
              setTimeout(() => {this.SuccessToogle = false, this.SuccessMassage = ''}, 5000)
              this.resetModal()
              this.getData()
            } else if (response.status === 200 && response.data.code === 401) {
              this.EditModalToogle = true
              this.AlertinEditToggle = true;
              this.AlertMassage = response.data.message;
              setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
            } else if (response.status === 200 && response.data.code !== 200 ){
              this.EditModalToogle = true
              this.AlertMassage = response.data.message
              this.AlertinEditToggle = true
              setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
            } else {
              this.EditModalToogle = true
              this.AlertMassage = response.data.message
              this.AlertinEditToggle = true
              setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
            }
          }).catch((error) => {
            if (error.response !== undefined) {
              this.AlertMassage = error.response
              this.AlertinEditToggle = true
              setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
            } else {
              this.AlertMassage = "Server is down"
              this.AlertinEditToggle = true
              setTimeout(() => {this.AlertinEditToggle = false, this.AlertMassage = ''}, 5000)
            }
          });
        }
      },
    resetLinking(){
      // console.log(this.Name + this.PhoneNumber + this.Active + this.companyId)
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      axios.post(`${BASE_URL}/reset`,{
        id: this.Id,
        phoneNumber: this.PhoneNumber,
        status: "active",
        companyId: Number(this.companyId),
        resignDate: "",
        phoneNumberRegister: this.PhoneNumber
          }, {headers}
          // }
        ).then(response => {
          if (response.status === 200 && response.data.code === 200) {
            this.EditModalToogle = false ;
            this.SuccessMassage = response.data.status
            this.SuccessToogle = true
            setTimeout(() => {this.SuccessToogle = false, this.SuccessMassage = ''}, 5000)
            this.getData()
          } else if (response.status === 200 && response.data.code === 401) {
            this.errorAlertinDetail = true;
            this.DetailToggle = true ;
            this.EditModalToogle = false ;
            this.AlertMassage = response.data.message;
            setTimeout(() => {this.errorAlertinDetail = false, this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
          } else if (response.status === 200 && response.data.code !== 200 && response.data.code !== 401 ){
            // console.log("kesini")
            // this.AddEmployeeModal = true
            this.EditModalToogle = false ;
            this.DetailToggle = true ;
            this.errorMessage = response.data.message
            this.errorAlertinDetail = true
            setTimeout(() => {this.errorAlertinDetail = false, this.errorMessage = ''}, 5000)
          } else {
            this.EditModalToogle = false ;
            this.DetailToggle = true ;
            this.AddEmployeeModal = true
            this.errorMessage = response.data.message
            this.errorAlertinDetail = true
            setTimeout(() => {this.errorAlertinDetail = false, this.errorMessage = ''}, 5000)
          }
        }).catch((error) => {
          if (error.response !== undefined) {
            this.EditModalToogle = false ;
            this.DetailToggle = true ;
            this.errorMessage = error.response
            this.errorAlertinDetail = true
            setTimeout(() => {this.errorAlertinDetail = false, this.errorMessage = ''}, 5000)
          } else {
            this.EditModalToogle = false ;
            this.DetailToggle = true ;
            this.errorMessage = "Server is down"
            this.errorAlertinDetail = true
            setTimeout(() => {this.errorAlertinDetail = false, this.errorMessage = ''}, 5000)
          }
        });
    },
    addEmployee(){
      this.token = getToken();
      if (this.SessionRole !== 'System Administrator') {
        this.companyId = getUserDataInSession2("userCompanyId");
      } else {
        this.companyId =this.companySelected
      }
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      axios.post(`${BASE_URL}/employee`,{
        name: this.Name,
        email: this.Email,
        identityId : this.NoKtp,
        employeeId: this.NikKaryawan,
        msisdn: this.PhoneNumber,
        companyId: Number(this.companyId)
            }, {headers}
          // }
        ).then(response => {
          if (response.status === 200 && response.data.code === 200) {
            console.log("SUCCESS")
            this.SuccessMassage = response.data.status
            this.SuccessToogle = true
            setTimeout(() => {this.SuccessToogle = false, this.SuccessMassage = ''}, 5000)
            this.resetModal('company')
            this.getData()
          } else if (response.status === 200 && response.data.code === 401) {
            this.AddEmployeeModal = true
            this.AlertMassage = response.data.message
            this.AlertinAddToggle = true
            setTimeout(() => {this.AlertinAddToggle = false ,this.AlertMassage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
          } else if (response.status === 200 && response.data.code !== 200 ){
            console.log("HERE")
            this.AddEmployeeModal = true
            this.AlertMassage = response.data.message
            this.AlertinAddToggle = true
            setTimeout(() => {this.AlertinAddToggle = false, this.AlertMassage = ''}, 5000)
          } else {
            this.AddEmployeeModal = true
            this.AlertMassage = response.data.message
            this.AlertinAddToggle = true
            setTimeout(() => {this.AlertinAddToggle = false, this.AlertMassage = ''}, 5000)
          }
        }).catch((error) => {
          if (error.response !== undefined) {
            this.AlertMassage = error.response
            this.AlertinAddToggle = true
            setTimeout(() => {this.AlertinAddToggle = false, this.AlertMassage = ''}, 5000)
          } else {
            this.AlertMassage = "Server is down"
            this.AlertinAddToggle = true
            setTimeout(() => {this.AlertinAddToggle = false, this.AlertMassage = ''}, 5000)
          }
        });
      }
    }
  };

</script>

<style scoped>

.search-box{
  font-size: 14px;
  /* height: 8 */
}
.button-reset {
  /* height: 38px; */
  margin-left: 4.5%;
  margin-right: 8px;
  right : 10
  /* width: 20%; */
}
.button-search {
  margin-left: 4%;
  margin-top: 7%;
  right : 10;
  height: 48%;
  width : 50%;
}
.flex {
  display: flex;
}
.small-box {
  margin-top: 5px;
  width: 50%;
}
.ottosg_phone_box {
  /* margin-top: 20px; */
  width: 73%;
}
.small-form {
  width : 95%;
}
.smaller-form {
  width : 60%;
}

.my-table {
  cursor: pointer;
  position: sticky;
  font-size: 14px;
}
/deep/ .my-table th {
  position: sticky;

  background-color: #007bff;
  color: white;
  font-size:  14px;
  font-weight: 500;
}
/deep/ .my-table > tbody > tr > td {
max-width: 30px;
}

.columnClass {
   max-width: 10000px;
}
</style>

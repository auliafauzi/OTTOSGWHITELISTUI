<template>
  <b-card>
    <template v-if="success">
      <b-row>
        <b-col sm="12">
          <b-alert show variant="success">{{successMessage}}</b-alert>
        </b-col>
      </b-row>
    </template>
    <template v-if="error">
      <b-row v-if="errorMessage instanceof Array">
        <b-col sm="12" v-for="(item, index) in errorMessage" v-bind:data="item" v-bind:key="index">
          <b-alert show variant="danger">{{item}}</b-alert>
        </b-col>
      </b-row>
      <b-row v-else>
        <b-col sm="12">
          <b-alert show variant="danger">{{errorMessage}}</b-alert>
        </b-col>
      </b-row>
    </template>


     <div>
       <p/>
         <div id="UploadBox"
                 ref="modal"
                 ok-title="Submit" @ok="uploadFile"
                 title="Upload file">
                 <template v-if="errorUploadFile">
                   <b-row>
                     <b-col sm="12">
                       <b-alert show variant="danger">{{errorMessage}}</b-alert>
                     </b-col>
                   </b-row>
                 </template>
                 <template v-if="successUploadFile">
                   <b-row>
                     <b-col sm="12">
                       <b-alert show variant="success">{{successMessage}}</b-alert>
                     </b-col>
                   </b-row>
                 </template>
                   <div class="container">
                     <div class="large-12 medium-12 small-12 cell">
                       <div>Company: <strong></strong></div>
                       <div class="mt-3" v-if="SuperAdmin===false" >{{CompanyName}}<strong></strong></div>
                       <b-form-select v-if="SuperAdmin" v-model="Company" :options="CompanyOptions"></b-form-select>
                      <p></p>
                       <label v-if="Company !== ''">
                         <input type="file" id="file" accept=".csv" ref="file" v-on:change="handleFileUpload()"/>
                       </label>
                     </div>
                   </div>
                   <b-btn v-if="Company !== ''" @click="uploadFile" size="m" variant="primary">
                     <i class="fa fa-dot-circle-o"></i>
                      Submit
                    </b-btn>
         </div>

    </div>
  </b-card>
</template>

<script>
// import { ModelSelect } from 'vue-search-select';
import { getToken,getUserDataInSession2, BASE_URL } from '../../utils';
import axios from "axios";

export default {
  name: 'UploadBatch',
  // components: {
  //   ModelSelect,
  // },
  data: () => (
    {
      success: false,
      error: false,
      errorMessage: '',
      successMessage: '',
      warningMessage: '',
      password: '',
      file: '',
      warning: false,
      warning2: false,
      successUploadFile: false,
      errorUploadFile: false,
      SuperAdmin : false,
      Company : '',
      CompanyName : '',
      CompanyOptions : [
        { value: null, text: 'Please Select Company ', disabled: true  },
      ],
      keyValueOption : {},
      HomeLocation : '/ottosg_webportal'

    }
  ),
  mounted() {
    this.getData();
    if (localStorage.getItem('userRole') === '"System Administrator"') {
			this.SuperAdmin = true;
      // this.getData();
		}
		else {
      if (getUserDataInSession2('userCompanyName') !== null) {
        this.CompanyName = getUserDataInSession2('userCompanyName').replace(/\"/gi, '')
      }
      this.Company = getUserDataInSession2('userCompanyId')
		};
  },
  methods: {
    getData() {
      this.token = getToken()
      var headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
      axios.get(`${BASE_URL}/companies`,{
            headers
            },
        ).then(response => {
          if (response.status === 200 && response.data.code === 200) {
            this.dataCompany = response.data.data;
            this.optionsFunc(this.dataCompany,this.CompanyOptions)
          } else if (response.status === 200 && response.data.code === 401) {
            this.error = true;
            this.errorMessage = response.data.message;
            setTimeout(() => {this.errorMessage = '', console.log(response.data.message),window.location.href = this.HomeLocation}, 2000);
          } else {
            // this.errorMessage = response.data.message
            // this.error = true
          }
        });
    },
    uploadFile(evt) {
      if (this.file === '' || this.file === undefined) {
        evt.preventDefault();
        // this.$refs.modalUploadFile.show();
        this.errorMessage = 'Mohon pilih file yang akan diupload';
        this.errorUploadFile = true;
        setTimeout(() => {this.errorUploadFile = false, this.file = ''}, 5000);
      } else if (this.Company === '') {
        evt.preventDefault();
        // this.$refs.modalUploadFile.show();
        this.errorMessage = 'Pilih Nama Company';
        this.errorUploadFile = true;
        setTimeout(() => {this.errorUploadFile = false, this.file = ''}, 5000);
      } else {
        this.errorUploadFile = false;
        this.submitFile();
        // this.handleUploadFile();
        // this.$refs.modalUploadFile.hide();
      }
    },
    handleUploadFile() {
      this.submitFile();
    },
    submitFile() {
      this.token = getToken()
      const formData = new FormData();
      formData.append('file', this.file);
      formData.append('company-id', this.Company);
      // for (var pair of formData.entries()) {
      //   console.log(pair[0]+ ', ' + pair[1]);
      // }
      var headers = {
        'Content-Type': 'multipart/form-data',
        'Authorization': `Bearer ${this.token}`
      };
      axios.post(`${BASE_URL}/employees`,
      // axios.post(`http://0.0.0.0:3000/api/sg-web-service/v1/employees`,
      // formData, { headers: { 'Content-Type': 'multipart/form-data', 'x-access-token': getToken() } }).then((response) => {
      formData,{ headers}).then((response) => {
        // console.log(response.data.result)
        if (response.status === 200) {
          if (response.data.status === "Success") {
            // this.$refs.file.files[0] = ''
            // this.clearFiles()
            // this.successMessage = 'File Upload Has Been Completed ';
            this.successMessage = response.data.status + response.data.message
            this.successUploadFile = true;
            setTimeout(() => {this.successUploadFile = false, this.file = '', location.reload()}, 3000);
          } else if (response.data.code === 400) {
            this.errorMessage = response.data.message
            this.errorUploadFile = true;
            setTimeout(() => {this.errorUploadFile = false, this.file = '', location.reload()}, 2000);
          } else if (response.data.code === 401) {
            this.error = true;
            this.errorMessage = response.data.message;
            setTimeout(() => {this.error = false, window.location.href = this.HomeLocation}, 2000);
          } else if (response.data.status === false) {
            this.errorMessage = response.data.status  + response.data.message
            this.errorUploadFile = true;
            setTimeout(() => {this.errorUploadFile = false, this.file = ''}, 5000);
          } else if (response.data.status === 'BadRequest') {
            this.errorMessage = response.data.status + ' - ' + response.data.message
            this.errorUploadFile = true;
            setTimeout(() => {this.errorUploadFile = false, this.file = ''}, 5000);
          } else {
            this.errorMessage = 'Internal Server Error';
            this.errorUploadFile = true;
            setTimeout(() => {this.errorUploadFile = false, this.file = ''}, 5000);
          }
        } else {
          this.errorMessage = 'Internal Server Error';
          this.errorUploadFile = true;
          setTimeout(() => {this.errorUploadFile = false, this.file = ''}, 5000);
        }
      }).catch((error) => {
        if (error.response !== undefined) {
          this.success = false;
          // setAlert(this, error.response);
        }
      });
    },

    handleFileUpload() {
      this.file = this.$refs.file.files[0];
      // console.log("this.file: " + this.file)
      // console.log("this.$refs.file.files[0]: " + this.$refs.file.files[0])
    },
    // clearFiles() {
    //   console.log("satu")
    //   // this.$refs.file.files.reset()
    //   console.log("dua")
    // },
    optionsFunc(data, target){
      for (let i= 0; i < data.length; i++) {
        this.keyValueOption["value"] = data[i]["id"]
        this.keyValueOption["text"] = data[i]["name"]
        target.push(this.keyValueOption) //append keyvalue obj to options array
        this.keyValueOption = {} //reset keyvalue Object
      }
      // console.log(target)
    },
  },
};

</script>

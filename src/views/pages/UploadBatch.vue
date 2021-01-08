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
         <div id="RestoreBox"
                 ref="modal"
                 ok-title="Submit" @ok="restore"
                 title="Upload the backup file">
                 <template v-if="errorRestore">
                   <b-row>
                     <b-col sm="12">
                       <b-alert show variant="warning">{{errorMessage}}</b-alert>
                     </b-col>
                   </b-row>
                 </template>
                 <template v-if="successRestore">
                   <b-row>
                     <b-col sm="12">
                       <b-alert show variant="success">{{successMessage}}</b-alert>
                     </b-col>
                   </b-row>
                 </template>
                   <div class="container">
                     <div class="large-12 medium-12 small-12 cell">
                      <p></p>
                       <label>File
                         <input type="file" id="file" accept=".csv,.gz,.zip" ref="file" v-on:change="handleFileUpload()"/>
                       </label>
                     </div>
                   </div>
                   <b-btn @click="restore" size="m" variant="primary">
                     <i class="fa fa-dot-circle-o"></i>
                      Submit
                    </b-btn>
         </div>

    </div>
  </b-card>
</template>

<script>
// import { ModelSelect } from 'vue-search-select';
// import { getToken, setAlert } from '../../utils';
import axios from "axios";

export default {
  name: 'BackupRestore',
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
      successRestore: false,
      errorRestore: false,
    }
  ),


  methods: {
    // download() {
    //   this.axios({
    //     url: '/dashboard/backup/download',
    //     method: 'GET',
    //     // headers: { 'x-access-token': getToken() },
    //     responseType: 'blob',
    //   }).then((response) => {
    //     const url = window.URL.createObjectURL(new Blob([response.data]));
    //     const link = document.createElement('a');
    //     link.href = url;
    //     link.setAttribute('download', 'nfg-backup.zip');
    //     document.body.appendChild(link);
    //     link.click();
    //     if (response.data.result === true || response.status === 200) {
    //       // setAlert(this, response);
    //       this.success = true;
    //       this.successMessage = 'Backup Has Been Completed';
    //       setTimeout(() => { this.success = false; this.successMessage = ''; }, 5500);
    //     } else if (response.data.result === false) {
    //       setAlert(this, response);
    //     } else {
    //       setAlert(this, response);
    //       this.errorMessage = 'Internal Server Error [Please check your root password]';
    //     }
    //   }).catch((error) => {
    //     if (error.response !== undefined) {
    //       this.success = false;
    //       setAlert(this, error.response);
    //     }
    //   });
    // },
    restore(evt) {
      if (this.file === '' || this.file === undefined) {
        evt.preventDefault();
        // this.$refs.modalRestore.show();
        this.errorMessage = 'Please select the file to upload';
        this.errorRestore = true;
        setTimeout(() => {console.log("sleeping"),this.errorRestore = false, this.file = ''}, 5000);
      }
      // else if (this.password === '') {
      //   evt.preventDefault();
      //   this.errorMessage = 'Please enter the root password';
      //   this.errorRestore = true;
      // }
      else {
        this.errorRestore = false;
        this.submitFile();
        // this.handleRestore();
        // this.$refs.modalRestore.hide();
      }
    },
    handleRestore() {
      this.submitFile();
    },
    submitFile2() {
      const formData = new FormData();
      formData.append('file', this.file);
      console.log(formData)
      // const posting = {
      //   root_password: this.password,
      //   formData,
      // };
      this.root_password = this.password;

      axios({
        method: "post",
        url: `http://0.0.0.0:3000/api/v1/uploadBatch/`,
        formData,
        headers: { 'Content-Type': 'multipart/form-data' },
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true
      })
    },
    async timesleep(seconds){
      console.log("timesleep is running")
      await sleep(seconds)
    },
    submitFile() {
      const formData = new FormData();
      formData.append('file', this.file);
      // const posting = {
      //   root_password: this.password,
      //   formData,
      // };
      this.root_password = this.password;
      axios.post(`http://0.0.0.0:3000/api/v1/uploadBatch/`,
      // this.axios.post(`dashboard/restore/${this.root_password}`,
      // formData, { headers: { 'Content-Type': 'multipart/form-data', 'x-access-token': getToken() } }).then((response) => {
      formData, { headers: { 'Content-Type': 'multipart/form-data' } }).then((response) => {
        console.log(response.data.result)
        if (response.data.result === true) {
          console.log("result is true")
          // setAlert(this, response);
          this.successMessage = 'File Upload Has Been Completed ';
          this.successRestore = true;
          setTimeout(() => {console.log("sleeping"),this.successRestore = false, this.file = ''}, 5000);
        } else if (response.data.result === false) {
          setAlert(this, response);
        } else {
          setAlert(this, response);
          this.errorMessage = 'Internal Server Error';
        }
      }).catch((error) => {
        if (error.response !== undefined) {
          this.success = false;
          setAlert(this, error.response);
        }
      });
    },

    handleFileUpload() {
      this.file = this.$refs.file.files[0];
    },
//
//  backup methods
    clearName() {
      this.password = '';
    },
    handleOk(evt) {
      // Prevent modal from closing
      if (!this.password) {
        evt.preventDefault();
        this.warningMessage = 'Please enter the root password';
        this.warning = true;
        // alert('Please enter the root password');
      } else {
        this.warning = false;
        this.handleSubmit();
      }
    },
    handleSubmit() {
      this.backup();
      // this.$refs.modalBackup.hide();
    },
  },
};

</script>

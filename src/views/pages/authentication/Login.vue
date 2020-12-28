<template>
	<vue-scroll class="login-page align-vertical">
		<div class="form-wrapper align-vertical-middle">
			<div class="form-box card-base card-shadow--extraLarge">
				<!-- <img class="image-logo" src="@/assets/images/logo.png" alt="logo"/> -->
				<!-- <div class="app-name">Partner Management</div> -->
				<img class="image-logo" src="@/assets/images/logo.png" alt="logo"/>
				<!-- <div class="app-name" @click="goto('/')">Partner Management</div> -->
				<div class="box-logo flex align-center">
					<!--<div class="letter-logo">P</div>-->
					<!-- <img class="image-logo" src="@/assets/images/logo.svg" alt="logo"/> -->
					<!-- <img class="image-logo" src="@/assets/images/logo.png" alt="logo"/> -->
					<div class="app-name" @click="goto('/')">Partner Management</div>
					<!-- <button class="collapse-nav" @click="collapseNavToggle">
						<i class="mdi mdi-menu"></i>
					</button> -->
				</div>

				<float-label class="styled">
					<input type="user name" placeholder="User Name">
				</float-label>
				<float-label class="styled">
					<input type="password" placeholder="Password">
				</float-label>

				<div class="flex">
					<div class="box grow"><el-checkbox>Remember Me </el-checkbox></div>
					<div class="box grow text-right"><router-link to="/dashboard">Forgot Password?</router-link></div>
				</div>

				<div class="flex text-center center pt-30 pb-10">
					<el-button plain size="small" @click="login" class="login-btn tada animated">
						LOGIN
					</el-button>
					<!-- <el-button @click="ErrorModalShow = !ErrorModalShow">Open Modal</el-button> -->
				</div>

				<!-- <div class="text-divider mv-10"></div> -->

				<!-- <div class="flex column center pt-10 pb-10">
					<el-button plain size="small" @click="login" class="social-btn google">
						Log in with Google
					</el-button>
					<el-button plain size="small" @click="login" class="social-btn facebook">
						Log in with Facebook
					</el-button>
				</div> -->

			 <b-modal v-model="modalShow" :body-bg-variant="modal_variants"
			 :header-bg-variant="modal_variants"
			 :footer-bg-variant="modal_variants"
			 :hide-header='hideHeader'
			 :footer-class='footerClass'
			 :cancel-disabled='cancelButtonDisable'
			 :hide-footer="true"
			 :body-text-variant="text_variants">{{errorText}}</b-modal>

			</div>
		</div>

		<div>

	 	</div>

	</vue-scroll>



</template>

<script>
import axios from "axios";
import { saveUserDataInSession2, getUserDataInSession2 } from '../../../utils';

export default {
	name: 'Login',
	data() {
		return {
			form: {
				email: null,
				password: null,
			},
			loginError: false,
	    expiryError: false,
	    serverError: false,
			modalShow: false,
			modal_variants: 'light',
			text_variants: 'danger',
			errorText: '',
			hideHeader: true,
			footerClass: 'p-2 border-top-0',
			cancelButtonDisable:true,
			freeText: '',
			localValue : {
				auth:'',
				role:''
			}
			// ErrorModal:{}
		}
	},
	methods: {
		loginold() {
			this.$store.commit('setLogin')
			this.$router.push('dashboard')
		},
		// showModal() {
		// 	this.ErrormodalShow = true
    // },
		login(logfilename) {
      axios({
        method: "post",
        url: `http://0.0.0.0:3000/api/v1/login`,
        // headers: {Authorization: `Bearer ${this.state.login.access_token}`},
        withCredentials: true // True otherwise I receive another error
      }).then(response => {
        if (response.status === 200) {
					// console.log(response.data)
					if (response.data.auth === 'allowed') {
						saveUserDataInSession2('userRole',response.data.role)
						this.localValue = getUserDataInSession2('userRole')
						// console.log(this.localValue)
						console.log(this.localValue)
						this.$router.push('/dashboard');
					}
					else {
						this.errorText = 'Invalid Username or Password'
						this.modalShow = true
					}
					// this.modalShow = true
					// showModal()
					// this.$root.$emit('bv::show::modal','Errormodal-1');
					// this.ErrormodalShow = true
					// this.$root.$emit('bv::show::modal', this.ErrorModal);
					// this.$router.push('dashboard');
          // this.logdata = response.data;
          // this.logfilename = logfilename;
          // this.$root.$emit('bv::show::modal', this.logModal.id)
				}
				else {
					console.log(response.data)
					this.errorText = response.status
					this.modalShow = true
					}
        }).catch((error) => {
	        if (error.response !== undefined) {
	          if (error.response.status === 401) {
							console.log(error.response)
	            // this.loginError = true;
	          } else {
							console.log(error.response)
	            // this.serverError = true;
	          }
	        } else {
						console.log(error.response)
						this.errorText = 'Server is Down'
						this.modalShow = true
	          // this.serverError = true;
	        }
	      });
    },
	}
}
</script>

<style lang="scss">
@import '../../../assets/scss/_variables';
@import '../../../assets/scss/_mixins';

.login-page {
	background: $text-color;
	margin-left: -30px;
	margin-right: -30px;

	.form-wrapper {
		width: 100%;
	}

	.form-box {
		width: 100%;
		max-width: 340px;
		padding: 30px;
		box-sizing: border-box;
		margin: 20px auto;

		a {
			font-size: 14px;
			color: $text-color-accent;
			text-decoration: none;
			font-weight: 500;
		}

		// .image-logo {
		// 	width: 80px;
		// 	margin: 0px auto;
		// 	margin-bottom: 30px;
		// 	display: block;
		// }

		.login-btn ,
		// .social-btn {
		// 	width: 160px;
		//
		// 	&.google {
		// 		margin-bottom: 10px;
		// 		background-color: #d73d32;
		// 		color: white;
		//
		// 		&:hover {
		// 			border-color: #d73d32;
		// 		}
		// 	}
		// 	&.facebook {
		// 		background-color: #3f5c9a;
		// 		color: white;
		//
		// 		&:hover {
		// 			border-color: #3f5c9a;
		// 		}
		// 	}
		// }

		.signin-box {
			font-size: 14px;
		}

		.image-logo {
			width: 40px;
			height: 80px;
			margin-right: 10px;
			// filter: drop-shadow(0px 2px 2px rgba(0, 0, 0, 0.3));
		}
		.box-logo {
			height: 40px;
			padding: 0 60px;
			box-sizing: border-box;
			font-size: 16px;
			font-weight: bold;
			position: relative;
			@include text-bordered-shadow();

			.letter-logo {
				width: 30px;
				height: 30px;
				line-height: 20px;
				text-align: center;
				background: $text-color;
				color: $text-color-accent;
				border-radius: 5px;
				margin-right: 10px;
				font-weight: bolder;
				font-size: 20px;
			}

			.image-logo {
				width: 40px;
				height: 40px;
				margin-right: 10px;
				filter: drop-shadow(0px 2px 2px rgba(0, 0, 0, 0.3));
			}

			.app-name {
				cursor: pointer;
			}
		}
	}
}

@media (max-width: 768px) {
	.layout-container .container .view.login-page {
		margin-left: -5px;
		margin-right: -5px;
		max-width: calc(100% + 10px);
	}
}
</style>

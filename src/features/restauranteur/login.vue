<template>
	<div class="loginForm">
		<div class="container">
				<div class="login-form">
					<div class="main-div mx-auto">
						<div class="panel">
							<h2>Restaurant Login</h2>
							<p>Please enter your email and password</p>
						</div>
						<div v-show="error" class="alert alert-warning alert-dismissible" role="alert">
							<strong>Warning! </strong>{{msg}}
						</div>
						<form v-on:submit.prevent="login" id="Login">
							<div class="form-group">
								<input v-model="email" type="email" class="form-control" id="inputEmail" placeholder="Email Address" required>
							</div>
							<div class="form-group">
								<input v-model="password" type="password" class="form-control" id="inputPassword" placeholder="Password" required>
							</div>
							<div class="forgot">
								<a href="">Forgot password?</a>
							</div>
							<button class="btn btn-lg btn-primary btn-block login-button" data-loading-text="Signing in..." type="submit">Login</button>
						</form>
					</div>
				</div>
		</div>
	</div>
</template>

<script>
	export default {
		name: "loginScreen",

		data() {
			return {
				email: '',
				password: '',
				error: false,
				msg: ''
			}
		},

		methods: {
			login() {
				if (!this.email || !this.password) {
					this.error = true;
					this.msg = "email and password fields can not be empty";
					return;
				}
				this.$http.post(
					process.env.VUE_APP_API_ROUTE + 'user/restaurant/login',
					{
						user: {
							email: this.email,
							password: this.password
						}
					}
				).then((response) => {
					if (response.ok) {
						this.$store.commit('restaurantOwnerLogin', response.data);
						this.$router.push('/restaurant');
						console.log('Login successful');
					}
				}).catch((e) => {
					this.error = true;
					this.msg = "The username and password combination is incorrect."
					console.log(e);
				});
			}
		}
	};

</script>

<style scoped>
.loginForm {
	background: #999;
	width:100%;
	height:100vh;
	padding-top: 10px;
}
.form-heading { color:#fff; font-size:23px;}
.panel h2{ color:#444444; font-size:18px; margin:0 0 8px 0;}
.panel p { color:#777777; font-size:14px; margin-bottom:30px; line-height:24px;}
.login-form .form-control {
  background: #f7ebd7 none repeat scroll 0 0;
  border: 1px solid #d4d4d4;
  border-radius: 4px;
  font-size: 14px;
  height: 50px;
  line-height: 50px;
}
.main-div {
  background: #ffffff none repeat scroll 0 0;
  border-radius: 2px;
  max-width: 600px;
  padding: 5% 10% 10% 10%;
}

.login-form .form-group {
  margin-bottom:10px;
}
.login-form{ text-align:center;}
.forgot a {
  color: #777777;
  font-size: 14px;
  text-decoration: underline;
}
.login-form  .btn.btn-primary {
  background: #f0ad4e none repeat scroll 0 0;
  border-color: #f0ad4e;
  color: #ffffff;
  font-size: 14px;
  width: 100%;
  height: 50px;
  line-height: 50px;
  padding: 0;
}
.forgot {
  text-align: left; margin-bottom:30px;
}
.botto-text {
  color: #ffffff;
  font-size: 14px;
  margin: auto;
}

</style>

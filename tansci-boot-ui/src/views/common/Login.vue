<script setup lang="ts">
	import {onBeforeMount,onMounted,reactive,ref,toRefs} from "vue"
	import type {FormInstance} from 'element-plus'
	import {useRouter} from 'vue-router'
	import {login,getCode} from '@/api/auth'
    import {HOME_URL} from '@/config/config'

	const router = useRouter()
	const loginFormRef = ref<FormInstance>() 
	
	const logo = new URL('../../assets/image/logo.png', import.meta.url).href
	const loginLogo = new URL('../../assets/image/login-left.png', import.meta.url).href
	const codeImg = ref()

	const state = reactive({
		loading: false,
		loginStyle: {
			height: '',
		},
		loginForm: {
			username: '',
			password: '',
			code: '',
			remember: 0
		},
	})

	onBeforeMount(() => {
		state.loginStyle.height = (document.body.clientHeight || document.documentElement.clientHeight) + "px"
	})

	onMounted(() =>{
		onCode()
	})

    function copyYear(){
		let date = new Date();
		return date.getFullYear();
	}

	function onCode(){
		getCode().then((res:any) =>{
			codeImg.value = 'data:image/png;base64,' + res.data.result
		})
	}

	async function onSubmit(formEl: FormInstance | undefined) {
		if (!formEl) return;
		await formEl.validate((valid)=>{
			if(valid){
				// 登录成功后设置token到缓存
				let param:any = {
					username: state.loginForm.username,
					password: state.loginForm.password,
					code: state.loginForm.code
				}
				state.loading = true;
				login(param).then((res:any) =>{
					if(res){
						state.loading = false;
						router.push({path: HOME_URL});
					}
				}).catch(()=>{
					onCode()
					state.loading = false;
				})
			}
		});
	}

</script>
<template>
	<div class="login-container" :style="state.loginStyle">
		<div class="login-header">
			<div>
				<el-image :src="logo" style="width: 100%; height: 100%;"></el-image>
			</div>
			<div>
				<span class="title">Tansci Boot</span>
			</div>
		</div>
		<div class="login-main">
			<div class="main-title">帐号登录</div>
			<div class="main-container">
				<div class="logo">
					<el-image :src="loginLogo" fit="fit"></el-image>
				</div>
				<div class="form">
					<el-form :model="state.loginForm" :rules="rules" size="large" ref="loginFormRef">
						<el-form-item prop="username" :rules="[
								{required: true,message: '请输入账号',trigger: 'blur'},
								{pattern: /^[a-zA-Z]\w{4,17}$/,message: '账号有误，请重新输入',trigger: 'blur'}]">
							<el-input v-model="state.loginForm.username" prefix-icon="Avatar" placeholder="请输入账号" style="width:100%"></el-input>
						</el-form-item>
						<el-form-item prop="password" :rules="[
								{required: true,message: '请输入密码',trigger: 'blur'}]">
							<el-input type="password" v-model="state.loginForm.password" prefix-icon="Lock" show-password placeholder="请输入密码" style="width:100%"></el-input>
						</el-form-item>
						<el-form-item prop="code" :rules="[{required: true,message: '请输入验证码',trigger: 'blur'}]">
							<el-input v-model="state.loginForm.code" prefix-icon="HelpFilled" placeholder="请输入验证码" style="width: calc(100% - 110px); margin-right: 6px;"></el-input>
							<el-image @click="onCode" :src="codeImg" fit="fill" style="width: 102px; height: 38px; border: 1px solid #dcdfe6; border-radius: 4px;"/>
						</el-form-item>
						<el-form-item>
							<el-checkbox v-model="state.loginForm.remember">记住密码</el-checkbox>
						</el-form-item>
						<el-form-item>
							<el-button type="primary" @click="onSubmit(loginFormRef)" :loading="loading" style="width:100%">登录</el-button>
						</el-form-item>
					</el-form>
				</div>
			</div>
		</div>
		<div class="login-footer">
			<div>
				<el-link href="https://typ1805.gitee.io" target="_blank">关于作者</el-link>
				<el-divider direction="vertical" />
				<el-text size="small">源码地址</el-text>
				<el-link href="https://gitee.com/typ1805/tansci-boot" target="_blank" style="padding:0 0.6rem">Gitee</el-link>
				<el-link href="https://github.com/typ1805/tansci-boot" target="_blank">GitHub</el-link>
				<el-divider direction="vertical" />
				<el-link href="https://typ1805.gitee.io" target="_blank">联系作者</el-link>
			</div>
			<div class="copy">
				&copy; {{copyYear()}} Tansci Boot 版权所有
			</div>
		</div>
	</div>
</template>
<style lang="scss" scoped="scoped">
	.login-container {
		// background-image: radial-gradient( white 0%, #FAFDFE 10%, #ddf8e7 50%, #FAFDFE 90%, white 100%);
		// background-image: radial-gradient(#ddf8e7 00%, #FAFDFE 80%, white 100%);
		.login-header{
			width: 100%;
			height: 5rem;
			line-height: 5rem;
			display: flex;
			padding: 0 20%;
			.title{
				padding: 0 1rem;
				color: var(--t);
				font-size: 20px;
				font-weight: 700;
			}
		}
		.login-main{
			height: 80%;
			.main-title{
				font-size: 32px;
				text-align: center;
				padding: 5rem 0;
				color: var(--t);
			}
			.main-container{
				display: flex;
				flex-wrap: wrap;
				justify-content: center;
				align-items: center;
				.logo{
					width: 26rem;
					padding-right: 2rem;
					// transition: all .2s;
				}
				// .logo:hover{
				// 	transform: scaleY(1.1) scaleX(1.1) translateZ(0);
				// }
				.form{
					width: 26rem;
					padding-left: 4rem;
					border-left: 1px solid #c7f6da;
				}
			}
		}
		.login-footer{
			height: 100%;
			text-align: center;
			color: #1d1d1f;
			padding-top: 1.2rem;
			.copy{
				padding-top: 1rem;
			}
		}
	}
</style>

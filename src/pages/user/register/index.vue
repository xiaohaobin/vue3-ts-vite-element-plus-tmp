<script setup lang="ts">
import { reactive, ref, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import { ElMessage, FormInstance, FormRules } from "element-plus";
import IconSvg from "@/components/IconSvg/index.vue";

const router = useRouter();
const formRef = ref<FormInstance>();

const formData = reactive({
	username: "",
	password: "",
	confirmPassword: "",
	phone: "",
	code: "",
});

const validateConfirmPassword = (rule: any, value: any, callback: any) => {
	if (value === "") {
		callback(new Error("请再次输入密码"));
	} else if (value !== formData.password) {
		callback(new Error("两次输入密码不一致!"));
	} else {
		callback();
	}
};

const rules = reactive<FormRules>({
	username: [{ required: true, message: "请输入用户名", trigger: "blur" }],
	password: [{ required: true, message: "请输入密码", trigger: "blur" }],
	confirmPassword: [{ validator: validateConfirmPassword, trigger: "blur" }],
	phone: [
		{ required: true, message: "请输入手机号", trigger: "blur" },
		{ pattern: /^1[3-9]\d{9}$/, message: "请输入正确的手机号格式", trigger: "blur" },
	],
	code: [{ required: true, message: "请输入验证码", trigger: "blur" }],
});

// 验证码倒计时
const codeLoading = ref(false);
const codeText = ref("获取验证码");
const codeSeconds = ref(60);
let timer: any = null;

const getCode = () => {
	if (!/^1[3-9]\d{9}$/.test(formData.phone)) {
		ElMessage.warning("请先输入正确的手机号");
		return;
	}
	codeLoading.value = true;
	// 模拟发送请求
	setTimeout(() => {
		ElMessage.success("验证码已发送（模拟）");
		codeText.value = `${codeSeconds.value}s后重发`;
		timer = setInterval(() => {
			codeSeconds.value--;
			if (codeSeconds.value <= 0) {
				clearInterval(timer);
				codeText.value = "获取验证码";
				codeSeconds.value = 60;
				codeLoading.value = false;
			} else {
				codeText.value = `${codeSeconds.value}s后重发`;
			}
		}, 1000);
	}, 1000);
};

onUnmounted(() => {
	if (timer) clearInterval(timer);
});

const loading = ref(false);
const onSubmit = async () => {
	if (loading.value === true) return;
	try {
		const valid = await formRef.value?.validate();
		if (valid) {
			loading.value = true;
			// 模拟注册请求
			setTimeout(() => {
				ElMessage.success("注册成功，请登录");
				router.push("/user/login");
				loading.value = false;
			}, 1500);
		}
	} catch (error) {
		ElMessage.warning("验证不通过，请检查输入");
	}
};
</script>

<template>
	<div class="user-register">
		<el-form ref="formRef" :model="formData" :rules="rules">
			<div class="title">
				<div>注册账号</div>
				<div class="sub-title">加入 Admin Element Vue</div>
			</div>

			<div class="item">
				<el-form-item prop="username">
					<el-input placeholder="用户名" v-model="formData.username" clearable>
						<template #prefix><IconSvg name="user" /></template>
					</el-input>
				</el-form-item>
			</div>

			<div class="item">
				<el-form-item prop="phone">
					<el-input placeholder="手机号" v-model="formData.phone" clearable>
						<template #prefix><IconSvg name="search" /></template>
					</el-input>
				</el-form-item>
			</div>

			<div class="item">
				<el-form-item prop="code">
					<div class="code-row">
						<el-input placeholder="验证码" v-model="formData.code" clearable style="flex: 1">
							<template #prefix><IconSvg name="unlock" /></template>
						</el-input>
						<el-button :disabled="codeLoading" @click="getCode" style="margin-left: 10px">
							{{ codeText }}
						</el-button>
					</div>
				</el-form-item>
			</div>

			<div class="item">
				<el-form-item prop="password">
					<el-input placeholder="密码" type="password" v-model="formData.password" show-password clearable>
						<template #prefix><IconSvg name="lock" /></template>
					</el-input>
				</el-form-item>
			</div>

			<div class="item">
				<el-form-item prop="confirmPassword">
					<el-input placeholder="确认密码" type="password" v-model="formData.confirmPassword" show-password clearable>
						<template #prefix><IconSvg name="lock" /></template>
					</el-input>
				</el-form-item>
			</div>

			<div class="item">
				<el-button @click="onSubmit" :loading="loading" class="width100" type="primary">立即注册</el-button>
			</div>

			<div class="item footer">
				<router-link to="/user/login">已有账号？返回登录</router-link>
			</div>
		</el-form>
	</div>
</template>

<style scoped lang="scss">
.user-register {
	width: 380px;
	padding-bottom: 40px;
	.title {
		padding: 0 20px 20px;
		font-size: 30px;
		line-height: 50px;
		.sub-title {
			font-size: 14px;
			color: #999;
			line-height: 20px;
		}
	}
	.item {
		padding: 5px 20px;
		.width100 {
			width: 100%;
		}
		&.footer {
			text-align: right;
			font-size: 14px;
		}
	}
	.code-row {
		display: flex;
		width: 100%;
	}
}
</style>

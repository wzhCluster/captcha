<template>
	<div id="captcha" :class="{success: (state == 4), init: (state == 0)}"
		:style="'height:' + height + ';width:' + width">
		<div @click='toggleWindow' class="text"><span v-text="content"></span></div>
		<transition name='fade'>
			<div class="x-window" v-show='showWindow'>

				<!-- 头部 -->
				<div class="header">
					<div :style="`background-image:url(${titleUrl})`"> </div>
				</div>

				<!-- 主要内容 -->
				<div class="content" ref="contentDom">
					<div :style="`background-image:url(${imageUrl})`" @click="addTag">
						<div v-for="(data, index) in tags.data" :style="{left: data.x + 'px', top: data.y + 'px'}"
							@click="removeTag" class="tag">
							{{ index + 1}}
						</div>
					</div>
				</div>

				<!-- 底部工具栏 -->
				<div class="bottom">
					<div>
						<div class="but close" @click="showWindow = false"></div>
						<div class="but about"></div>
						<div class="but refresh" @click="loadPic"></div>
					</div>
					<button class='submit' @click="submit">确认</button>
				</div>

			</div>
		</transition>
	</div>
</template>

<script setup lang="ts">
import {ref, reactive, onBeforeMount} from 'vue'
import axios from 'axios'
const props = defineProps<{
	gate: string,
	token: string,
	width: string,
	height: string
}>()
let titleUrl = ref("")
let contentDom = ref()
let imageUrl = ref("")
let tagAble = ref(false)
let token = ref("")
var content = ref("验证码初始化中...")
var state = ref(0)
var showWindow = ref(false)
var tags: any = reactive({data: []})

console.log(contentDom)

const request = axios.create({
	baseURL: props.gate,
	timeout: 10000,
	headers: {'X-API-Header': ''}
});

// 显示隐藏窗口
const toggleWindow: any = () => {
	if (state.value === 1 || state.value === 3) showWindow.value = !showWindow.value
}

// 加载图片
const loadPic = async () => {
	if (state.value !== 1) return;
	state.value = 3 // loading 

	// 刷新文字
	await request.get(`/captcha/refresh/${token.value}`)

	// 清空并停用tag
	clearAllTag()
	tagAble.value = false;
	titleUrl.value = ""
	imageUrl.value = ""

	// 加载图片
	const img = `${props.gate}/captcha/image/${token.value}?r=${Math.random()}`
	console.log(img)

	var content = new Image();
	content.addEventListener("load", () => {
		imageUrl.value = img
		tagAble.value = true;
		state.value = 1
	});
	content.src = img

	//加载头部
	titleUrl.value = `${props.gate}/captcha/title/${token.value}?r=${Math.random()}`
}

// 初始化
onBeforeMount(async () => {
	try {
		// debug
		const data = await axios.get("http://127.0.0.1:8081/captcha/auth/localhost")
		const key = data.data.key
		// -----
		token.value = key

		await request.get("/captcha/init/" + key)
		content.value = "点击此处进行验证"
		state.value = 1
		await loadPic()
	} catch (e) {
		content.value = "初始化失败，请刷新重试"
		console.error(content)
	}
})

const clearAllTag = () => {
	for (let i = 0; i < tags.data.length; i++) tags.data.pop()
	tags.data = []
}

const addTag = (e: any) => {
	if (state.value !== 1) return;
	let x = e.pageX - parseInt(contentDom.value.getBoundingClientRect().left);
	let y = e.pageY - parseInt(contentDom.value.getBoundingClientRect().top);
	tags.data.push({x, y})
}
const removeTag = (e: any) => {
	const index = parseInt(e.target.innerHTML)
	for (let o = tags.data.length - 1; o >= index - 1; o--) {
		tags.data.pop()
	}
	e.preventDefault()
	e.stopPropagation()
}

const submit = async () => {
	try {
		if (tags.data.length === 0) return;
		const i = await request.post(`/captcha/check/${token.value}`,{"data":tags.data})

		content.value = "验证成功"
		state.value = 4 // loading 
		showWindow.value = false
	} catch (e : any) {
		alert(e.response.data.detail)
	} finally {

	}
}

</script>
<style scoped>
.fade-enter-from,
.fade-leave-to {
	opacity: 0;
}

.fade-leave-active,
.fade-enter-active {
	transition: all .2s ease;
}

.fade-leave-from,
.fade-enter-to {
	opacity: 1;
}

.success {
	color: #0c8d42 !important;
	cursor: default;
	border-color: #0c8d42 !important;
	animation: success_ani 0.3s;
	animation-fill-mode: forwards;
	background-color: #d7f5e3 !important;
}

.init {
	color: #888 !important;
	background-color: #ddd !important;
	border-color: #d5d5d5 !important;
	cursor: default;
}

#captcha {
	background-color: #ddd;
	color: #555;
	/* background: linear-gradient(to bottom, #fff, #ccc); */
	border: 1px solid #999;
	box-sizing: border-box;
	user-select: none;
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 1000;
}

#captcha .text {
	width: 100%;
	height: 100%;
	display: flex;
	justify-content: center;
	align-items: center;
}

.x-window {
	border: 1px solid #ccc;
	position: fixed;
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);
	background: white;
	box-shadow: 0 0 15px 10px #ddd;
	padding: 5px 10px;
	width: 300px;
	height: 400px;
}

.x-window .header {
	width: 100%;
	height: 10%;
	box-sizing: border-box;
}

.x-window .header div,
.x-window .content div {
	width: 100%;
	height: 100%;
}

.x-window .content {
	width: 100%;
	height: 75%;
	box-sizing: border-box;
	background-repeat: no-repeat;
	position: relative;
	overflow: hidden;
	display: flex;
	justify-content: center;
	align-items: center;
	background-color: #ddd;
}

.x-window .content img {
	width: 100%;
	height: 100%;
	position: absolute;
	-webkit-user-drag: none;
}

.x-window .bottom {
	padding-top: 10px;
	width: 100%;
	height: 15%;
	box-sizing: border-box;
	display: flex;
	align-items: center;
	flex-direction: row;
	justify-content: space-between;
}

.x-window .bottom button {
	float: right;
	height: 80%;
	width: 120px;
	background: #539ffe;
	color: white;
	border: 0px;
	border-radius: 2px;
	cursor: pointer;
	font-size: 17px;
}

.x-window .bottom .but {
	float: left;
	height: 20px;
	width: 20px;
	margin: 10px 8px;
	background-repeat: no-repeat;
	background-size: 345%;
	cursor: pointer;
	background-image: url(../assets/button.png);
}

.x-window .bottom .close {
	background-position: 0px 0px;
}

.x-window .bottom .close:hover {
	background-position: 0px -26px;
}

.x-window .bottom .about {
	background-position: -49px 0px;
}

.x-window .bottom .about:hover {
	background-position: -49px -26px;
}

.x-window .bottom .refresh {
	background-position: -25px 0px;
}

.x-window .bottom .refresh:hover {
	background-position: -25px -26px;
}

.x-window .content .tag {
	width: 27px;
	height: 27px;
	line-height: 27px;
	position: absolute;
	background-color: #539ffe;
	border-radius: 50%;
	text-align: center;
	border: 3px solid white;
	color: white;
	transform: translate(-50%, -50%);
	cursor: default;
	user-select: none;
	font-weight: 600;
}

.x-window .content .tag {
	width: 27px;
	height: 27px;
	line-height: 27px;
	position: absolute;
	background-color: #539ffe;
	border-radius: 50%;
	text-align: center;
	border: 3px solid white;
	color: white;
	transform: translate(-50%, -50%);
	cursor: default;
	user-select: none;
	font-weight: 600;
}
</style>

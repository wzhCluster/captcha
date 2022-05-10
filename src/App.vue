<template>
	<Captcha gate="https://api.wzh.one" token="" height="40px" width="330px" />
</template>

<script setup lang="ts">

import Captcha from './components/Captcha.vue';
import {ref} from 'vue'
import axios from 'axios'


</script>

<style>
body,
html {
	height: 100%;
	width: 100%;
	margin: 0px;
	padding: 0px;
}

#app {
	display: flex;
	margin-top: 10vh;
	justify-content: center;
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
}
</style>

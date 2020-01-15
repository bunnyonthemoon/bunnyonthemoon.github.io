<template lang="pug">

#app
	.functions
		.functions-btn(v-for="(item, ID) in functions" @click="activeFunc = ID" :class="{active: activeFunc == ID}") {{item.name}}
	| $1:
	input(v-model="value[0]")

	| $2:
	input(v-model="value[1]")
	.sum сумма: {{ sum }}

</template>

<script>
import Axios from 'axios'

export default {
	created(){
		this.loadFunctions()
	},
	data(){
		return {
			value: [
				0,
				0
			],
			activeFunc: '',
			functions: {}
		}
	},
	methods: {
		async loadFunctions(){
			let { data } = await Axios.get('settings.json', {
				method: 'GET',
				mode: 'no-cors',
				headers: {
					'Access-Control-Allow-Origin': '*',
					'Content-Type': 'application/json',
				},
				withCredentials: true,
				credentials: 'same-origin',
			})
			this.functions = data.functions
			this.activeFunc = data.default
		}
	},
	computed:{
		sum(){
			if (this.activeFunc == '') return 0

			let activeFunc = this.functions[this.activeFunc]

			activeFunc.error = activeFunc.error || []
			for (let error of activeFunc.error){

				if ((error.valueID == '0' && this.value[0] != error.value)) continue
				if ((error.valueID == '1' && this.value[1] != error.value)) continue
				if ((error.valueID == 'any' && this.value[0] != error.value && this.value[1] != error.value)) continue
				if ((error.valueID == 'both' && (this.value[0] != error.value || this.value[1] != error.value))) continue

				alert(error.alertText)
				if (!error.calc) return 'Ошибка'
			}
				

			let calc = activeFunc.function.replace(/\$1/g, this.value[0])
			calc = calc.replace(/\$2/g, this.value[1])
			return eval(calc)
		}
	},
	watch: {
		value(val) {
			val[0] = String(val[0]).split(' ').join('').replace(/[^-0-9]/gim, '') * 1
			val[1] = String(val[1]).split(' ').join('').replace(/[^-0-9]/gim, '') * 1
			
		}
	}
}

</script>

<style lang="stylus">

.functions
	&-btn
		display inline-block
		background-color #ddd
		border-radius 5px
		box-shadow 0 2px 6px 0 rgba(#000, .2)
		padding 10px 15px
		font-size 20px
		margin 0 5px
		cursor pointer
		transition all .3s
		&:hover
			box-shadow 0 2px 4px 0 rgba(#000, .2)
		&.active
			background-color #19ff19
	
input
	border 1px solid #aaa
	font-size 18px
	border-radius 3px
	margin 10px
	padding 3px 10px

.sum
	font-size 30px

</style>
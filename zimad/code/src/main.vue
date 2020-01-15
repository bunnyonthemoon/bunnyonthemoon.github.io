<template lang="pug">

#app
	.form
		.control
			.control-btn(v-for="item in functions" @click="activeFunc = item" :class="{active: activeFunc.name == item.name}") {{item.name}}
		.input
			span $1:
			input(v-model="values[0]" @input="format(0)")
		.input
			span $2:
			input(v-model="values[1]" @input="format(1)")
		.sum Cумма: {{ sum }}
		.error(v-if="error") {{ error }}

</template>

<script>
import Axios from 'axios'

export default {
	created(){
		this.loadFunctions()
	},
	data(){
		return {
			values: [
				0,
				0
			],
			activeFunc: null,
			functions: {},
			error: null
		}
	},
	methods: {
		async loadFunctions(){
			let { data } = await Axios.get('settings.json')
			this.functions = data.functions
			this.activeFunc = this.functions[data.default]
		},
		format(id){
			this.values[id] = String(this.values[id]).split(' ').join('').replace(/[^-0-9]/gim, '') * 1
		},
		showError(text){
			this.error = text
		}
	},
	computed:{
		sum(){
			if (!this.activeFunc) return 0

			let isError = false
			if (this.activeFunc.error)
				for (let error of this.activeFunc.error){

					if ((error.target == '$1' && this.values[0] != error.value)) continue
					if ((error.target == '$2' && this.values[1] != error.value)) continue
					if ((error.target == 'any' && this.values[0] != error.value && this.values[1] != error.value)) continue
					if ((error.target == 'both' && (this.values[0] != error.value || this.values[1] != error.value))) continue

					this.showError(error.alertText)
					if (!error.calc) return 'Ошибка'
					isError = true
				}
			if (!isError) this.showError(null)

			let calc = this.activeFunc.function.replace(/\$1/g, this.values[0])
			calc = calc.replace(/\$2/g, this.values[1])
			return eval(calc)
		}
	}
}

</script>

<style lang="stylus">
.form
	padding-top 120px
	display flex
	align-items center
	justify-content center
	flex-direction column
	.control
		margin-bottom 30px
		box-shadow 0 2px 6px 0 rgba(#000, .2)
		&-btn
			display inline-block
			background-color #ddd
			padding 10px 15px
			font-size 20px
			cursor pointer
			transition all .3s
			&:hover
				box-shadow 0 2px 4px 0 rgba(#000, .2)
			&.active
				background-color #19ff19
	.input
		span
			font-size 20px
		input
			border 1px solid #aaa
			font-size 18px
			border-radius 3px
			margin 10px
			padding 3px 10px

	.sum
		font-size 30px

</style>
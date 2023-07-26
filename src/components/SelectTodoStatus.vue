<template>
		<select v-model="selStatusValue" >
			<option v-for="selTodoStatus in todoStatusList" :key="selTodoStatus.index" >{{ selTodoStatus }}</option>
		</select>	
</template>


<script>
export default {

	props: {
		todoStatusList: {
			type: Array,
			requered: false,
			// default: "по умолчанию"   //для пропсов массивов и объектов значение по умолчанию должно быть функцией
			default() {
				return ["значение не установлено"]
			} 	
		}
	},

	data() {
		return {
			selStatusValue: this.todoStatusList[0],  
		};
	},
	
	mounted() {
		console.log(this.selStatusValue)
		this.$emit('selectionListInput', this.selStatusValue);	
	},

	methods: {

	},

	watch: {
		selStatusValue(value) {
			this.$emit('selectionListInput', value);	
		}
	}
}
</script>
<style lang="">
	
</style>
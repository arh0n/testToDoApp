<template>
	<section>
		<div class="flex">
			<div class="min-w-full">
				<label for="newtask" class="block text-sm font-medium text-gray-700">Новая задача (статус задачи)</label>

				
				
				<div class="mt-1 flex rounded-md shadow-md">
					<input v-model="enter_todo_text" @keydown.enter="addNewTask" type="text" name="newtask" id="newtask"
						class="block w-10/12 pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
						placeholder="Текст задачи" />

						<select-todo-status @selectionListInput="changeStatusForNewTodo" :todoStatusList="todoStatusList" class="block w-2/12 rounded-md shadow-md border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm" /> <!--  сделать что бы компонент возвращал выбранный статус для добавления нового таска -->		
				
					</div>

			</div>
		</div>
		<add-button @click="addNewTask" type="button" class="my-4" />
	</section>
</template>

<script>

import AddButton from './AddButton.vue';
import SelectTodoStatus from './SelectTodoStatus.vue';

export default{
	components: {
		AddButton,
		SelectTodoStatus
	},

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
			enter_todo_text: '',
			enter_todo_status: ''
		}
	},
//! Кир - не работает если попытаться добавить туду, после загрузки странице, не выбрав статус
	methods: {
		addNewTask() {
			console.log(this.enter_todo_status);
			if (this.enter_todo_text!="") {
				console.log({txt:this.enter_todo_text, stat:this.enter_todo_status});
				this.$emit('add-todo-textandstatus', {txt:this.enter_todo_text, stat:this.enter_todo_status});
				this.enter_todo_text = "";
			}
		}, 

		changeStatusForNewTodo(newSelStat) {
			this.enter_todo_status = newSelStat;
			//console.log(this.enter_todo_status); 
		}
	}
}

</script>
<template>
	<div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">

		<div class="container">
			<section>
				<div class="flex">
					<div class="min-w-full">
						<label for="newtask" class="block text-sm font-medium text-gray-700">Новая задача</label>
						<div class="mt-1 relative rounded-md shadow-md">
							<input v-model="enter_todo_text" @keydown.enter="addNewTask" type="text" name="newtask" id="newtask"
								class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
								placeholder="Текст задачи" />
						</div>

					</div>
				</div>
				<button @click="addNewTask" type="button"
					class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500">
					<!-- Heroicon name: solid/mail -->
					<svg class="-ml-0.5 mr-2 h-6 w-6" xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 24 24"
						fill="#ffffff">
						<path
							d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z">
						</path>
					</svg>
					Добавить
				</button>
			</section>

			<template v-if="todoList.length > 0">
				<hr class="w-full border-t border-gray-600 my-4" />

				<div>Фильтр: 
					<input v-model="currentTextFilter" /> 
						<button 
							@click="currentPage = currentPage-1"
							v-if="currentPage > 1"
							class = "my-4 mx-2 inline-flex items-center py-1 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500">
							Назад
						</button> 
						<button
							@click="currentPage = currentPage+1" 
							v-if="hasNextpage"
							class = "my-4 mx-2 inline-flex items-center py-1 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500">
							Вперед
						</button>				
				</div>

				<hr class="w-full border-t border-gray-600 my-4" />
				<dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">

					<div v-for="selTodo in forViewTodoList" :key="selTodo.index"
						class="bg-white overflow-hidden shadow-lg rounded-lg border-purple-800 border-solid cursor-pointer">
						
						<div @click="changeTodoStatus(selTodo)" class="px-4 py-5 sm:p-6 text-center">
							<dt class="text-sm font-medium text-gray-500 truncate">
								{{ selTodo.todoStatus }}
							</dt>
							<dd
								:class="{
									'line-through text-gray-300':selTodo.todoStatus=='отменен',
									'text-red-900':selTodo.todoStatus=='критический'
								}"
								class="mt-1 text-3xl font-semibold">
								{{selTodo.todoText}}
							</dd>
						</div>

						<div class="w-full border-t border-gray-200"></div>
						<button
						@click.stop="handleDelete(selTodo)"
							class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none">
							<svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="#718096" aria-hidden="true">
								<path fill-rule="evenodd"
									d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
									clip-rule="evenodd"></path>
							</svg>Удалить
						</button>
					</div>


				</dl>
				<hr class="w-full border-t border-gray-600 my-4" />

			</template>

		</div>
	</div>
</template>

<script>
export default {
	name: "App",

	data() {
		return {
			enter_todo_text: "",
			todoList: [],
			todoStatusList: [
				"ожидает",
				"важный",
				"критический",
				"отменен"
			],    //это не изменяемые данные, не состояния ... куда их лучше перенести? Где объявить, аналог типизированного типа СпизоскЗначений

			selectedTodo: null,   //потом можно за <div @click="changeTodoStatus(selTodo)" class="px-4 py-5 sm:p-6 text-center"> просто присваивать этому полю ссылку на элемент, а всю логику выполнять в вейтере
			
			currentPage: 1,
			currentTextFilter: "",
			currentStatusFilter: "",

			countElementsOnPage: 6 //количество элементов на странице
		};
	},

	created() {
		const windowData = Object.fromEntries(new URL(window.location).searchParams.entries());
		if (windowData.filter) {
			this.currentTextFilter = windowData.filter;
		}
		if (windowData.page) {
			this.currentPage = windowData.page;
		}


		const todosData = localStorage.getItem("todo-list");
		if (todosData) {
			this.todoList = JSON.parse(todosData);
		} 
	},

	computed: {
		filtredByTextTodoList() {
			
			let newArr = [];

			//фильтруем по тексту todo
			newArr = this.todoList.filter(elTodo=>elTodo.todoText.includes(this.currentTextFilter));

			return newArr;
		},

		filtredByStatusTodoList() {
			
			let newArr = [];
			
			//фильтруем по статусу todo
			newArr = this.todoList.filter(elTodo=>elTodo.todoStatus.includes(this.currentStatusFilter));

			return newArr;
		},

		hasNextpage() {
			if (this.filtredByTextTodoList.length - this.countElementsOnPage*this.currentPage > 0 ) {
				return true;
			} else {
				return false;
			}
		},

		filtredByPageTodoList() {
			
			var newArr = [];
			
			const start = (this.currentPage-1)*this.countElementsOnPage;
			const end = this.currentPage*this.countElementsOnPage; //for slice() end NOT included

			//пагинация
			newArr = this.filtredByTextTodoList.slice(start, end);

			return newArr;
		},

		forViewTodoList() {
			return this.filtredByPageTodoList;
		}

	},

	watch: {
		currentTextFilter() {
			this.currentPage = 1;

			//разобрать более внимательно
			window.history.pushState(
				null, 
				document.title, 
				`${window.location.pathname}?filter=${this.currentTextFilter}&page=${this.currentPage}`
				);
		},

		currentPage() {
			//разобрать более внимательно
			window.history.pushState(
				null, 
				document.title, 
				`${window.location.pathname}?filter=${this.currentTextFilter}&page=${this.currentPage}`
				);
		}

	},

	methods: {
		addNewTask() {
			if (this.enter_todo_text!="") {
					const new_todo = {
					todoText: this.enter_todo_text,
					todoStatus: this.getNextTodoStatus()
				};

				this.todoList.push(new_todo);

				this.enter_todo_text = "";
				
				localStorage.setItem("todo-list", JSON.stringify(this.todoList)); //вынести в вейтеры (обновление todoList) для обновления данных в локалстор
			
				this.currentTextFilter = "";
				this.currentStatusFilter = "";
			}
		}, 

		//при удалении, не совсем хорошо работает пагинация, наступает ситуация когда остается пустая страниица, а надо что бы currentPage изменился на -1
		handleDelete(todoToRemove) {
			this.todoList = this.todoList.filter(t=>t!=todoToRemove);

			localStorage.setItem("todo-list", JSON.stringify(this.todoList)); //вынести в вейтеры (обновление todoList) для обновления данных в локалстор
		},

		getNextTodoStatus(currentStatus) {
			let indexOfCurrentStatus = this.todoStatusList.indexOf(currentStatus, 0);
			let nextStatus = "";
			if ((indexOfCurrentStatus==-1) || (indexOfCurrentStatus==(this.todoStatusList.length-1))) {
				nextStatus = this.todoStatusList[0];	
			} else {
				nextStatus = this.todoStatusList[indexOfCurrentStatus+1];
			}
			return nextStatus;
		},

		changeTodoStatus(currentElement) {
			this.selectedTodo = currentElement;
			currentElement.todoStatus = this.getNextTodoStatus(currentElement.todoStatus);

			localStorage.setItem("todo-list", JSON.stringify(this.todoList)); //вынести в вейтеры (обновление todoList) для обновления данных в локалстор
		}
	}
};
</script>

<style src="./app.css">

</style>

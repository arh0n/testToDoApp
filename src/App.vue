<template>
	<div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">

		<div class="container" ref="container_div">  <!-- кир использую реф для обучения -->

			<add-todo @add-todo-textandstatus="addNewTask" :todoStatusList="this.todoStatusList" />

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
				<dl class="mt-5 grid grid-cols-1 gap-5 lg:grid-cols-3">

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

						<del-button @click.stop="handleDelete(selTodo)"/>
						<!-- <button
						@click.stop="handleDelete(selTodo)"
							class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none">
							<svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="#718096" aria-hidden="true">
								<path fill-rule="evenodd"
									d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
									clip-rule="evenodd"></path>
							</svg>Удалить
						</button> -->
					</div>


				</dl>
				<hr class="w-full border-t border-gray-600 my-4" />
				
				<div>
					Работа с рефом (ширина container_div): {{containerDivWidth}}
				</div>
			</template>

		</div>
	</div>
</template>



<script>

//проблемы:
// - если у нас есть тудушки, и мы производим изменение названия статуса в списке статусов, то у старых которые в локал стор. и на странице не меняется ...

import AddTodo from './components/AddTodo.vue';
import DelButton from './components/DelButton.vue';


export default {
	name: "App",

	components: {
		AddTodo,
		DelButton
	},

	data() {
		return {
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

			countElementsOnPage: 6, //количество элементов на странице

			containerDivWidth: "" //в методе будем расчитывать через подписку в маунтед на виндоус ресайз
		};
	},

	created() {
		//загружаем состояние фильтра и страницы из урла
		const windowData = Object.fromEntries(new URL(window.location).searchParams.entries());
		if (windowData.filter) {
			this.currentTextFilter = windowData.filter;
		}
		if (windowData.page) {
			this.currentPage = windowData.page;
		}

		//загружаем наши тудушки из локального хранилища
		const todosData = localStorage.getItem("todo-list");
		if (todosData) {
			this.todoList = JSON.parse(todosData);
		} 
		
	},

	mounted() {
		window.addEventListener("resize", this.calculateContainerDivWidth); //подписался на события ресайза для вычисления ширины дива (можно было и в крейтед, но в маунте я показываю что для реф. мне важно наличие сформированного дома)
		this.calculateContainerDivWidth(); //вызовем при создании не дожидаясь ресайза, что бы цифра была видна сразу
	},

	beforeUnmount() {
		window.removeEventListener("resize", this.calculateContainerDivWidth); //отпишимся на всякий случай
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
		},

		currentPageStateOptions() {
			return {
				currentTextFilter: this.currentTextFilter,
				currentPage: this.currentPage
			}
		}

	},

	watch: {

		currentPageStateOptions(value) {
			window.history.pushState(
				null, 
				document.title, 
				`${window.location.pathname}?filter=${value.currentTextFilter}&page=${value.currentPage}`
				);
		},

		currentTextFilter() {
			this.currentPage = 1;
		},

		filtredByPageTodoList() {
			if (this.filtredByPageTodoList.length === 0 && this.currentPage>1) {
				this.currentPage -= 1;
			}
		},

		//вотч на удаление и добавление данных в наш начальный массив (может нужен модификатор deep:, что то было в документации) (отработка на добавлении и удалении тудушки - проверить)
		todoList: {
			handler() {
				//срабатывает в начальной инициализации, может убрать
				localStorage.setItem("todo-list", JSON.stringify(this.todoList)); 
			},
			deep:true  //можно и без дипа обойтись, но тогда при добавлении эл-та в массив надо не пушем, а обновлением ссылки делать, типа mas = [...mas, '5']
		}

	},

	methods: {
		addNewTask(enter_todo) {
			if (enter_todo.txt!="") {
					const new_todo = {
					todoText: enter_todo.txt,
					todoStatus: enter_todo.stat   //должны получить из компонента статус новой тудушки и подставить сюда
				};

				this.todoList.push(new_todo);  //что бы в вотчере не использовать дип, надо здесь не пуш, а mas = [...mas, '5']
				
				// перенес в вотч todoList()
				this.currentTextFilter = ""; //кир - проверить так ли
				this.currentStatusFilter = "";
			}

		}, 

		
		handleDelete(todoToRemove) {
			this.todoList = this.todoList.filter(t=>t!=todoToRemove);

			// перенес в вотч todoList() - работает.. без дип, почему? разобраться (идет ли здесь обновление ссылки?)
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
		},

		calculateContainerDivWidth() {
			if (!this.$refs.container_div) {
				return;
			}  //предохранитель, здесь не очень важно..
			this.containerDivWidth = this.$refs.container_div.clientWidth;
		}
	}
};
</script>

<style src="./app.css">

</style>

<template>
	<div>
		<input
			type="text"
			class="todo-input"
			placeholder="What needs to be done"
			v-model="newTodo"
			@keyup.enter="addTodo"
		/>
		<transition-group
			name="fade"
			enter-active-class="animated fadeInUp"
			leave-active-class="animated fadeOutDown"
		>
			<!-- keyup.enter : 엔터를 눌렀다 뗄 때 반응 -->
			<!-- blur: 엘리먼트의 포커스가 해제되었을때 발생 -->
			<!-- TodoItem 컴포넌트에 todo와 index를 전달 -->
			<todo-item
				v-for="(todo, index) in todosFiltered"
				:key="todo.id"
				:todo="todo"
				:index="index"
				:checkAll="!anyRemaining"
			></todo-item>
		</transition-group>

		<div class="extra-container">
			<todo-check-all :anyRemaining="anyRemaining"></todo-check-all>
			<todo-items-remaining :remaining="remaining"></todo-items-remaining>
		</div>

		<div class="extra-container">
			<todo-filtered></todo-filtered>
			<div>
				<transition name="fade">
					<todo-clear-completed :showClearCompletedButton="showClearCompletedButton"></todo-clear-completed>
				</transition>
			</div>
		</div>
	</div>
</template>

<script>
import TodoItem from './TodoItem'
import TodoItemsRemaining from './TodoItemsRemaining'
import TodoCheckAll from './TodoCheckAll'
import TodoFiltered from './TodoFiltered'
import TodoClearCompleted from './TodoClearCompleted'


export default {
	name: 'todo-list',
	components: {
		TodoItem, TodoItemsRemaining, TodoCheckAll, TodoFiltered, TodoClearCompleted
	},
	data() {
		return {
			newTodo: '',
			idForTodo: 3,
			beforeEditCache: '',
			filter: 'all',
			todos: [
				{
					'id': 1,
					'title': 'Finish Vue Screencast',
					'completed': false,
					'editing': false,
				},
				{
					'id': 2,
					'title': 'Take over world',
					'completed': false,
					'editing': false,
				},
			]
		}
	},
	created() {
		eventBus.$on('removedTodo', (index) => this.removeTodo(index))
		eventBus.$on('finishedEdit', (data) => this.finishedEdit(data))
		eventBus.$on('checkAllChanged', (checked) => this.checkAllTodos(checked))
		eventBus.$on('filterChanged', (filter) => this.filter = filter)
		eventBus.$on('clearCompletedTodos', () => this.clearCompleted())
	},
	beforeDestroy() {
		eventBus.$off('removedTodo', (index) => this.removeTodo(index))
		eventBus.$off('finishedEdit', (data) => this.finishedEdit(data))
		eventBus.$off('checkAllChanged', (checked) => this.checkAllTodos(checked))
		eventBus.$off('filterChanged', (filter) => this.filter = filter)
		eventBus.$off('clearCompletedTodos', () => this.clearCompleted())
	},
	computed: {
		remaining() {
			//filter : 주어진 함수의 테스트를 통과하는 모든 요소를 모아 새로운 배열로 반환
			return this.todos.filter(todo => !todo.completed).length
		},
		anyRemaining() {
			return this.remaining != 0
		},
		todosFiltered() {
			if (this.filter == 'all') {
				return this.todos
			} else if (this.filter == 'active') {
				return this.todos.filter(todo => !todo.completed)
			} else if (this.filter == 'completed') {
				return this.todos.filter(todo => todo.completed)
			}
			return this.todos
		},
		showClearCompletedButton() {
			return this.todos.filter(todo => todo.completed).length > 0
		}
	},
	methods: {
		addTodo() {
			//공백 허용 X
			if (this.newTodo.trim().length == 0) {
				return;
			}

			this.todos.push({
				id: this.idForTodo,
				title: this.newTodo,
				completed: false,
			})

			this.newTodo = ''
			this.idForTodo++
		},
		removeTodo(index) {
			//index 배열부터 1개 제거
			this.todos.splice(index, 1);
		},
		checkAllTodos() {
			this.todos.forEach((todo) => todo.completed =
				event.target.checked)
		},
		clearCompleted() {
			this.todos = this.todos.filter(todo => !todo.completed)
		},
		finishedEdit(data) {
			this.todos.splice(data.index, 1, data.todo)
		}
	}
}
</script>

<style>
@import url("https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css");

.todo-input {
	width: 100%;
	padding: 10px 18px;
	font-size: 18px;
	margin-bottom: 16px;
}
.todo-input:focus {
	outline: 0;
}
.todo-item {
	margin-bottom: 12px;
	display: flex;
	align-items: center;
	justify-content: space-between;
	font-size: 24px;
	animation-duration: 0.3s;
}
.remove-item {
	cursor: pointer;
	margin-left: 14px;
}
.remove-item:hover {
	color: black;
}
.todo-item-left {
	display: flex;
	align-items: center;
}
.todo-item-label {
	padding: 10px;
	border: 1px solid white;
	margin-left: 12px;
}
.todo-item-edit {
	font-size: 24px;
	color: #2c3e50;
	margin-left: 12px;
	width: 100%;
	padding: 10px;
	border: 1px solid #ccc;
	font-family: "Avenir", Helvetica, Arial, sans-serif;
}
.todo-item-edit:focus {
	outline: none;
}
.completed {
	text-decoration: line-through;
	color: grey;
}
.extra-container {
	display: flex;
	align-items: center;
	justify-content: space-between;
	font-size: 16px;
	border-top: 1px solid lightgrey;
	padding-top: 14px;
	margin-bottom: 14px;
}
button {
	font-size: 14px;
	background-color: white;
	appearance: none;
}
button:hover {
	background: lightgreen;
}
button:focus {
	outline: none;
}
.active {
	background: lightgreen;
}

.fade-enter-active,
fade-leave-active {
	transition: opacity 0.2s;
}
.fade-enter,
.fade-leave-to {
	opacity: 0;
}
</style>

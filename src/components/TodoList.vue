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
			<div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item">
				<div class="todo-item-left">
					<input type="checkbox" v-model="todo.completed" />
					<div
						v-if="!todo.editing"
						@dblclick="editTodo(todo)"
						class="todo-item-label"
						:class="{completed : todo.completed}"
					>{{todo.title}}</div>
					<input
						v-else
						class="todo-item-edit"
						type="text"
						v-model="todo.title"
						@blur="doneEdit(todo)"
						@keyup.enter="doneEdit(todo)"
						@keyup.esc="cancelEdit(todo)"
						v-focus
					/>
					<div class="remove-item" @click="removeTodo(index)">&times;</div>
				</div>
			</div>
		</transition-group>
		<div class="extra-container">
			<div>
				<label>
					<input type="checkbox" :checked="!anyRemaining" @change="checkAllTodos()" />Check All
				</label>
			</div>
			<div>{{remaining}}items left</div>
		</div>

		<div class="extra-container">
			<div>
				<button :class="{ active: filter == 'all'}" @click="filter = 'all'">All</button>
				<button :class="{ active: filter == 'active'}" @click="filter = 'active'">Active</button>
				<button :class="{ active: filter == 'completed'}" @click="filter = 'completed'">Completed</button>
			</div>
			<div>
				<transition name="fade">
					<button v-if="showClearCompletedButton" @click="clearCompleted">Clear Completed</button>
				</transition>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: 'todo-list',
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
					'title': 'Take ovver world',
					'completed': false,
					'editing': false,
				},
			]
		}
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
	directives: { //v-focus로 실행, vue 가이드에 있음
		focus: {
			inserted: function (el) {
				el.focus()
			}
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
		editTodo(todo) {
			this.beforeEditCache = todo.title
			todo.editing = true;
		},
		doneEdit(todo) {
			//수정시 공백 X
			if (todo.title.trim() == '') {
				todo.title = this.beforeEditCache
			}
			todo.editing = false;
		},
		cancelEdit(todo) {
			todo.title = this.beforeEditCache
			todo.editing = false;
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
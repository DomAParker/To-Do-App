<script setup>
import { ref, onMounted, computed, watch } from 'vue'

// Setting Variable and Constants------------------------------------------------------------------

var todos = ref([{
  "taskID": '',
  "taskName": "",
  "taskCategory": "",
  "taskStatus": ""
}])
const name = ref('')
var categories = ref('')
var categoriesAmount = ref('')
var new_category = ref('')
var nameSubmitted = ref(false)

var isCatOpen = ref(false)

var input_task = ref('')
var input_taskCat = ref(null)

const todos_desc = computed(() => todos.value.sort((a, b) => {
  return b.taskID - a.taskID
}))

var i = ref()
var tempTodos = ref([])
var nextTaskID = ref('')


// Vue Functions (watch, onMounted)----------------------------------------------------------------

onMounted(() => {
  name.value = localStorage.getItem('name') || ""
  if (name.value == null || name.value == "") {
    nameSubmitted.value = false;
  } else {
    nameSubmitted.value = true;
  }

  nextTaskID.value = JSON.parse(localStorage.getItem('nextTaskID')) || 0
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
  categories.value = JSON.parse(localStorage.getItem('categories')) || (["Personal", "Work", "Coding"])

  isCatOpen.value = false

})

watch(name, newVal => {
  localStorage.setItem('name', newVal)
})

watch(todos, newVal => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, { deep: true })

watch(nextTaskID, newVal => {
  localStorage.setItem('nextTaskID', newVal)
})

watch(categories, newVal => {
  localStorage.setItem('categories', JSON.stringify((newVal)))
  categoriesAmount = categories.value.length
}, { deep: true })


// Custom Functions--------------------------------------------------------------------------------

const submitName = () => {
  if (name.value != "") {
    nameSubmitted.value = true;
  } 
}

const resetName = () => {
  nameSubmitted.value = false;
  name.value = "";
}

const submitTask = () => {
  if (input_task.value != "") {
    todos.value.push({
      taskID: nextTaskID.value,
      taskName: input_task.value,
      taskCategory: input_taskCat.value,
      taskStatus: "In Progress"
    })
    nextTaskID.value = nextTaskID.value + 1;
    input_task.value = ""
    input_taskCat.value = null
  }
}

const deleteTask = (deleteID) => {
  tempTodos = JSON.parse(localStorage.getItem('todos'))
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (deleteID == todos.value[i].taskID) {
      tempTodos.splice(i, 1)
      todos.value = tempTodos
  }
  }
}

const addCategory = () => {
  if (new_category.value != "") {
  categories.value.push(new_category.value)
    new_category.value = ""
}
}

const openAddCat = () => {
  if (isCatOpen.value == true) {
    isCatOpen.value = false
} else {
    isCatOpen.value = true
}
}

const completeTask = (completeID) => {
  tempTodos = JSON.parse(localStorage.getItem('todos'))
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (completeID == todos.value[i].taskID) {
      todos.value[i].taskStatus = "Complete"
  }
  }
}

const reopenTask = (reopenID) => {
  tempTodos = JSON.parse(localStorage.getItem('todos'))
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (reopenID == todos.value[i].taskID) {
      todos.value[i].taskStatus = "In Progress"
  }
  }
}

const removeCategory = (removeCatIndex) => {
  for (let i = 0; i < categories._rawValue.length; i++) {
    if (removeCatIndex == i) {
      if (categories.value.length != 1) {
        categories.value.splice(i, 1)
    } else {
      categories.value = []
    }
  }
}
}

//--------------------------------------------------------------------------------------------------
</script>

<template>
  <main class="class">
    <!-- Greeting section -->
    <section class="greeting">
      <span v-if="nameSubmitted == false">Welcome, what is your name?<br>
      <input v-model="name" type="text">
      <button @click="submitName">Submit</button></span>
      <span class="welcomeName" v-else>Welcome back, {{ name }}.</span>
      <button class="resetName" v-if="nameSubmitted == true" @click="resetName">Reset Name</button>
    </section>

    <!-- Input section -->
    <section class="taskInput">
      <h1>Input a task</h1>
      <input type="text" v-model="input_task"><br>
      <h3>Category</h3>
      <div v-if="isCatOpen == false">
        <button @click="openAddCat">Add Categories</button>
      </div>
      <div v-if="isCatOpen == true">
        <input type="text" v-model="new_category">
        <button @click="addCategory">Add</button>
        <button @click="openAddCat">Close</button>
      </div>
      <span v-if="categoriesAmount == 0">You need to add some categories!</span>
      <br>
      <div v-for="(category, index) in categories">
        <input type="radio" :id="index" name="category" :value="category" v-model="input_taskCat">{{ category }}
        <button v-if="isCatOpen == true" @click="removeCategory(index)">Remove</button>
      </div><br>
      <button @click="submitTask">Submit</button>
    </section>
    <br>


    <!-- Task Display -->
    <li v-for="category in categories">
      <div class="categoryTitle">{{ category }}</div>
      <li v-for="task in todos">
        <div class = "task.taskStatus" v-if="category == task.taskCategory" :class="task.taskStatus">
          Name: {{ task.taskName }} <br> 
          Status: {{ task.taskStatus }} <br>
          <button class="completeButton" v-if="task.taskStatus == 'In Progress'" @click="completeTask(task.taskID)" :id=task.taskID>Complete</button>
          <button v-if="task.taskStatus == 'Complete'" @click="reopenTask(task.taskID)" :id=task.taskID>Reopen</button>
          <button @click="deleteTask(task.taskID)" :id=task.taskID>Delete</button>
        </div>
      </li>
    </li>

  </main>
  
</template>

<style scoped>
li {
  list-style-type: none;
  padding-left: 5px;
}

.resetName {
  margin: 5px;
  opacity: 0;
  transition: 0.5s;
}

.welcomeName:hover + .resetName {
  opacity: 1;
}

.resetName:hover {
  opacity: 1;
}

.Complete {
  color: rgb(87, 167, 87);
}

.categoryTitle {
  padding-top: 20px;
  font-weight: bold;
}

.task.taskStatus {
  background-color: black;
  padding: 4px;
}

.completeButton {
  margin-bottom: 10px;
}
</style>

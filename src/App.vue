<script setup>
import { ref, onMounted, computed, watch } from 'vue'

// Setting Variables and Constants------------------------------------------------------------------

var todos = ref([{
  "taskID": '',
  "taskName": "",
  "taskCategory": "",
  "taskStatus": "",
  "subtaskList": {
    "subtaskID" : "",
    "subtaskName": "",
    "subtaskStatus": ""
  }
}])
const name = ref('')
var categories = ref('')
var categoriesAmount = ref('')
var new_category = ref('')
var nameSubmitted = ref(false)

var isCatOpen = ref(false)

var input_task = ref('')
var input_taskCat = ref(null)
var input_subtask = ref('')

var i = ref()
var tempTodos = ref([])
var tempSubtasks = ref([])
var nextTaskID = ref('')
var nextSubtaskID = ref('')

var numberInProgress = ref('')
var currentTaskID = ref('')

// Commenting this code in case needed in future (ordering of To Do tasks)
//const todos_desc = computed(() => todos.value.sort((a, b) => {
//  return b.taskID - a.taskID
//}))


// Vue Functions (watch, onMounted)----------------------------------------------------------------

onMounted(() => {
  name.value = localStorage.getItem('name') || ""
  if (name.value == null || name.value == "") {
    nameSubmitted.value = false;
  } else {
    nameSubmitted.value = true;
  }

  nextTaskID.value = JSON.parse(localStorage.getItem('nextTaskID')) || 0
  nextSubtaskID.value = JSON.parse(localStorage.getItem('nextSubtaskID')) || 0
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

watch(nextSubtaskID, newVal => {
  localStorage.setItem('nextSubtaskID', newVal)
})

watch(categories, newVal => {
  localStorage.setItem('categories', JSON.stringify((newVal)))
  categoriesAmount = categories.value.length
}, { deep: true })


// Custom Functions--------------------------------------------------------------------------------

// Functions for user's name at top of page

const submitName = () => {
  if (name.value != "") {
    nameSubmitted.value = true;
  } 
}

const resetName = () => {
  nameSubmitted.value = false;
  name.value = "";
}


// Functions relating to tasks
const submitTask = () => {
  if (input_task.value != "" && input_taskCat.value != null) {
    todos.value.push({
      taskID: nextTaskID.value,
      taskName: input_task.value,
      taskCategory: input_taskCat.value,
      taskStatus: "Pending Completion",
      subtaskList: []
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

const completeTask = (completeID) => {
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (completeID == todos.value[i].taskID) {
      todos.value[i].taskStatus = "Complete"
  }
  }
}

const reopenTask = (taskID) => {
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (taskID == todos.value[i].taskID) {
      todos.value[i].taskStatus = "In Progress"
  }
  }
  checkSubtaskComplete(taskID)
}

// Functions relating to categories
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

const removeCategory = (removeCatIndex) => {
  for (let i = 0; i < categories._rawValue.length; i++) {
    if (removeCatIndex == i) {
      if (categories.value.length != 1) {ess
        categories.value.splice(i, 1)
    } else {
      categories.value = []
    }
  }
}
}


// Functions relating to subtasks
const addSubtask = (taskID) => {
  if (input_subtask.value != "") {
    for (let i = 0; i < todos._rawValue.length; i++) {
      if (taskID == todos.value[i].taskID) {
        currentTaskID = i;
        todos.value[i].subtaskList.push({
          subtaskID: nextSubtaskID.value,
          subtaskName: input_subtask,
          subtaskStatus: "In Progress"
        }
        )
      }
    }
    input_subtask = ""
    nextSubtaskID.value = nextSubtaskID.value + 1
    todos.value[currentTaskID].taskStatus = "In Progress"
    console.log(input_subtask)
  }
}

const deleteSubtask = (taskID, subtaskID) => {
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (taskID == todos.value[i].taskID) {
      tempSubtasks = todos.value[i].subtaskList
      for (let j = 0; j < tempSubtasks.length; j++) {
        if (subtaskID == tempSubtasks[j].subtaskID) {
          tempSubtasks.splice(j, 1)
          todos.value[i].subtaskList.value = tempSubtasks
        }
      }
  }
  }
  checkSubtaskComplete(taskID)
}

const completeSubtask = (taskID, subtaskID) => {
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (taskID == todos.value[i].taskID) {
      for (let j = 0; j < todos.value[i].subtaskList.length; j++) {
        if (subtaskID == todos.value[i].subtaskList[j].subtaskID) {
          todos.value[i].subtaskList[j].subtaskStatus = "Complete"
        }
      }
  }
  }
  checkSubtaskComplete(taskID)
}

const reopenSubtask = (taskID, subtaskID) => {
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (taskID == todos.value[i].taskID) {
      for (let j = 0; j < todos.value[i].subtaskList.length; j++) {
        if (subtaskID == todos.value[i].subtaskList[j].subtaskID) {
          todos.value[i].subtaskList[j].subtaskStatus = "In Progress"
        }
      }
  }
  }
  checkSubtaskComplete(taskID)
}

const checkSubtaskComplete = (taskID) => {
  numberInProgress = 0
  for (let i = 0; i < todos._rawValue.length; i++) {
    if (taskID == todos.value[i].taskID) {
      currentTaskID = i
      for (let j = 0; j < todos.value[i].subtaskList.length; j++) {
        if (todos.value[i].subtaskList[j].subtaskStatus == "In Progress") {
          numberInProgress = numberInProgress + 1;
        } 
      }
  }
  }
  
  if (numberInProgress == 0) {
    todos._rawValue[currentTaskID].taskStatus = "Pending Completion"
  } else {
    todos._rawValue[currentTaskID].taskStatus = "In Progress"
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
    <section class="inputSection">

      <!-- Task Input -->
      <h1>Input a task</h1>
      <input type="text" v-model="input_task"><br>

      <!-- Category Input -->
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
      <!-- Display Categories-->
      <div v-for="(category, index) in categories">
        <input type="radio" :id="index" name="category" :value="category" v-model="input_taskCat">{{ category }}
        <button v-if="isCatOpen == true" @click="removeCategory(index)">Remove</button>
      </div><br>

      <!-- Submit Task Button -->
      <button @click="submitTask">Submit</button>
    </section>
    <br>


    <!-- Task Display -->
    <!-- Section for each category -->
    <li v-for="category in categories">
      <div class="categoryTitle">{{ category }}</div>
      <!-- Each task within each category -->
      <li v-for="task in todos">
        <div class = "task.taskStatus" v-if="category == task.taskCategory" :class="task.taskStatus">
          Name: {{ task.taskName }} 
          <button @click="deleteTask(task.taskID)" :id=task.taskID>Delete</button><br> 
          Status: {{ task.taskStatus }} 
          <button v-if="task.taskStatus == 'Pending Completion'" @click="completeTask(task.taskID)" :id=task.taskID>Complete</button>
          <button v-if="task.taskStatus == 'Complete'" @click="reopenTask(task.taskID)" :id=task.taskID>Reopen</button><br>
          <!-- Each subtask within each task -->
          Sub-tasks: 
          <!-- Subtask Input -->
          <input v-model="input_subtask" type="text">
          <button @click="addSubtask(task.taskID)" :id="task.taskID">Add</button>
          <!-- Subtask List-->
          <li class="subtaskList" v-for="(subtask, index) in task.subtaskList">
            <div :class="subtask.subtaskStatus">
              {{index + 1}}: {{ subtask.subtaskName }} - {{ subtask.subtaskStatus }}
              <button v-if="subtask.subtaskStatus =='In Progress'" @click="completeSubtask(task.taskID, subtask.subtaskID)">Complete</button>
              <button v-if="subtask.subtaskStatus =='Complete'" @click="reopenSubtask(task.taskID, subtask.subtaskID)">Reopen</button>
              <button @click="deleteSubtask(task.taskID, subtask.subtaskID)">Delete</button>
            </div>
        </li> <br>
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
</style>

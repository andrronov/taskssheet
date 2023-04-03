<template>
  <div class="wrapper">
    <div v-if="firstTime == true" class="modal__window">
      <div class="modal__content">
        <h1>Hey! What's your name?</h1>
        <input
          class="modal_input"
          v-model="user_name_enter"
          @keydown.enter="confirmUserName()"
          type="text"
          placeholder="Enter your name:"
        />
        <button @click="confirmUserName()" class="modal_butt">Confirm</button>
      </div>
    </div>
    <header class="header">
      <div class="header__container _container">
        <div class="header__logo">
          <h1>Your task sheet</h1>
        </div>
        <button @click="deleteAll" class="addTask">Delete All</button>
      </div>
    </header>
    <main class="main">
      <div class="main__container _container">
        <div class="header__input">
          <h2 class="hello__title">Hello, {{ user_name }}!</h2>
          <input
            v-model="input_text"
            @keydown.enter="addTask"
            class="enterTask"
            placeholder="Write your task:"
            type="text"
            id="add"
          />
          <!-- <p class="err">There must be more than 1 symbol</p> -->
          <button @click="addTask" class="addTask">Add</button>
        </div>
        <div class="task__todo">
          <div class="tasks">
            <h2>Tasks</h2>
            <p class="added__Counter _counter">{{ task_items.length }}</p>
          </div>
          <div class="line"></div>
          <p v-if="task_items.length < 1">Write your first task!</p>
          <div
            v-for="(t, task_index) in task_items"
            :key="t.task_index"
            class="item__task"
          >
            <input
              v-model="t.checked"
              @change="moveTask(task_index, 'todo')"
              true-value="done"
              false-value="todo"
              class="check__task"
              type="checkbox"
            />
            <p class="text__task">{{ t.task_text }}</p>
            <button @click="deleteTask(t)" class="remove__Task">delete</button>
          </div>
        </div>
        <div class="task__done">
          <div class="tasks">
            <h2>Done</h2>
            <p class="removed__Counter _counter">
              {{ done_task_items.length }}
            </p>
          </div>
          <div class="line"></div>
          <p v-if="done_task_items.length < 1">Do your first task!</p>
          <div
            v-for="(t, task_index) in done_task_items"
            :key="t.task_index"
            class="item__task _done"
          >
            <input
              v-show="done_task_items.length >= 1"
              type="checkbox"
              v-model="t.checked"
              @change="moveTask(task_index, 'done')"
              true-value="done"
              false-value="todo"
              checked
              class="check__removed"
            />
            <p v-show="done_task_items.length >= 1" class="text__task">
              {{ t.task_text }}
            </p>
            <button
              v-show="done_task_items.length >= 1"
              @click="deleteTask(t)"
              class="remove__Task"
            >
              delete
            </button>
          </div>
        </div>
      </div>
    </main>
    <footer class="footer">
      <div class="footer__container _container">
        <p class="footer_text">by andrronov.a, 30.12.2022</p>
      </div>
    </footer>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      task_items: [],
      done_task_items: [],
      input_text: "",
      index: 0,
      user_name_enter: "",
      user_name: "",
      firstTime: true,
    };
  },

  created() {
    const tasksToDoData = localStorage.getItem("tasksToDo");
    const tasksDoneData = localStorage.getItem("tasksDone");
    const userNameData = localStorage.getItem("userName");
    const firstTime = localStorage.getItem("firstTime");
    if (tasksToDoData) {
      this.task_items = JSON.parse(tasksToDoData);
    }
    if (tasksDoneData) {
      this.done_task_items = JSON.parse(tasksDoneData);
    }
    if (userNameData) {
      this.user_name = JSON.parse(userNameData);
    }
    if (firstTime) {
      this.firstTime = JSON.parse(firstTime);
    }
  },

  methods: {
    confirmUserName() {
      this.user_name = this.user_name_enter;
      this.firstTime = false;
      localStorage.setItem("userName", JSON.stringify(this.user_name));
      localStorage.setItem("firstTime", JSON.stringify(this.firstTime));
    },
    addTask() {
      const newTask = {
        checked: "todo",
        task_text: this.input_text,
        task_index: this.index,
      };
      const inputBorder = document.querySelector(".enterTask");
      const inputValue = document.querySelector("#add").value;
      // const inputContent = inputValue.toLowerCase();
      if (inputValue.length > 0) {
        this.index++;
        this.task_items.push(newTask);
        inputBorder.classList.remove("errer");
        this.input_text = "";
      } else {
        inputBorder.classList.add("errer");
      }
      localStorage.setItem("tasksToDo", JSON.stringify(this.task_items));
    },
    deleteTask(taskToRemove) {
      this.index--;
      this.task_items = this.task_items.filter((t) => t !== taskToRemove);
      this.done_task_items = this.done_task_items.filter(
        (t) => t !== taskToRemove
      );
      localStorage.setItem("tasksToDo", JSON.stringify(this.task_items));
      localStorage.setItem("tasksDone", JSON.stringify(this.done_task_items));
    },
    moveTask(task_index, checked) {
      if (checked === "todo") {
        const doneTask = this.task_items.splice(task_index, 1);
        this.done_task_items.push(...doneTask);
        localStorage.setItem("tasksToDo", JSON.stringify(this.task_items));
        localStorage.setItem("tasksDone", JSON.stringify(this.done_task_items));
      } else {
        const taskTo = this.done_task_items.splice(task_index, 1);
        this.task_items.push(...taskTo);
        localStorage.setItem("tasksToDo", JSON.stringify(this.task_items));
        localStorage.setItem("tasksDone", JSON.stringify(this.done_task_items));
      }
    },
    deleteAll() {
      const resltConfirm = confirm(`Are you sure, ${this.user_name}?`);
      if (resltConfirm == true) {
        (this.task_items = []), (this.done_task_items = []);
        this.index = 0;
        localStorage.removeItem("tasksToDo");
        localStorage.removeItem("tasksDone");
      }
    },
    exit() {
      this.firstTime = false;
    },
  },
  watch: {
    page() {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}&page=${this.page}`
      );
    },
  },
};
</script>

<style></style>

<template>
<div>
  <h1>Things To Do</h1>
  <div id="projects">
    <button :style="{backgroundColor: project.color}" :class="{ white: darkColor(project.color), selected: active(project)}" v-for="project in projects" :key=project.id @click=selectProject(project)>{{project.name}}</button>
  </div>
  <form @submit.prevent="addProject">
      <input type="text" v-model="projectName">
      <swatches-picker v-model="color"/>
      <button type="submit">Add a Project</button>
    </form>
  <div class="todoItems" v-if="project">
    <p v-show="activeItems.length === 0">You are done with all your tasks! Good job!</p>
    <form @submit.prevent="addItem">
      <input type="text" v-model="text">
      <button type="submit">Add an Item</button>
    </form>
    <div class="controls">
      <button @click="showAll()">Show All</button>
      <button @click="showActive()">Show Active</button>
      <button @click="showCompleted()">Show Completed</button>
      <button @click="deleteCompleted()">Delete Completed</button>
    </div>
    <ul>
      <li v-for="item in filteredItems" :key="item.id">
        <label :class="{ item: true, completed: item.completed }">
          {{ item.text }}
          <input type="checkbox" v-model="item.completed" @click="completeItem(item)" />
          <span class="checkmark"></span>
        </label>
        <button @click="deleteItem(item)" class="delete">X</button>
      </li>
    </ul>
  </div>
</div>
</template>

<script>
import axios from 'axios';
import { Swatches } from 'vue-color'
export default {
  name: 'Home',
  components: {
    'swatches-picker': Swatches
  },
  data() {
    return {
      projects: [],
      project: null,
      projectName: '',
      color: "#000",
      items: [],
      text: '',
      show: 'all',
    }
  },
  created() {
    this.getProjects();
  },
  computed: {
    activeItems() {
      return this.items.filter(item => {
        return !item.completed;
      });
    },
    filteredItems() {
      if (this.show === 'active')
        return this.items.filter(item => {
          return !item.completed;
        });
      if (this.show === 'completed')
        return this.items.filter(item => {
          return item.completed;
        });
      return this.items;
    },
  },
  methods: {
    async addProject() {
      try {
        await axios.post("/api/projects", {
          name: this.projectName,
          color: this.color.hex,
        });
        await this.getProjects();
      } catch (error) {
        console.log(error);
      }
    },
    async getProjects() {
      try {
        const response = await axios.get("/api/projects");
        this.projects = response.data;
      } catch (error) {
        console.log(error);
      }
    },
    selectProject(project) {
      this.project = project;
      this.getItems();
    },
    async getItems() {
      try {
        const response = await axios.get(`/api/projects/${this.project._id}/items`);
        this.items = response.data;
      } catch (error) {
        console.log(error);
      }
    },
    async addItem() {
      try {
        await axios.post(`/api/projects/${this.project._id}/items`, {
          text: this.text,
          completed: false
        });
        this.text = "";
        this.getItems();
      } catch (error) {
        console.log(error);
      }
    },
    async completeItem(item) {
      try {
        axios.put(`/api/projects/${this.project._id}/items/${item._id}`, {
          text: item.text,
          completed: !item.completed,
        });
        this.getItems();
      } catch (error) {
        console.log(error);
      }
    },
    async deleteItem(item) {
      try {
        await axios.delete(`/api/projects/${this.project._id}/items/${item._id}`);
        this.getItems();
      } catch (error) {
        console.log(error);
      }
    },
    showAll() {
      this.show = 'all';
    },
    showActive() {
      this.show = 'active';
    },
    showCompleted() {
      this.show = 'completed';
    },
    deleteCompleted() {
      this.items.forEach(item => {
        if (item.completed)
          this.deleteItem(item);
      });
    },
    active(project) {
      return (this.project && project._id === this.project._id);
    },
    darkColor(color) {
      return (this.lightOrDark(color) === 'dark');
    },
    lightOrDark(color) {
      let hex = color;
      if (typeof color === 'object' && color !== null)
        hex = color.hex;

    // Convert it to RGB: http://gist.github.com/983661
      let rgb = +("0x" + hex.slice(1).replace( 
      hex.length < 5 && /./g, '$&$&'));

      const r = rgb >> 16;
      const g = rgb >> 8 & 255;
      const b = rgb & 255;
      
      // HSP (Highly Sensitive Poo) equation from http://alienryderflex.com/hsp.html
      const hsp = Math.sqrt(
      0.299 * (r * r) +
      0.587 * (g * g) +
      0.114 * (b * b)
      );

      // Using the HSP value, determine whether the color is light or dark
      if (hsp>127.5) {

          return 'light';
      } 
      else {

          return 'dark';
      }
    }
  }
}
</script>

<style scoped>
/* List */
ul {
  list-style: none;
}

li {
  background: #fff;
  width: 500px;
  min-height: 40px;
  padding: 10px;
  margin-bottom: 10px;
  font-size: 1em;
  display: flex;
  align-items: center;
}

.delete {
  display: none;
  margin-left: auto;
}

li:hover .delete {
  display: block;
}

label {
  width: 400px;
}

.completed {
  text-decoration: line-through;
}

/* Form */
input[type=checkbox] {
  transform: scale(1.5);
  margin-right: 10px;
}

input[type=text] {
  font-size: 1em;
}

.vc-swatches {
  margin-bottom: 20px;
}

#projects {
  margin-bottom: 20px;
}



button {
  font-family: 'Arvo';
  font-size: 1em;
  border: none;
  background-color: #bbb;
  padding: 5px 10px;
}

button:focus {
  outline: none;
}

button.selected {
  border-bottom: 2px solid #000;
}

.white {
  color: #fff;
}

/* Controls */
.controls {
  margin-top: 20px;
}

/* Custom checkbox
/* Customize the label (the container) */
.item {
  display: block;
  position: relative;
  padding-left: 35px;
  margin-bottom: 12px;
  cursor: pointer;
  font-size: 22px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* Hide the browser's default checkbox */
.item input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

/* Create a custom checkbox */
.checkmark {
  position: absolute;
  top: 0;
  left: 0;
  height: 25px;
  width: 25px;
  background-color: #eee;
}

/* On mouse-over, add a grey background color */
.item:hover input~.checkmark {
  background-color: #ccc;
}

/* When the checkbox is checked, add a blue background */
.item input:checked~.checkmark {
  background-color: #2196F3;
}

/* Create the checkmark/indicator (hidden when not checked) */
.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

/* Show the checkmark when checked */
.item input:checked~.checkmark:after {
  display: block;
}

/* Style the checkmark/indicator */
.item .checkmark:after {
  left: 9px;
  top: 5px;
  width: 5px;
  height: 10px;
  border: solid white;
  border-width: 0 3px 3px 0;
  -webkit-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  transform: rotate(45deg);
}
</style>

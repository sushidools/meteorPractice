<template>
  <div class="container">
    <header>
      <h1>Todo List</h1>
      <h4>You have {{ incompleteCount }} tasks left.</h4>
      <label className="hide-completed">
        <input
          type="checkbox"
          readOnly
          checked="hideCompleted"
          v-model="hideCompleted"
          @click="toggleHideCompleted"
        />
        Hide Completed Tasks
      </label>

      <blaze-template template="loginButtons" tag="span"></blaze-template>
      <template v-if="currentUser">
        <form className="new-task" @submit.prevent="handleSubmit">
          <input
            type="text"
            placeholder="Type to add new tasks"
            v-model="newTask"
          />
        </form>
      </template>
    </header>
    <ul>
      <Task v-for="task in tasks" v-bind:key="task._id" v-bind:task="task" />
    </ul>
  </div>
</template>
 
<script>
import Vue from "vue";
import Task from "./Task.vue";
import { Tasks } from "../api/tasks.js";

export default {
  components: {
    Task
  },
  data() {
    return {
      newTask: "",
      hideCompleted: false
    };
  },
  methods: {
    handleSubmit(event) {
      Meteor.call("tasks.insert", this.newTask);
      // All of this below was before defining methods in tasks.js
      // Tasks.insert({
      //   text: this.newTask,
      //   createdAt: new Date(), //current time
      //   owner: Meteor.userId(), // _id of logged in user
      //   username: Meteor.user().profile.name, // this would work for facebook
      //   // username: Meteor.user().username // this would work for those using email
      //   //how to deal with this? maybe some kind of logic outside of this to check if the user is logged in with facebook or with a created account
      // });

      // Clear form
      this.newTask = "";
      // console.log(Meteor.user().profile.name)
      // I am pretty sure this would only work for facebook
    },
    toggleHideCompleted() {
      this.hideCompleted = !this.hideCompleted;
    }
    
  },
  meteor: {
    $subscribe: {
      // Subscribes to the 'threads' publication with no parameters
      tasks: []
    },
    tasks() {
      let filteredTasks = Tasks.find({}, { sort: { createdAt: -1 } }).fetch();
      if (this.hideCompleted) {
        filteredTasks = filteredTasks.filter(task => !task.checked);
      }
      return filteredTasks;
    },
    incompleteCount() {
      return Tasks.find({ checked: { $ne: true } }).count();
    },
    currentUser() {
      return Meteor.user();
    }
  }
};
</script>
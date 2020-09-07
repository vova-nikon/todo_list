<template>
  <div id="app" class="container">
    <div class="col-sm-10">
      <h1>My Todo List</h1>
      <confirmation :showAlert="showConfirmation"
        :message="confirmationMessage"
        @dismiss-alert="showConfirmation=false"
      ></confirmation>
      <b-row align-h="between" align-v="end">
        <b-col cols="4" style="text-align: left;">
          <b-button type="button" id="task-add" variant="outline-success"
            d-block pill
            v-b-modal.todo-modal
          >
            Add a Todo
          </b-button>
        </b-col>
        <b-col cols="6" class="completed-not-number">
          <p>Not Completed: {{ todosNotCompleted.length }}</p>
          <p>Completed: {{ todosCompleted.length }}</p>
          <p>Total: {{ todosTotal.length }}</p>
        </b-col>
      </b-row>

      <table class="table table-dark table-striped table-hover">
        <thead class="thead-light">
          <tr>
            <th>UID</th>
            <th>Description</th>
            <th>Status</th>
            <th></th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td class="todo-description">{{ todo.description }}</td>
            <td class="todo-status">
              <span v-if="todo.is_completed.length > 0">Completed</span>
              <span v-else>Not Completed</span>
            </td>
            <td class="todo-buttons">
              <!-- <div class="btn-group" role="group"> -->
              <button type="button"
                class="btn update btn-info btn-sm"
                v-b-modal.todo-update-modal
                @click="updateTodo(todo)"
              >
                Update
              </button>
              <button type="button"
                class="btn remove btn-danger btn-sm"
                @click="showDeleteBox(todo)"
              >
                X
              </button>
              <!-- </div> -->
            </td>
          </tr>
          <tr class="tr-empty">
            <td class="todo-uid"></td>
            <td class="todo-description"></td>
            <td class="todo-status">
            </td>
            <td class="todo-buttons">
            </td>
          </tr>
        </tbody>
      </table>

      <b-button class="w-100" variant="outline-danger" pill
        @click="showRemoveAllBox"
      >
        Remove all Todos
      </b-button>

      <b-modal ref="addTodoModal"
        id="todo-modal"
        title="Add a Todo"
        hide-footer
        hide-header-close
        no-close-on-esc
        no-close-on-backdrop
      >
        <b-form class="w-100" @submit="onSubmit" @reset="onReset">
          <b-form-group id="form-description-group"
            label="Description"
            label-for="form-description-input"
          >
            <b-form-input id="form-description-input"
              type="text"
              required
              placeholder="Make up a Todo"
              autofocus
              autocomplete="off"
              v-model="addTodoForm.description"
            >
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-complete-group">
            <b-form-checkbox-group id="form-checks"
              v-model="addTodoForm.is_completed"
            >
              <b-form-checkbox value="true">Completed</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button class="abtn" type="submit" pill variant="outline-success">Add</b-button>
          <b-button type="reset" pill variant="outline-danger">Cancel</b-button>
        </b-form>
      </b-modal>

      <b-modal
        ref="updateTodoModal"
        id="todo-update-modal"
        title="Update"
        hide-footer
        hide-header-close
        no-close-on-esc
        no-close-on-backdrop
      >
        <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
          <b-form-group id="form-update-description-group"
            label="Description:"
            label-for="form-update-description-input"
          >
            <b-form-input
              id="form-update-description-input"
              type="text"
              v-model="updateTodoForm.description"
              required
              autofocus
              autocomplete="off"
            >
            </b-form-input>
          </b-form-group>
          <b-form-group id="form-update-complete-group">
            <b-form-checkbox-group
              v-model="updateTodoForm.is_completed"
              id="form-update-checks"
            >
              <b-form-checkbox value="true">Completed</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
          <b-button class="abtn" type="submit" pill variant="outline-info">Update</b-button>
          <b-button type="reset" pill variant="outline-danger">Cancel</b-button>
        </b-form>
      </b-modal>

    </div>
  </div>
</template>

<script>
import Confirmation from './Confirmation.vue';

export default {
  name: 'todoApp',
  data() {
    return {
      todos: [],
      todosCompleted: [],
      todosNotCompleted: [],
      todosTotal: [],
      addTodoForm: {
        uid: Number,
        description: '',
        is_completed: [],
      },
      updateTodoForm: {
        uid: Number,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '',
      showConfirmation: false,
      deleteBox: '',
      removeAllBox: '',
    };
  },
  components: {
    confirmation: Confirmation,
  },
  methods: {
    countTodos() {
      const todosToCount = this.todos;
      this.todosCompleted = [];
      this.todosNotCompleted = [];
      this.todosTotal = [];
      todosToCount.forEach((element) => {
        if (element.is_completed.length > 0) {
          this.todosCompleted.push(1);
          this.todosTotal.push(1);
        } else {
          this.todosNotCompleted.push(0);
          this.todosTotal.push(0);
        }
      });
    },
    getTodos() {
      localStorage.removeItem('loglevel:webpack-dev-server');
      const storageArray = Object.keys(localStorage);
      storageArray.sort((a, b) => a - b);
      this.todos = [];
      storageArray.forEach((key) => {
        const usedTodo = JSON.parse(localStorage.getItem(key));
        this.todos.push(usedTodo);
        localStorage.removeItem(key);
        usedTodo.uid = this.todos.length;
        localStorage.setItem(usedTodo.uid, JSON.stringify(usedTodo));
      });
      this.countTodos();
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },
    onSubmit(e) {
      e.preventDefault();
      this.$refs.addTodoModal.hide();
      const newTodo = {
        uid: this.todos.length + 1,
        description: this.addTodoForm.description,
        is_completed: this.addTodoForm.is_completed,
      };
      this.todos.push(newTodo);
      localStorage.setItem(Number(newTodo.uid), JSON.stringify(newTodo));
      this.confirmationMessage = `The Todo "${newTodo.description}" has been added`;
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
      this.resetForm();
    },
    onReset() {
      this.$refs.addTodoModal.hide();
      this.todos = [];
      this.getTodos();
    },
    updateTodo(todo) {
      this.updateTodoForm = todo;
    },
    onUpdateSubmit(e) {
      e.preventDefault();
      this.$refs.updateTodoModal.hide();
      localStorage.removeItem(this.updateTodoForm.uid);
      localStorage.setItem(Number(this.updateTodoForm.uid), JSON.stringify(this.updateTodoForm));
      this.confirmationMessage = `The Todo "${this.updateTodoForm.description}" has been updated`;
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
      this.resetForm();
    },
    onUpdateReset() {
      this.$refs.updateTodoModal.hide();
      this.todos = [];
      this.getTodos();
    },
    showDeleteBox(todo) {
      this.deleteBox = '';
      this.$bvModal.msgBoxConfirm(`Delete the Todo "${todo.description}"?`, {
        title: 'Please Confirm',
        size: 'm',
        buttonSize: 'm',
        okVariant: 'outline-danger',
        okTitle: 'YES',
        cancelVariant: 'outline-primary',
        cancelTitle: 'NO',
        footerClass: 'p-2',
        hideHeaderClose: false,
        headerBgVariant: 'danger',
        headerTextVariant: 'light',
        centered: true,
      })
        .then((value) => {
          if (value === true) {
            this.deleteTodo(todo);
          } else {
            this.resetForm();
          }
        });
    },
    deleteTodo(todo) {
      localStorage.removeItem(todo.uid);
      this.confirmationMessage = `The Todo "${todo.description}" has been removed`;
      this.showConfirmation = true;
      this.todos = [];
      this.getTodos();
    },
    showRemoveAllBox() {
      if (this.todos.length > 0) {
        this.RemoveAllBox = '';
        this.$bvModal.msgBoxConfirm('Are you sure you want to remove all your Todos?', {
          title: 'Please Confirm',
          size: 'm',
          buttonSize: 'm',
          okVariant: 'outline-danger',
          okTitle: 'YES',
          cancelVariant: 'outline-primary',
          cancelTitle: 'NO',
          footerClass: 'p-2',
          hideHeaderClose: false,
          headerBgVariant: 'danger',
          headerTextVariant: 'light',
          centered: true,
        })
          .then((value) => {
            if (value === true) {
              this.removeAll();
            } else {
              this.resetForm();
            }
          });
      }
    },
    removeAll() {
      localStorage.clear();
      this.getTodos();
      this.confirmationMessage = 'All Todos have been removed';
      this.showConfirmation = true;
    },
  },
  created() {
    this.getTodos();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
button#task-add {
  margin-top: 20px;
  margin-bottom: 20px;
}
h1 {
  text-align: left;
}
th, td {
  border: 2px solid;
  vertical-align: middle !important;
}
.btn {
  border-radius: 16px;
}
.abtn {
  margin-right: 10px;
}
.remove {
  margin: 3px 5px;
  width: 31px;
}
.update {
  margin: 3px 5px;
}
.todo-uid {
  text-align: center;
  width: 10%;
}
.todo-description {
  text-align: center;
}
.todo-status {
  text-align: center;
  width: 25%;
}
.todo-buttons {
  text-align: center;
  width: 20%;
}
.completed-not-number {
  text-align: right;
}
.tr-empty {
  height: 3.5em;
}
</style>

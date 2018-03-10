<template>
  <div class="container">
    <div class="row">
      <div class="col">
        <div class="card">
          <div class="card-body">
            <h5 class="card-title">My Assignments</h5>
            <button @click="initAddTask" class="btn btn-success" style="padding: 5px">Add New Assignment</button>
            <p class="card-text">
              <table class="table table-bordered table-striped table-responsive" v-if="tasks.length > 0">
                <tbody>
                  <tr>
                    <th>No.</th>
                    <th>Name</th>
                    <th>Description</th>
                    <th>Action</th>
                  </tr>
                  <tr v-for="(task, index) in tasks" :key="index">
                    <td>{{ index + 1 }}</td>
                    <td>{{ task.name }}</td>
                    <td>{{ task.description }}</td>
                    <td>
                      <button @click="initUpdate(index)" class="btn btn-warning btn-xs" style="padding: 8px"><span class="oi oi-pencil" aria-hidden="true"></span></button>
                      <button @click="deleteTask(index)" class="btn btn-danger btn-xs" style="padding: 8px"><span class="oi oi-trash" aria-hidden="true"></span></button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </p>
          </div>
        </div>
      </div>
    </div>

    <div class="modal fade" tabindex="-1" role="dialog" id="add_task_model">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Add New Task</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
          </div>
          <div class="modal-body">
            <div class="alert alert-danger" v-if="errors.length > 0">
              <ul>
                <li v-for="(error, index) in errors" :key="index">{{ error }}</li>
              </ul>
            </div>
            <div class="form-group">
              <label for="names">Name:</label>
              <input type="text" name="name" id="name" placeholder="Task Name" class="form-control" v-model="task.name">
            </div>
            <div class="form-group">
              <label for="description">Description:</label>
              <textarea name="description" id="description" cols="30" rows="5" class="form-control" placeholder="Task Description" v-model="task.description"></textarea>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="button" class="btn btn-primary" @click="createTask">Submit</button>
          </div>
        </div><!-- /.modal-content -->
      </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->

    <div class="modal fade" tabindex="-1" role="dialog" id="update_task_model">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Update Task</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
          </div>
          <div class="modal-body">
            <div class="alert alert-danger" v-if="errors.length > 0">
              <ul>
                <li v-for="(error, index) in errors" :key="index">{{ error }}</li>
              </ul>
            </div>
            <div class="form-group">
              <label for="names">Name:</label>
              <input type="text" name="name" id="name" placeholder="Task Name" class="form-control" v-model="update_task.name">
            </div>
            <div class="form-group">
              <label for="description">Description:</label>
              <textarea name="description" id="description" cols="30" rows="5" class="form-control" placeholder="Task Description" v-model="update_task.description"></textarea>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="button" class="btn btn-primary" @click="updateTask">Submit</button>
          </div>
        </div><!-- /.modal-content -->
      </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->
  </div>
</template>

<script>
  export default {
    data() {
      return {
        task: {
          name: '',
          description: ''
        },
        errors: [],
        tasks: [],
        update_task: []
      }
    },
    mounted() {
      this.readTasks();
    },
    methods: {
      deleteTask(index) {
        let conf = confirm("Do you ready want to delete this task?");
        if(conf === true) {
          axios.delete('/task/' + this.tasks[index].id)
            .then(response => {
              this.tasks.splice(index, 1);
            })
            .catch(error => {
              // do nothing
            });
        }
      },
      initAddTask() {
        $("#add_task_model").modal("show");
      },
      createTask() {
        axios.post('/task', {
          name: this.task.name,
          description: this.task.description
        })
          .then(response => {
            this.reset();
            this.tasks.push(response.data.task);
            $("#add_task_model").modal("hide");
          })
          .catch(error => {
            this.errors = [];
            if(error.response.data.errors && error.response.data.errors.name) {
              this.errors.push(error.response.data.errors.name[0]);
            }
            if(error.response.data.errors && error.response.data.errors.description) {
              this.errors.push(error.response.data.errors.description[0]);
            }
          });
      },
      reset() {
        this.task.name = '';
        this.task.description = '';
      },
      readTasks() {
        axios.get('/task')
          .then(response => {
            this.tasks = response.data.tasks;
          });
      },
      initUpdate(index) {
        this.errors = [];
        $("#update_task_model").modal("show");
        this.update_task = this.tasks[index];
      },
      updateTask() {
        axios.patch('/task/' + this.update_task.id, {
          name: this.update_task.name,
          description: this.update_task.description,
        })
          .then(response => {
            $("#update_task_model").modal("hide");
          })
          .catch(error => {
            this.errors = [];
            if(error.response.data.errors && error.response.data.errors.name) {
              this.errors.push(error.response.data.errors.name[0]);
            }
            if(error.response.data.errors && error.response.data.errors.description) {
              this.errors.push(error.response.data.errors.description[0]);
            }
          });
      }
    }
  }
</script>
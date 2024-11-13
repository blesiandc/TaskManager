<template>
  <div class="about">
    <div class="header">
      <h1>List of Task</h1>
      <v-dialog max-width="500" theme="dark">
        <template v-slot:activator="{ props: activatorProps }">
          <v-btn
            v-bind="activatorProps"
            color="surface-variant"
            text="Add Task"
            variant="flat"
          ></v-btn>
        </template>

        <!-- DIALOG -->
        <template v-slot:default="{ isActive }">
          <v-card title="Dialog">
            <v-card-text>
              <v-sheet class="mx-auto" width="300">
                <v-form @submit.prevent="addTask">
                  <v-container>
                    <v-row>
                      <v-col cols="12" md="6">
                        <v-text-field
                          v-model="newTaskTitle"
                          :rules="titleRules"
                          label="Task Title"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" md="6">
                        <v-text-field
                          v-model="newTaskStatus"
                          :rules="statusRules"
                          label="Status"
                        ></v-text-field>
                      </v-col>
                    </v-row>
                  </v-container>

                  <v-btn class="mt-2" type="submit" block>Submit</v-btn>
                </v-form>
              </v-sheet>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn text="Close Dialog" @click="isActive.value = false"></v-btn>
            </v-card-actions>
          </v-card>
        </template>
      </v-dialog>
    </div>

    <!-- TABLE -->
    <v-data-table theme="dark">
      <thead>
        <tr>
          <th class="text-left">Title</th>
          <th class="text-left">Status</th>
          <th class="text-left">Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in tasks" :key="item.id">
          <td>{{ item.title }}</td>
          <td>{{ item.status }}</td>
          <td class="buttons">
            <v-btn color="surface-variant" @click="toggleStatus(item)">
              {{ item.status === 'pending' ? 'Mark as Completed' : 'Mark as Pending' }}
            </v-btn>
            <v-btn color="surface-variant" @click="deleteTask(item.id)">
              <v-icon icon="mdi-trash-can-outline"></v-icon>
            </v-btn>
          </td>
        </tr>
      </tbody>
    </v-data-table>
  </div>
</template>

<script lang="ts"></script>

<script lang="ts">
import axios from 'axios'

export default {
  name: 'TaskManager',

  data() {
    return {
      titleRules: [(value: string) => !!value || 'You must enter a title name.'],
      statusRules: [(value: string) => !!value || 'You must enter a status.'],
      tasks: [],
      newTaskTitle: '',
      newTaskStatus: 'pending',
    }
  },

  methods: {
    async fetchTasks() {
      try {
        console.log('Fetching tasks...')
        const response = await axios.get('http://localhost:3000/tasks')
        console.log('Response data:', response.data)
        this.tasks = response.data
        console.log('Tasks have been set:', this.tasks)
      } catch (error) {
        console.error('Error fetching tasks:', error)
      }
    },

    async addTask() {
      try {
        const newTask = {
          title: this.newTaskTitle,
          status: this.newTaskStatus,
        }
        await axios.post('http://localhost:3000/tasks', newTask)

        this.newTaskTitle = ''
        this.newTaskStatus = 'pending'

        // Close dialog not working
        this.isActive.value = false

        await this.fetchTasks()
      } catch (error) {
        console.error('Error adding task:', error)
      }
    },

    async toggleStatus(task) {
      const updatedStatus = task.status === 'pending' ? 'completed' : 'pending'
      try {
        await axios.put(`http://localhost:3000/tasks/${task.id}`, {
          status: updatedStatus,
        })

        await this.fetchTasks()
      } catch (error) {
        console.error('Error toggling task status:', error)
      }
    },

    async deleteTask(taskId) {
      try {
        await axios.delete(`http://localhost:3000/tasks/${taskId}`)
        await this.fetchTasks()
      } catch (error) {
        console.error('Error deleting task:', error)
      }
    },
  },

  mounted() {
    this.fetchTasks()
  },
}
</script>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    gap: 20px;
    margin-top: 20%;
  }
  .header {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }
  .buttons {
    display: flex;
    flex-direction: row;
    /* align-items: center; */
    justify-content: space-evenly;
  }
}
</style>

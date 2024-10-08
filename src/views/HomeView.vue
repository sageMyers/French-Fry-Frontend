<script setup>
import { VBtn } from '../../node_modules/vuetify/lib/components/VBtn'
import { VDataTable } from '../../node_modules/vuetify/lib/components/VDataTable'
import { VTextField } from '../../node_modules/vuetify/lib/components/VTextField'
import { VCard } from '../../node_modules/vuetify/lib/components/VCard'
import { VIcon } from '../../node_modules/vuetify/lib/components/VIcon'
import { VToolbar } from '../../node_modules/vuetify/lib/components/VToolbar'
import { VDivider } from '../../node_modules/vuetify/lib/components/VDivider'
import { VDialog } from '../../node_modules/vuetify/lib/components/VDialog'
import { VSnackbar } from '../../node_modules/vuetify/lib/components/VSnackbar'
import courseService from '../services/courseService'
</script>

<script>
export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    search: '',
    snackbar: false,  // For showing toast messages
    snackbarMessage: '',  // Message content for the snackbar
    snackbarColor: '',  // Color of the snackbar (success or error)
    headers: [
      { title: 'Course #', key: 'courseNumber' },
      { title: 'Department', key: 'dept' },
      { title: 'Course Level', key: 'courseLevel' },
      { title: 'Course Hours', key: 'courseHours' },
      { title: 'Course Name', key: 'courseName' },
      { title: 'Course Description', key: 'courseDescription' },
      { title: 'Actions', key: 'actions', sortable: false }
    ],
    courses: [],
    editedIndex: -1,
    editedItem: {
      courseID: 0,
      courseNumber: '',
      dept: '',
      courseLevel: 0,
      courseHours: 0,
      courseName: '',
      courseDescription: ''
    },
    defaultItem: {
      courseID: 0,
      courseNumber: '',
      dept: '',
      courseLevel: 0,
      courseHours: 0,
      courseName: '',
      courseDescription: ''
    }
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New Course' : 'Edit Course'
    }
  },

  watch: {
    dialog(val) {
      if (!val) this.close()
    },
    dialogDelete(val) {
      if (!val) this.closeDelete()
    }
  },

  created() {
    this.initialize()
  },

  methods: {
    initialize() {
      courseService.getAllCourses()
        .then((response) => {
          this.courses = response.data
        })
        .catch((error) => {
          this.showSnackbar(`Failed to fetch courses: ${error.message}`, 'error')
        })
    },

    showSnackbar(message, color) {
      this.snackbarMessage = message
      this.snackbarColor = color === 'success' ? 'green' : 'red'
      this.snackbar = true
    },

    editItem(item) {
      this.editedIndex = this.courses.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.courses.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      let success = await courseService.deleteCourse(this.editedItem.courseID)  
      .then(() => {
        return true
      })
      .catch((e) => {
        console.log(e)
        return false
      });
      if(success === true){
        this.courses.splice(this.editedIndex, 1)
      }
      this.closeDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      if (this.editedIndex > -1) {
        // Update existing course
        Object.assign(this.courses[this.editedIndex], this.editedItem)
        courseService.updateCourse(this.editedItem.courseID, this.editedItem)
          .then(() => {
            this.showSnackbar('Course updated successfully', 'success')
          })
          .catch((error) => {
            this.showSnackbar(`Error updating course: ${error.message}`, 'error')
          })
      } else {
        // Add new course
        courseService.createCourse(this.editedItem)
          .then((response) => {
            this.courses.push(response.data)
            this.showSnackbar('Course added successfully', 'success')
          })
          .catch((error) => {
            this.showSnackbar(`Error creating course: ${error.message}`, 'error')
          })
      }
      this.close()
    }
  }
}
</script>

<template>
  <main>
    <head>
      
    </head>
    <v-data-table
      :headers="headers"
      :items="courses"
      :search="search"
      :sort-by="[{ key: 'courseNumber', order: 'asc' }]"
    >
      <template v-slot:top>
        <v-toolbar flat color="#F7F6FE">
          <v-toolbar-title style="margin-left: 20px;"><b>Courses List</b></v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <div style="width:30%">
            <v-text-field
                v-model="search"
                label="Search"
                prepend-inner-icon="mdi-magnify"
                variant="outlined"
                hide-details
                single-line    
            ></v-text-field>
          </div>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ props }">
              <div style="width:68%">
                <v-btn class="mb-1" color="#FFFFFF" light v-bind="props" style="background-color: #624DE3; float:right; margin-right: 30px;">
                  + Add Course
                </v-btn>
              </div>
            </template>
            <v-card>
              <v-card-title style="text-align: center;">
                <span class="text-h6">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-col>
                    <v-row>
                      <v-text-field
                        class="formField"
                        v-model="editedItem.courseNumber"
                        label="Course Number"
                        required ></v-text-field>
                    </v-row>
                    <v-row>
                      <v-text-field
                        class="formField"
                        v-model="editedItem.dept"
                        label="Department"
                        required ></v-text-field>
                    </v-row>
                    <v-row>
                      <v-text-field
                        class="formField"
                        v-model="editedItem.courseLevel"
                        label="Course Level"
                        required ></v-text-field>
                    </v-row>
                    <v-row>
                      <v-text-field
                        class="formField"
                        v-model="editedItem.courseHours"
                        label="Course Hours"
                        required ></v-text-field>
                    </v-row>
                    <v-row>
                      <v-text-field
                        class="formField"
                        v-model="editedItem.courseName"
                        label="Course Name"
                        required ></v-text-field>
                    </v-row>
                    <v-row>
                      <v-text-field
                        class="formField"
                        v-model="editedItem.courseDescription"
                        label="Course Description"
                        required ></v-text-field>
                    </v-row>
                  </v-col>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <div style="float:right">
                  <v-btn class="formBtnCancel" variant="text" @click="close">
                    Cancel
                  </v-btn>
                  <v-btn class="formBtnConfirm" variant="text" @click="save">
                    Save
                  </v-btn>
                </div>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card style="text-align: center;">
              <v-card-title class="text-h6" style="padding-top:10px"
                >Are you sure you want to delete this item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn class="formBtnConfirm" variant="text" @click="closeDelete"
                  >Cancel</v-btn
                >
                <v-btn
                  class="formBtnCancel"
                  variant="text"
                  @click="deleteItemConfirm"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon icon="mdi-pencil" color="#624DE3" class="me-2" size="small" @click="editItem(item)">
          mdi-pencil
        </v-icon>
        <v-icon size="small" color="#A30D11" @click="deleteItem(item)"> mdi-delete </v-icon>
      </template>
    </v-data-table>

    <!-- Snackbar for displaying success/error messages -->
    <v-snackbar v-model="snackbar" :color="snackbarColor" top right timeout="3000">
      {{ snackbarMessage }}
    </v-snackbar>
  </main>
</template>

<style>
.formField {
    padding-left: 15px;
    padding-right: 15px;
}

.formBtnConfirm {
  color: white;
  background-color:#4D98E3;
  margin:10px;

}

.formBtnCancel {
  color: white;
  background-color: #A30D11;
  margin: 10px;
}
</style>

<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>
    
        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
               
                <v-spacer></v-spacer>
                <v-select
                    class="mr-5 mt-7"
                    label="Urutkan"
                    :items="['Penting', 'Tidak Penting']"
                    single-line
                    dense
                    outlined
                    v-model="sortDir"
                    @change= "sortBy('idPriority',sortDir)"
                ></v-select>
                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table :headers="headers" :items="todos" :search="search" :expanded.sync="expanded" item-key="note" show-expand :custom-sort="customSort">
                <template v-slot:[`item.priority`]="{ item }">
                    <td>
                        <v-chip color="red" label outlined v-if="item.priority == 'Penting'">{{ item.priority }}</v-chip>
                        <v-chip color="green" label outlined v-else-if="item.priority == 'Biasa'">{{ item.priority }}</v-chip>
                        <v-chip color="blue" label outlined v-else-if="item.priority == 'Tidak Penting'">{{ item.priority }}</v-chip>
                    </td>
                </template>
                <template v-slot:[`item.actions`]="{ item }">
                    <v-btn small class="mr-2" @click="editItem(item)">
                        edit
                    </v-btn>
                    <v-btn small @click="deleteItem(item)">
                        delete
                    </v-btn>
                </template>
                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length" class="pa-5 ml-10">
                        <v-row class="ml-5">
                            <h1>Note : </h1>
                        </v-row>
                        <v-row class="ml-10 mt-5">
                            {{ item.note }}
                        </v-row>
                    </td>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                        ></v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting', 'Biasa', 'Tidak Penting']"
                            label="Priority"
                            required
                        ></v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required
                        ></v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">Yes</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>
<script>
export default {
    name: "List",
    data() {
        return {
            expanded: [],
            search: null,
            dialog: false,
            dialogDelete: false,
            headers: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Actions", value: "actions" },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                    idPriority: 3,
                },
                {
                    task: "nongkrong",
                    priority: "Tidak Penting",
                    note: "bersama tman2",
                    idPriority: 1,
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                    idPriority: 2,
                },
            ],
            formTodo: {
                task: null,
                priority: null,
                note: null,
                idPriority: null,
            },
            editedIndex: -1,
            editedItem: {
                task: '',
                priority: '',
                note: '',
                idPriority: '',
            },
            defaultItem: {
                task: '',
                priority: '',
                note: '',
                idPriority: '',
            },
            sortKey: "",
            direction: null,
            sortDir: null,
        };
    },
    computed: {
        formTitle () {
            return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
        },
    },
    watch: {
        dialog (val) {
            val || this.close()
        },
        dialogDelete (val) {
            val || this.closeDelete()
        },
    },
    methods: {
        cancel() {
            this.resetForm();
            this.dialog = false;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
                idPriority: null,
            };
        },
        close () {
            this.dialog = false
            this.$nextTick(() => {
                this.formTodo = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },
        editItem (item) {
            this.editedIndex = this.todos.indexOf(item)
            this.formTodo = Object.assign({}, item)
            this.dialog = true
        },

        deleteItem (item) {
            this.editedIndex = this.todos.indexOf(item)
            this.formTodo = Object.assign({}, item)
            this.dialogDelete = true
        },
        closeDelete () {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.formTodo = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },
        deleteItemConfirm () {
            this.todos.splice(this.editedIndex, 1)
            this.closeDelete()
        },
        save () {
            if (this.editedIndex > -1) {
                if(this.formTodo.priority == "Penting"){
                    this.formTodo.idPriority = 3;
                } else if(this.formTodo.priority == "Biasa"){
                    this.formTodo.idPriority = 2;
                } else {
                    this.formTodo.idPriority = 1;
                }
                Object.assign(this.todos[this.editedIndex], this.formTodo)
            } else {
                if(this.formTodo.priority == "Penting"){
                    this.formTodo.idPriority = 3;
                } else if(this.formTodo.priority == "Biasa"){
                    this.formTodo.idPriority = 2;
                } else {
                    this.formTodo.idPriority = 1;
                }
                this.todos.push(this.formTodo)
            }
            this.close()
        },
        sortBy (id,prop){
            if(prop == 'Penting'){
                this.direction = false;
                this.todos.sort((a,b) => a[id] < b[id] ? 1 : -1)
            } else {
                this.direction = true;
                this.todos.sort((a,b) => a[id] < b[id] ? -1 : 1)
            }
        },
        customSort(items, index, isDesc) {
            isDesc = this.direction;
            items.sort((a, b) => {
                if (index === "date") {
                    if (isDesc) {
                        return b.idPriority - a.idPriority;
                    } else {
                        return a.idPriority - b.idPriority;
                    }
                } else {
                    if (!isDesc) {
                        return a[index] < b[index] ? -1 : 1;
                    } else {
                        return b[index] < a[index] ? -1 : 1;
                    }
                }
            });
            return items;
        },
    },
};
</script>
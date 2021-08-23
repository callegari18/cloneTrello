<template>
    <div class="card">
        <div class="card-header title">
            <h1 class="text-center">Clone Trello</h1>
        </div>
        <!-- <div class="card-body"> -->
            <div class="makes">
                <div class="make-list">
                    <form @submit.prevent="addBoard">
                        <h2 class="text-center">Lista</h2>
                        <div class="form-group">
                            <label for="board-name">
                                <span class="h5">Nome da Lista</span>
                            </label>
                            <input type="text" class="form-control" v-model="board.name" />
                        </div>
                        <button type="submit" class="btn-primary btn-block">
                            <span class="h5">Adicionar / Editar</span>
                        </button>
                    </form>
                </div>
                <div class="make-card">
                    <form @submit.prevent="addTask">
                        <h2 class="text-center">Tarefa</h2>
                        <div class="form-group">
                            <label for="task-name">
                                <span class="h5">Nome da Lista</span>
                            </label>
                            <select class="custom-select" v-model="task.board_name">
                                <option value disabled>Selecione a lista</option>
                                <option v-for="board in boards" v-bind:key="board.id" :value="board.name">{{ board.name }}</option>
                            </select>
                        </div>
                        <div class="form-group">
                            <label for="task-name">
                                <span class="h5">Nome da Tarefa</span>
                            </label>
                            <input type="text" class="form-control" v-model="task.name" />
                        </div>
                        <div class="form-group">
                            <label for="task-decription">
                                <span class="h5">Descrição da Tarefa</span>
                            </label>
                            <textarea class="form-control" v-model="task.description"></textarea>
                        </div>
                        <button type="submit" class=" btn-primary btn-block">
                            <span class="h5">Adicionar / Editar</span>
                        </button>
                    </form>
                </div>
            </div>


            
        <!-- </div> -->
        <div class="frame" >
               
               <div class="list-card" v-for="board in boards" v-bind:key="board.id">
                    <div class="title-Card" >
                        <h3 class="alert alert-dark">{{ board.name }}</h3>
                    </div>
                    <div class="body-card" v-for="board in boards" v-bind:key="board.id">

                        <Board :id="board.id">
                            <div v-for="task in tasks" v-bind:key="task.id">
                                <div v-if="board.name == task.board_name">
                                    <Task :id="task.id" draggable="true">
                                        <h4>{{ task.name }}</h4>
                                        <p>{{ task.description }}</p>
                                        <div class="list-button">
                                            <button @click="editTask(task)" class="btn btn-warning btn-sm ">
                                                <span class="h6">Editar</span>
                                            </button>
                                            <button @click="deleteTask(task.id)" class="btn btn-danger btn-sm ">
                                                <span class="h6">Deletar</span>
                                            </button>
                                        </div>
                                    </Task>
                                </div>
                            </div>
                        </Board>


                    </div>
                    <div class="list-button">
                        <button @click="editBoard(board)" class="btn btn-warning ">
                            <span class="h5">Edit</span>
                        </button>
                        <button @click="deleteBoard(board.id)" class="btn btn-danger ">
                            <span class="h5">Delete</span>
                        </button>
                    </div>
                </div>
                
                
               

            </div>
        </div>
    </div>
</template>

<script>
//  get current user's id from meta tag
Vue.prototype.$user_id = document
    .querySelector("meta[name='user-id']")
    .getAttribute("content");

export default {
    data() {
        return {
            boards: [],
            board: {
                id: "",
                name: ""
            },
            board_id: "",

            tasks: [],
            task: {
                id: "",
                board_name: "",
                name: "",
                description: ""
            },
            task_id: "",
            edit: false
        };
    },
    created() {
        //  get all boards
        this.fetchBoards();
        //  get all tasks
        this.fetchTasks();
    },
    methods: {
        //  board data handling

        fetchBoards() {
            fetch("api/boards/" + this.$user_id)
                .then(res => res.json())
                .then(res => {
                    this.boards = res.data;
                });
        },
        deleteBoard(id) {
            if (confirm("Are you sure?")) {
                fetch("api/board/" + id + "/" + this.$user_id, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        this.fetchBoards();
                    })
                    .catch(error => console.log(error));
            }
        },
        addBoard() {
            if (this.edit === false) {
                fetch("api/board/" + this.$user_id, {
                    method: "post",
                    body: JSON.stringify(this.board),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.board.name = "";
                        this.fetchBoards();
                    })
                    .catch(error => console.log(error));
            } else {
                fetch("api/board/" + this.$user_id, {
                    method: "put",
                    body: JSON.stringify(this.board),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.board.name = "";
                        alert("Board Updated. Wait for page reload");
                        window.location.reload();
                    })
                    .catch(error => console.log(error));
            }
        },
        editBoard(board) {
            this.edit = true;
            this.board.id = board.id;
            this.board.board_id = board.id;
            this.board.name = board.name;
            alert("You can edit this in the Board section above.");
            window.scrollTo(0, 0);
        },

        //  task data handling

        fetchTasks() {
            fetch("api/tasks/" + this.$user_id)
                .then(res => res.json())
                .then(res => {
                    this.tasks = res.data;
                });
        },
        deleteTask(id) {
            if (confirm("Are you sure?")) {
                fetch("api/task/" + id + "/" + this.$user_id, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        alert("Task Deleted. Wait for page reload");
                        window.location.reload();
                        this.fetchTasks();
                    })
                    .catch(error => console.log(error));
            }
        },
        addTask() {
            if (this.edit === false) {
                fetch("api/task/" + this.$user_id, {
                    method: "post",
                    body: JSON.stringify(this.task),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.task.board_name = "";
                        this.task.name = "";
                        this.task.description = "";
                        alert("Task added. Wait for page reload");
                        window.location.reload();
                    })
                    .catch(error => console.log(error));
            } else {
                fetch("api/task/" + this.$user_id, {
                    method: "put",
                    body: JSON.stringify(this.task),
                    headers: {
                        "Content-Type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.task.board_name = "";
                        this.task.name = "";
                        this.task.description = "";
                        alert("Task Updated. Wait for page reload");
                        window.location.reload();
                    })
                    .catch(error => console.log(error));
            }
        },
        editTask(task) {
            this.edit = true;
            this.task.id = task.id;
            this.task.task_id = task.id;
            this.task.board_name = task.board_name;
            this.task.name = task.name;
            this.task.description = task.description;
            alert("You can edit this in the Task section above.");
            window.scrollTo(0, 0);
        }
    }
};
</script>

<style lang="scss" scoped>
    * {
    margin: 0;
    padding: 0;
}
.area-total {
    width: 100%;
    height: 100vh;
}
.windows{
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 20px;
}
.makes {
display: flex;
justify-content: space-around;
width: 850px;
}
.frame {
display: flex;
justify-content: start;

}

.title {
display: flex;
justify-content: center;
align-items: center;
background-color: white;
width: 100%;
height: 60px;
}
.make-list {
background-color: blue;
width: 350px;
height: 400px;
border-radius: 10px;
}
.make-card {
background-color: yellow;
width: 350px;
height: 400px;
border-radius: 10px;
}
.list-card {
background-color: rgb(99, 99, 99);
width: 250px;
height: 400px;
border-radius: 10px;
margin: 10px;
}
.title-Card {
    text-align: center;
    padding: 5px;
}
.list-button {
    display: flex;
    padding: 5px;
    justify-content: space-around;
}
.btn {
    width: 40%;
    height: 30px;
}
.body-card {
    background-color: gray;
    border-radius: 5px;
    margin: 5px;
    padding: 5px;
}

</style>
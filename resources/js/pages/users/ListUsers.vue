<script setup>
import {ref, onMounted, reactive} from "vue";

let users = ref([]);

const getUsers =() => {
    axios.get('/api/users')
        .then((response) => {
            users.value = response.data;
    })
}
const form = reactive({
    name: '',
    email: '',
    password: '',
});

const createUser = () => {
    axios.post('/api/users', form)
        .then((response) => {
            users.value.unshift(response.data)
            form.name = '';
            form.email = '';
            form.password = '';
            $('#createUserModal').modal('hide');
        });
}

onMounted(() => {
    getUsers();
})
</script>
<template>
    <div class="content-header">
        <div class="container-fluid">
            <div class="row mb-2">
                <div class="col-sm-6">
                    <h1 class="m-0">Users</h1>
                </div>
                <div class="col-sm-6">
                    <ol class="breadcrumb float-sm-right">
                        <li class="breadcrumb-item"><a href="#">Home</a></li>
                        <li class="breadcrumb-item active">Users</li>
                    </ol>
                </div>
            </div>
        </div>
    </div>


    <div class="content">
        <div class="container-fluid">
            <button type="button" class="mb2 btn btn-primary" data-toggle="modal" data-target="#createUserModal">
                    Add New User
            </button>
            <div class="card">
                <div class="card-body">
                    <div class="table-bordered">
                        <table class="table">
                            <thead>
                            <tr>
                                <th style="width: 10px">#</th>
                                <th>Name</th>
                                <th>Email.</th>
                                <th>Registered Date</th>
                                <th>Role</th>
                                <th>Options</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-for="(user, index) in users" :key="user.id">
                                <td>{{ index+1 }}</td>
                                <td>{{ user.name }}</td>
                                <td>{{ user.email }}</td>
                                <td>12 May 2017</td>
                                <td>12 May 2017</td>
                                <td>12 May 2017</td>

                            </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="createUserModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="exampleModalLabel">Add New User</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form autocomplete="off">
                        <div class="form-group">
                            <label for="name">Name</label>
                            <input v-model="form.name" type="text" class="form-control" id="name" aria-describedby="" placeholder="Enter full name">
                        </div>

                        <div class="form-group">
                            <label for="name">Email</label>
                            <input v-model="form.email" type="text" class="form-control" id="email" aria-describedby="" placeholder="Enter email">
                        </div>

                        <div class="form-group">
                            <label for="name">Password</label>
                            <input v-model="form.password" type="text" class="form-control" id="name" aria-describedby="" placeholder="Enter password">
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    <button @click="createUser" type="button" class="btn btn-primary">Save changes</button>
                </div>
            </div>
        </div>
    </div>
</template>

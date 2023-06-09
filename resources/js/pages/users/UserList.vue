<script setup>
import axios from "axios";
import { ref, onMounted, reactive, watch } from "vue";
import { Form, Field } from 'vee-validate';
import * as yup from 'yup';
import { useToaster } from "../../toaster";
import UserListItem from "./UserListItem.vue";
import { debounce } from "lodash";
import { Bootstrap4Pagination } from "laravel-vue-pagination";

const toastr = useToaster();
const users = ref({'data' : []});
const editing = ref(false);
const formValues = ref();
const form = ref(null);


const getUsers = (page = 1) => {
    axios.get(`/api/users?page=${page}`)
        .then((response) => {
            users.value = response.data;
    })
}

const createUserSchema = yup.object({
    name: yup.string().required(),
    email: yup.string().email().required(),
    password: yup.string().required().min(8),
});

const editUserSchema = yup.object({
    name: yup.string().required(),
    email: yup.string().email().required(),
    password: yup.string().when((password, schema) => {
        return password[0] !== undefined ? schema.required().min(8) : schema
    }),
});

const createUser = (values, { resetForm, setErrors }) => {
    axios.post('/api/users', values)
        .then((response) => {
            users.value.unshift(response.data)
            getUsers();

            $('#userFormModal').modal('hide');
            resetForm();
            toastr.success('user created successfully!');
        })
        .catch((error) => {
            if (error.response.data.errors) {
                setErrors(error.response.data.errors);
            }
        });
}

const addUser = () => {
    editing.value = false;
    $('#userFormModal').modal('show');
}

const editUser = (user) => {
    editing.value = true;
    form.value.resetForm();
    $('#userFormModal').modal('show');
    formValues.value = {
        id: user.id,
        name: user.name,
        email: user.email,
    };
};

const updateUser = (values, { setErrors }) => {
    axios.put('/api/users/' + formValues.value.id, values)
        .then((response) => {
            const index = users.value.findIndex(user => user.id === response.data.id);
            users.value[index] = response.data;
            $('#userFormModal').modal('hide');
            toastr.success('user updated successfully!');
        }).catch((error) => {
            setErrors(error.response.data.errors);
    });
}

const handleSubmit = (values, actions) => {
    if (editing.value) {
        updateUser(values, actions);
    } else {
        createUser(values, actions);
    }
}

const userDeleted = (userId) => {
    users.value = users.value.filter(user => user.id !== userId);
}

const searchQuery = ref(null);

const search = () => {
    axios.get('/api/users/search', {
        params: {
            query: searchQuery.value
        }
    })
        .then(response => {
            users.value = response.data;
        })
        .catch(error => {
            console.log(error);
        });
};

watch(searchQuery, debounce(() => {
    search();
}, 300));

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
            <div class="d-flex justify-content-between">
                <button @click="addUser" type="button" class="mb2 btn btn-primary">
                        Add New User
                </button>
                <div>
                    <input type="text" v-model="searchQuery" class="form-control" placeholder="Search..." />
                </div>
            </div>

            <div class="card">
                <div class="card-body">
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
                            <tbody v-if="users.data.length > 0">
                                <UserListItem v-for="(user, index) in users.data"
                                         :key="user.id"
                                         :user=user
                                         :index=index
                                         @edit-user="editUser"
                                         @user-deleted="userDeleted"
                           />

                            </tbody>
                            <tbody v-else>
                            <tr>
                                <td class="text-center" colspan="6">No results found</td>
                            </tr>
                            </tbody>
                        </table>
                </div>
            </div>
            <Bootstrap4Pagination :data="users" @pagination-change-page="getUsers" />
        </div>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="userFormModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="exampleModalLabel">
                        <span v-if="editing">Edit User</span>
                        <span v-else>Add New User</span>
                    </h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <Form ref="form" @submit="handleSubmit" :validation-schema="editing ? editUserSchema : createUserSchema"
                      v-slot="{ errors }" :initial-values="formValues">
                    <div class="modal-body">
                            <div class="form-group">
                                <label for="name">Name</label>
                                <Field name="name" type="text" class="form-control" :class="{'is-invalid': errors.name}" id="name"
                                       aria-describedby="nameHelp" placeholder="Enter full name" />
                                <span class="invalid-feedback">{{ errors.name }}</span>
                            </div>

                            <div class="form-group">
                                <label for="name">Email</label>
                                <Field name="email" type="email" class="form-control" :class="{'is-invalid': errors.email}" id="email"
                                       aria-describedby="nameHelp" placeholder="Enter email" />
                                <span class="invalid-feedback">{{ errors.email }}</span>
                            </div>

                            <div class="form-group">
                                <label for="name">Password</label>
                                <Field name="password" type="password" class="form-control" :class="{'is-invalid': errors.password}" id="password"
                                       aria-describedby="nameHelp" placeholder="Enter password" />
                                <span class="invalid-feedback">{{ errors.password }}</span>
                            </div>
                    </div>

                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="submit" class="btn btn-primary">Save changes</button>
                    </div>
                </Form>
            </div>
        </div>
    </div>


</template>

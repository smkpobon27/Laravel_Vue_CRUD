<template>
    <div class="customer">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <div class="row">
                            <div class="col-md-6">
                                Customer
                            </div>
                            <div class="col-md-2"></div>
                            <div class="col-md-4">
                                <button type="button" class="btn btn-primary float-right" @click="create"><i class="fas fa-plus"></i> Add New</button>
                                <button type="button" class="btn btn-info" @click="reload"><i class="fas fa-sync"></i> Reload</button>
                            </div>
                        </div>
                    </div>

                    <div class="card-body">
                        <div class="row">
                            <div class="col-md-1">
                                <p><strong>Search By:</strong></p>
                            </div>
                            <div class="col-md-3">
                               <select class="form-select" v-model="queryField" aria-label="Default select example">
                                    <option value="name">Name</option>
                                    <option value="email">Email</option>
                                    <option value="phone">Phone</option>
                                    <option value="address">Address</option>
                                    <option value="total">Total</option>
                                </select>
                            </div>
                            <div class="col-md-8">
                                <div class="mb-3">
                                    <input type="text" v-model="query" class="form-control" id="search" placeholder="Search here">
                                </div>
                            </div>
                        </div>
                        <table class="table table-bordered table-striped small table-sm">
                            <thead>
                                <tr>
                                <th scope="col">#</th>
                                <th scope="col">Name</th>
                                <th scope="col">Email</th>
                                <th scope="col">Phone</th>
                                <th scope="col">Address</th>
                                <th scope="col">Total</th>
                                <th scope="col" class="text-center">Action</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-show="customers.length" v-for="(customer, index) in customers" :key="customer.id">
                                    <td>{{index + 1}}</td>
                                    <td>{{customer.name}}</td>
                                    <td>{{customer.email}}</td>
                                    <td>{{customer.phone}}</td>
                                    <td>{{customer.address}}</td>
                                    <td>{{customer.total}}</td>
                                    <td class="text-center">
                                        <button type="button" class="btn btn-info btn-sm"><i class="fas fa-eye"></i></button>
                                        <button type="button" @click="edit(customer)" class="btn btn-primary btn-sm"><i class="fas fa-edit"></i></button>
                                        <button type="button" class="btn btn-danger btn-sm"><i class="fas fa-trash-alt"></i></button>
                                    </td>
                                </tr>
                                <tr v-show="!customers.length">
                                    <td colspan="7">
                                        <div class="alert alert-danger text-center">
                                            Sorry! No data found.
                                        </div>
                                    </td>
                                </tr>
                            </tbody>
                            
                        </table>
                        <pagination v-if="pagination.last_page > 1"
                                        :pagination="pagination"
                                        :offset="5"
                                        @paginate="query===''? getData() : searchData()"
                                    ></pagination>
                    </div>
                </div>
            </div>
        </div>

        <div class="modal fade" id="customerModalLong" tabindex="-1" role="dialog" aria-labelledby="customerModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="customerModalLongTitle">{{ editMode ? "Edit" : "Add New" }} Customer</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="editMode ? update() : store()" @keydown="form.onKeydown($event)">
                    <div class="modal-body">
                        <AlertError :form="form" />
                        <!-- <AlertErrors :form="form" /> -->
                        <!-- <AlertSuccess :form="form" message="Your changes have beend saved!" /> -->

                        <div class="mb-3">
                            <label for="name" class="form-label">Name</label>
                            <input id="name" v-model="form.name" type="text" name="name" class="form-control">
                            <HasError :form="form" field="name" />
                        </div>
                        <div class="mb-3">
                            <label for="email" class="form-label">Email</label>
                            <input id="email" v-model="form.email" type="email" name="email" class="form-control">
                            <HasError :form="form" field="email" />
                        </div>
                        <div class="mb-3">
                            <label for="phone" class="form-label">Phone</label>
                            <input id="phone" v-model="form.phone" type="tel" name="phone" class="form-control">
                            <HasError :form="form" field="phone" />
                        </div>
                        <div class="mb-3">
                            <label for="address" class="form-label">Address</label>
                            <textarea id="address" v-model="form.address" type="tel" name="address" class="form-control">
                                </textarea>
                            <HasError :form="form" field="address" />
                        </div>
                        <div class="mb-3">
                            <label for="total" class="form-label">Total</label>
                            <input id="total" v-model="form.total" type="number" name="total" class="form-control">
                            <HasError :form="form" field="total" />
                        </div>

                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="submit" class="btn btn-primary" data-dismiss="modal">Save Changes</button>
                    </div>
                </form>
                </div>
            </div>
        </div>

        <vue-progress-bar></vue-progress-bar>
         <vue-snotify></vue-snotify>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                editMode : false,
                customers : [],
                pagination: {
                    current_page: 1,
                },
                query: "",
                queryField: "name",
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    phone: '',
                    address: '',
                    total: ''
                }),
            }
        },
        mounted() {
            console.log('Component mounted.');
            this.getData();
            
        },
        watch:{
            query: function(newQ, oldQ){
                if(newQ === ''){
                    this.getData();
                }else{
                    this.searchData();
                }
            }
        },

        methods:{
            getData : function(){
                 this.$Progress.start();
                axios.get('/api/customers?page='+this.pagination.current_page)
                    .then(response => {
                        this.customers = response.data.data;
                        this.pagination = response.data.meta;
                        this.$Progress.finish();
                        // console.log(response);
                        // console.log(this.customers);
                    })
                    .catch(e => {
                        console.log(e); 
                        this.$Progress.fail();   
                    })
            },

            searchData : function(){
                this.$Progress.start();
                axios.get('/api/search/customers/'+this.queryField+'/'+this.query+'?page='+this.pagination.current_page)
                    .then(response =>{
                        this.customers = response.data.data;
                        this.pagination = response.data.meta;
                        this.$Progress.finish();
                    })
                    .catch(e =>{
                        this.$Progress.fail();
                    })
            },

            reload : function(){
                this.getData();
                this.query = '';
                this.queryField = 'name';
                this.$snotify.success('Data Refresh successfully', 'Success');
            },

            create : function(){
                this.editMode = false;
                this.form.reset();
                this.form.clear();
                $('#customerModalLong').modal('show');
            },

            store : function(){
                this.$Progress.start();
                this.form.busy = true;
                this.form.post('/api/customers')
                    .then(response =>{
                        this.getData();
                         $('#customerModalLong').modal('hide');
                         if(this.form.successful){
                             this.$Progress.finish();
                             this.$snotify.success('Customer Added Successfully.', 'Success');
                         }else{
                             this.$Progress.fail();
                             this.$snotify.error('Something went wrong!', 'Error');
                         }
                    })
                    .catch(e =>{
                        this.$Progress.fail();
                        console.log(e);
                    })
            },

            edit : function(customer){
                this.editMode = true;
                this.form.reset();
                this.form.clear();
                this.form.fill(customer);
                $('#customerModalLong').modal('show');

            },

            update : function(){
                this.$Progress.start();
                this.form.busy = true;
                this.form.put('/api/customers/'+this.form.id)
                    .then(response =>{
                        this.getData();
                         $('#customerModalLong').modal('hide');
                         if(this.form.successful){
                             this.$Progress.finish();
                             this.$snotify.success('Customer Updated Successfully.', 'Success');
                         }else{
                             this.$Progress.fail();
                             this.$snotify.error('Something went wrong!', 'Error');
                         }
                    })
                    .catch(e =>{
                        this.$Progress.fail();
                        console.log(e);
                    })
            }

        }
    }
</script>

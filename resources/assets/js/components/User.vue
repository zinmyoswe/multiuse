<template>
    <div class="container">
         <div class="row" v-if="$gate.isAdminOrAuthor()">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Management Table</h3>

                <div class="card-tools">
                  <div class="input-group input-group-sm" style="width: 150px;">
                    

                    
                      <button type="button" class="btn btn-block btn-primary btn-flat" @click="newModal"><i class="fas fa-user-plus fa-fw"></i> Add New User</button>
                    
                  </div>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Status</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users.data" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td><span class="tag tag-success">Approved</span></td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                      <a href="#" @click="editModal(user)">
                        <i class="fa fa-edit blue"></i>
                      </a>/
                      <a href="#" @click="deleteUser(user.id)">
                        <i class="fa fa-trash red"></i>
                      </a>
                    </td>
                  </tr>
                 
                </table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div><!-- /.row -->

        <div v-if="!$gate.isAdminOrAuthor()">
          <not-found></not-found>
        </div>
        <!-- Modal -->
<div class="modal fade" id="exampleModalCenter" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" v-show="editmode" id="exampleModalCenterTitle">Update User Information</h5>
        <h5 class="modal-title" v-show="!editmode" id="exampleModalCenterTitle">Create New User</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <form @submit.prevent="editmode? updateUser(): createUser()">
      <div class="modal-body">
            <div class="form-group">
      <input v-model="form.name" type="text" name="name"
        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }" placeholder="Name">
      <has-error :form="form" field="name"></has-error>
        </div>

            <div class="form-group">
      <input v-model="form.email" type="email" name="email"
        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }" placeholder="email">
      <has-error :form="form" field="email"></has-error>
        </div>

          <div class="form-group">
      <textarea v-model="form.bio" name="bio"
        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }" placeholder="bio"></textarea>
      <has-error :form="form" field="bio"></has-error>
        </div>

         <div class="form-group">
      <select v-model="form.type" name="type"
        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
          <option value="">Select User Role</option>
          <option value="admin">Admin</option>
          <option value="user">User</option>
          <option value="author">Author</option>
        </select>
      <has-error :form="form" field="bio"></has-error>
        </div>

         <div class="form-group">
      <input v-model="form.password" type="password" name="password"
        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
      <has-error :form="form" field="password"></has-error>
    </div>
      </div><!-- modal end -->
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
        <button v-show="editmode" type="submit" class="btn btn-warning">Update</button>
        <button v-show="!editmode" type="submit" class="btn btn-primary">Save</button>
      </div>
    </form>
    </div>
  </div>
</div>
<!-- End Modal -->
    </div>
</template>

<script>
    export default {
        data(){
          return{
            editmode : false,
            users : {},
            form: new Form({
                id: '',
                name: '',
                email: '',
                password: '',
                type: '',
                bio: '',
                photo: ''
            })
          }
        },
        methods:{
          getResults(page = 1) {
          axios.get('api/user?page=' + page)
              .then(response => {
                this.users = response.data;
              });
            },
          updateUser(){
            this.$Progress.start();
               this.form.put('api/user/'+this.form.id)
               .then(()=>{
                $('#exampleModalCenter').modal('hide');
                 Swal.fire(
                        'Updated!',
                        'Your Information has been updated.',
                        'success'
                      )
                 this.$Progress.finish();
                 Fire.$emit('AfterCreate');
              })
               .catch(()=>{
                this.$progress.fail();
              });
          },
          editModal(user){
            this.editmode = true;
            this.form.reset();
            $('#exampleModalCenter').modal('show');
            this.form.fill(user);
          },
          newModal(){
            this.editmode = false;
            this.form.reset();
            $('#exampleModalCenter').modal('show');
          },
          deleteUser(id){
            Swal.fire({
            title: 'Are you sure?',
            text: "You won't be able to revert this!",
            type: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#3085d6',
            cancelButtonColor: '#d33',
            confirmButtonText: 'Yes, delete it!'
          }).then((result) => {
              //Send Request to the server 
            if (result.value) {
              this.form.delete('api/user/'+id).then(()=>{
                    
                      Swal.fire(
                        'Deleted!',
                        'Your file has been deleted.',
                        'success'
                      )
                   Fire.$emit('AfterCreate');
              }).catch(()=>{
                  Swal.fire(
                        'Failed!',
                        'Something Wrong.',
                        'Warning'
                      );
              })
            }
          
          })
          },
          loadUsers(){
            if(this.$gate.isAdminOrAuthor()){
            axios.get("api/user").then(({data})=>(this.users=data));
            }
          },
          createUser(){
            this.$Progress.start();
            this.form.post('api/user')
            .then(()=>{
                  Fire.$emit('AfterCreate');
                $('#exampleModalCenter').modal('hide')
                Toast.fire({
                  type: 'success',
                  title: 'User Created in successfully'
                })
                this.$Progress.finish();
            })
            .catch(()=>{

            })
            
          }
        },
        created() {
            Fire.$on('searching',()=>{
              let query = this.$parent.search;
              axios.get('api/findUser?q=' + query)
              .then((data) =>{
                this.users = data.data
              })
              .catch(() => {

              })
            })

            this.loadUsers();
            Fire.$on('AfterCreate',()=>{
              this.loadUsers();
            });
            // setInterval(()=>this.loadUsers(),3000);
        }
    }
</script>

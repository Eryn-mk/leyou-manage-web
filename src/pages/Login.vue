<template>
  <v-app>
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md4>
            <v-card class="elevation-12">
              <v-toolbar dark color="primary">
                <v-toolbar-title>Welcome to LeYou Management System</v-toolbar-title>
                <v-spacer></v-spacer>
              </v-toolbar>
              <v-card-text>
                <v-form>
                  <v-text-field prepend-icon="person" v-model="username" label="Username" type="text"/>
                  <v-text-field
                    prepend-icon="lock"
                    v-model="password"
                    label="Password"
                    id="password"
                    :append-icon="e1 ? 'visibility' : 'visibility_off'"
                    :append-icon-cb="() => (e1 = !e1)"
                    :type="e1 ? 'text' : 'password'"
                 ></v-text-field>
                </v-form>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="doLogin">Log in</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <v-dialog v-model="dialog" width="300px">
      <v-alert icon="warning" color="error" :value="true">
      Username and Password cannot be empty
      </v-alert>
    </v-dialog>
  </v-app>
</template>

<script>
export default {
  data: () => ({
    username: "",
    password: "",
    dialog: false,
    e1:false,
    backPath:'',
  }),
  beforeRouteEnter(to,from,next){
    next(vm => {
      vm._data.backPath = from.path;
    });
  },
  methods: {
    doLogin() {
      if (!this.username || !this.password) {
        this.dialog = true;
        return false;
      }
      const form ={};
      form.username = this.username;
      form.password = this.password;

      this.$http.post("/auth/adminLogin", this.$qs.stringify(form)).then(resp =>{
        if (resp.status === 204){
          //页面跳转
          if (this.backPath === "/"){
            this.$router.push("/item/category");
          } else {
            this.$router.push(this.backPath);
          }
        }
      }).catch(() => {
        this.$message.error("Wrong username or password");
      });
      // console.log(this.username + " ... " + this.password);
      // this.$router.push("/");
    }
  }
};
</script>

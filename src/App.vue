<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />

    <div class="content">
      <b-card title="Card Title" no-body>
        <b-card-header header-tag="nav">
          <b-nav card-header tabs>
            <b-nav-item
              v-if="authenticated"
              to="/login"
              v-on:click.native="logout()"
              replace
              >Logout</b-nav-item
            >
            <b-nav-item   v-if="authenticated" to="/dashboard" exact exact-active-class="active"
              >Dashboard</b-nav-item
            >
            <b-nav-item   v-if="authenticated" to="/create" exact exact-active-class="active"
              >Create</b-nav-item
            >
            <b-nav-item  v-if="authenticated" to="/update" exact exact-active-class="active"
              >Edit</b-nav-item
            >
          </b-nav>
        </b-card-header>

        <b-card-body class="text-center">
          <router-view @authenticated="setAuthenticated" />
        </b-card-body>
      </b-card>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
        authenticated: false,
        mockAccount: {
            username: "user",
            password: "pass"
        }
    }
},
  mounted() {
    if(!this.authenticated) {
        this.$router.replace({ name: "login" });
    }
},
  methods: {
    setAuthenticated(status) {
      this.authenticated = status;
    },
    logout() {
      this.authenticated = false;
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.create {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 40px;
}

.content {
  margin-top: 60px;
  margin-bottom: 60px;
  width: 70%;
  margin-left: 15%;
}
</style>

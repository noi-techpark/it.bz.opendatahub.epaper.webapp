<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div id="app">
    <div v-if="authenticated" class="topbar">
      <img class="logo" alt="Vue logo" src="./assets/logo_bn.svg" />
      <div class="routerHeader">
        <h3>{{ siteName }}</h3>
        <h4 v-if="currentRouteName">{{ currentRouteName }}</h4>
      </div>
      <b-nav class="navbar">
        <b-nav-item
          v-for="route in routes"
          :to="route.link"
          :key="route.link"
          exact
          exact-active-class="active"
          >{{ route.name }}</b-nav-item
        >
        <div class="userBox">
          Hi, {{ username }}
          <b-nav-item v-if="username" v-on:click="logout()">Logout</b-nav-item>
        </div>
      </b-nav>
    </div>
    <div v-else>
      <img class="bigLogo" alt="Vue logo" src="./assets/logo.png" />
      <h3>{{ siteName }}</h3>
    </div>

    <div class="content" v-if="dataLoaded || !authenticated">
      <b-card title="Card Title" no-body>
        <b-card-body class="text-center">
          <router-view />
        </b-card-body>
      </b-card>
    </div>
    <b-spinner v-else></b-spinner>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      username: null,
      routes: [
        { name: "Displays", link: "/displays" },
        { name: "Templates", link: "/templates" },
      ],
      siteName: "eInk Display management site",
    };
  },
  computed: {
    currentRouteName() {
      if (this.routes.some((e) => e.name === this.$route.name)) {
        return this.$route.name;
      }
      return "";
    },
    authenticated() {
      return this.$store.state.authenticated;
    },
    dataLoaded() {
      return this.$store.state.dataLoaded;
    },
  },
  mounted() {
    if (this.authenticated) {
      this.$keycloak.loadUserInfo().then((info) => {
        this.username = info.name;
      });
    }

    if (!localStorage.getItem("noPrivacyMessage")) {
      this.showPrivacyToast();
    }
  },
  methods: {
    logout() {
      const redirectPath = window.location.origin + "/";
      this.$keycloak.logout({ redirectUri: redirectPath });
    },
    showPrivacyToast() {
      const id = `privacy-toast`;
      const $okButton = this.$createElement(
        "b-button",
        {
          on: {
            click: () => {
              this.$bvToast.hide(id);
              localStorage.setItem("noPrivacyMessage", true);
            },
          },
          class: "toastCloseButton",
        },
        "OK, don't show again"
      );

      // Create the toast
      this.$bvToast.toast(
        [
          "This site stores information regarding user identification. This information is necessary for the site to function.",
          $okButton,
        ],
        {
          id: id,
          title: "",
          toastClass: "privacyToast",
          toaster: "b-toaster-bottom-center",
          variant: "info",
          noAutoHide: true,
          noCloseButton: true,
        }
      );
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}

.content {
  margin-top: 20px;
}

.topbar {
  height: 75px;
}

.navbar {
  float: right;
}

.logo {
  margin-left: 20px;
  margin-top: 8px;
  float: left;
}

.bigLogo {
  margin-bottom: 25px;
}

.routerHeader {
  float: left;
  text-align: left;
  margin-left: 30px;
}

.userBox {
  border-radius: 8px;
  border: 2px solid #aaaaaa;
}

.privacyToast {
  padding-bottom: 35px;
}

.toastCloseButton {
  float: right;
  margin-top: 20px;
}
</style>

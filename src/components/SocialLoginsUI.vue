<script setup>
import { ref } from "vue";
import firebaseConfig from "../firebaseConfig";
import firebase from "firebase/compat/app";
import * as firebaseui from "firebaseui";
import "firebaseui/dist/firebaseui.css";
import { getAuth, signOut } from "firebase/auth";

firebase.initializeApp(firebaseConfig);
const auth = getAuth();

const user = ref(null);
const isSignedIn = ref(false);

const uiConfig = {
  signInFlow: "popup",
  signInOptions: [
    firebase.auth.GoogleAuthProvider.PROVIDER_ID,
    firebase.auth.TwitterAuthProvider.PROVIDER_ID,
    firebase.auth.GithubAuthProvider.PROVIDER_ID,
  ],
  callbacks: {
    signInSuccessWithAuthResult: function (authResult) {
      user = authResult.user.displayName;
      console.log(authResult);
      isSignedIn.value = true;
      console.log("Signed in by user " + user);

      // so it doesn't refresh the page
      return false;
    },
    uiShown: function () {
      // The widget is rendered.
      // Hide the loader.
      document.getElementById("loader").style.display = "none";
    },
  },
};

// Initialize the FirebaseUI Widget using Firebase.
var ui = new firebaseui.auth.AuthUI(firebase.auth());

ui.start("#firebaseui-auth-container", uiConfig);

const handleSignOut = () => {
  signOut(auth)
    .then(() => {
      // Sign-out successful.
      user.value = null;
      isSignedIn.value = false;
      console.log("Signed out");
      ui.start("#firebaseui-auth-container", uiConfig);
    })
    .catch((error) => {
      // An error happened.
      console.log(error);
    });
};
</script>

<template>
  <header>
    <h1 class="green">Login</h1>
  </header>
  <main>
    <div>
      <h2 v-if="user">Signed In User: {{ user }}</h2>
      <div class="uicontainer" id="firebaseui-auth-container"></div>
      <div id="loader">Loading...</div>
      <br />
      <div v-if="isSignedIn">
        <button @click="handleSignOut">Sign Out</button>
      </div>
    </div>
  </main>
</template>
<style scoped>
.green {
  text-decoration: none;
  text-align: center;
  margin: auto;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

.uicontainer {
  margin: 50px 50px 50px 50px;
}
</style>

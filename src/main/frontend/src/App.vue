<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>

    <div v-else>
      <button :class="signingUp ? 'button-outline' : ''" @click="signingUp = false">Logowanie</button> <!--  dwukropek oznacza że zaczynam pisać w JS -->
      <button :class="!signingUp ? 'button-outline' : ''" @click="signingUp = true">Rejestracja</button>

      <div v-if="message" :class="successfulRegistration ? 'alert' : 'errorAlert'">
        {{ message }}
      </div>

      <LoginForm v-if="!signingUp" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Załóż konto"></LoginForm>
    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios";

export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      authenticatedUsername: '',
      signingUp: false, 
      message: '',
      successfulRegistration: false,
    }
  },
  methods: {
    logMeIn(user) {
      this.authenticatedUsername = user.login;
    },
    logMeOut() {
      this.authenticatedUsername = '';
    },
    register(user) {
      axios.post('/api/participants', user)
      .then(response => {
          // udało się
          //console.log("Konto założone!")
          this.message = 'Udało się założyć konto';
          this.successfulRegistration = true;
      })
      .catch(response => {
          // nie udało sie    
          //console.log("Błąd. Nie udało sie utworzyć konta.") 
          this.message = 'Nie udało sie założyć konta';
          this.successfulRegistration = false;
      });
    }
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}
.alert{
  padding: 5px;
  margin: 30px;
  border: 2px solid green;
  background: lightgreen;
  font-size: 2em
}
.errorAlert{
  padding: 5px;
  margin: 30px;
  border: 2px solid red;
  background: lightcoral;
  font-size: 2em
}
</style>

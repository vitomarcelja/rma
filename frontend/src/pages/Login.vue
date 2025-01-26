<template>
  <q-page class="q-pa-md">
    <div v-if="!isLoggedIn">
      <h1>Prijava</h1>
      <q-form @submit="handleLogin" class="q-gutter-md">
        <q-input
          v-model="username"
          label="Korisničko ime"
          outlined
          dense
          required
        />
        <q-input
          v-model="password"
          label="Lozinka"
          type="password"
          outlined
          dense
          required
        />
        <q-btn label="Prijava" color="primary" type="submit" unelevated />
      </q-form>
      <q-banner class="q-mt-md" v-if="errorMessage" type="negative">
        {{ errorMessage }}
      </q-banner>
    </div>

    <div v-else>
      <h1>Dobrodošli, {{ user.korime }}</h1>
      <q-btn label="Odjava" color="negative" unelevated @click="logout" />

      <div v-if="user.role === 'admin'">
        <q-btn
          label="Idi na admin panel"
          color="primary"
          unelevated
          @click="goToAdminPanel"
        />
      </div>

      <div v-else>
        <h2>Obični korisnik - dobrodošli!</h2>
        <!-- Ostali sadržaji za korisnike -->
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import { useQuasar } from "quasar";
import axios from "axios";

const $q = useQuasar();
const isLoggedIn = ref(false);
const user = ref(null);
const username = ref("");
const password = ref("");
const errorMessage = ref("");

// Provjera lokalne pohrane
const storedUser = $q.localStorage.getItem("loggedInUser");
if (storedUser) {
  user.value = JSON.parse(storedUser);
  isLoggedIn.value = true;
}

// Prijava korisnika
const handleLogin = async () => {
  try {
    const response = await axios.post("http://localhost:3000/login", {
      username: username.value,
      password: password.value,
    });

    if (response.data.message === "Prijava uspješna!") {
      user.value = response.data.user;
      $q.localStorage.setItem("loggedInUser", JSON.stringify(user.value));
      isLoggedIn.value = true;
      errorMessage.value = "";
    }
  } catch (error) {
    errorMessage.value = error.response?.data?.error || "Greška pri prijavi.";
  }
};

// Odjava korisnika
const logout = () => {
  $q.localStorage.removeItem("loggedInUser");
  user.value = null;
  isLoggedIn.value = false;
};

// Navigacija na admin panel
const goToAdminPanel = () => {
  window.location.href = "/admin"; // Preusmjeravanje na admin stranicu
};
</script>

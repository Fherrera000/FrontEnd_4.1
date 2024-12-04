<template>
  <v-app>

    <v-btn @click="toggleDrawer" icon>
      <v-icon>mdi-menu</v-icon> 
    </v-btn>

    <!-- v-navigation-drawer -->
    <v-navigation-drawer app v-model="drawer" temporary>
      <v-list>
        <v-list-item @click="navigateToEstudiantes">
          <v-list-item-title>Tabla de Estudiantes</v-list-item-title>
        </v-list-item>

        <v-list-item @click="navigateToCursos">
          <v-list-item-title>Tabla de Cursos</v-list-item-title>
        </v-list-item>

        <v-list-item @click="navigateToProfesores">
          <v-list-item-title>Tabla de Profesores</v-list-item-title>
        </v-list-item>

 
      </v-list>
    </v-navigation-drawer>

    <v-container class="fill-height">
      <v-responsive class="align-center fill-height mx-auto" max-width="900">
        <v-img class="mb-4" height="150" src="@/assets/logo.png" />

        <div class="text-center">
          <div class="text-body-2 font-weight-light mb-n1">Welcome to</div>
          <h1 class="text-h2 font-weight-bold">Vuetify</h1>
        </div>

        <div class="py-4" />

        <v-row>
          <v-col cols="12">
   
            <router-view></router-view>
          </v-col>
        </v-row>
      </v-responsive>
    </v-container>
  </v-app>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const drawer = ref(false); 
const router = useRouter();

function toggleDrawer() {
  drawer.value = !drawer.value; 
}

function navigateToEstudiantes() {
  router.push('/tablaEstudiantes'); 
  drawer.value = false; 
}

function navigateToCursos() {
  router.push('/tablaCursos'); 
  drawer.value = false; 
}

function navigateToProfesores() {
  router.push('/tablaProfesores'); 
  drawer.value = false; 
}
</script>

<style scoped>

</style>
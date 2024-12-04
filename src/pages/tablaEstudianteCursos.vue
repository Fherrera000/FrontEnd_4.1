<template>
    <v-container>
      <v-text-field
        v-model="matricula"
        label="Matrícula del estudiante"
        @keyup.enter="obtenerCursos"
      ></v-text-field>
  
      <v-btn @click="obtenerCursos">Buscar Cursos</v-btn>
      <v-btn @click="obtenerProfesores">Buscar Profesores</v-btn>
  
      <v-btn @click="toggleTablas">Ocultar/Mostrar Tablas</v-btn>
  
      <!-- Tabla de cursos, solo si se ha ingresado matrícula -->
      <v-table v-if="mostrarTablas && matricula && cursos.length">
        <thead>
          <tr>
            <th class="text-left">Nombre del Curso</th>
            <th class="text-left">Clave</th>
            <th class="text-left">Calificación</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="curso in cursos" :key="curso.clave">
            <td>{{ curso.nombre }}</td>
            <td>{{ curso.clave }}</td>
            <td>{{ curso.calificacion !== null ? curso.calificacion : 'N/A' }}</td>
          </tr>
        </tbody>
      </v-table>
  
      <!-- Tabla de profesores, solo si se ha ingresado matrícula -->
      <v-table v-if="mostrarTablas && matricula && profesores.length">
        <thead>
          <tr>
            <th class="text-left">Nombre del Profesor</th>
            <th class="text-left">Número de Empleado</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="profesor in profesores" :key="profesor.noEmpleado">
            <td>{{ profesor.nombre }}</td>
            <td>{{ profesor.noEmpleado }}</td>
          </tr>
        </tbody>
      </v-table>
  
    </v-container>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        matricula: '',
        cursos: [],  
        profesores: [],  
        mostrarTablas: true  
      };
    },
    methods: {
      async obtenerCursos() {
        if (!this.matricula) {
          console.error('Matrícula no válida');
          return;
        }
  
        try {
          console.log('Matrícula enviada:', this.matricula); 
          const response = await axios.get(`https://localhost:4000/estudiantes/${this.matricula}/cursos`);
          
          if (response.data && response.data.cursosInscritos) {
            this.cursos = response.data.cursosInscritos; 
          } else {
            console.error('No se encontraron cursos para esta matrícula');
            this.cursos = []; 
          }
        } catch (error) {
          console.error('Error al obtener los cursos:', error);
          this.cursos = []; 
        }
      },
  
      async obtenerProfesores() {
        if (!this.matricula) {
          console.error('Matrícula no válida');
          return;
        }
  
        try {
          console.log('Matrícula enviada:', this.matricula); 
          const response = await axios.get(`https://localhost:4000/estudiantes/${this.matricula}/profesores`);
  
          if (response.data && Array.isArray(response.data)) {
            this.profesores = response.data; 
          } else {
            console.error('No se encontraron profesores para esta matrícula');
            this.profesores = []; 
          }
        } catch (error) {
          console.error('Error al obtener los profesores:', error);
          this.profesores = []; 
        }
      },
  
      toggleTablas() {
        this.mostrarTablas = !this.mostrarTablas;
      }
    }
  };
  </script>
  import TablaEstCur from '@/components/tablaEstCur.vue';
  <style scoped>
  .v-container {
    max-width: 800px;
    margin: 0 auto;
  }
  </style>
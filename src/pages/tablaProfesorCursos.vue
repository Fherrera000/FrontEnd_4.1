<template>
    <v-container>
      
      <v-text-field
        v-model="noEmpleado"
        label="Número de Empleado del Profesor"
        @keyup.enter="obtenerDatos"
      ></v-text-field>
  
     
      <v-btn @click="obtenerCursos">Buscar Cursos</v-btn>
      <v-btn @click="obtenerEstudiantes">Buscar Estudiantes</v-btn>
  
     
      <v-btn @click="toggleTablas">{{ mostrarTablas ? 'Ocultar Tablas' : 'Mostrar Tablas' }}</v-btn>
  
     
      <v-table v-if="mostrarTablas && noEmpleado">
        <thead>
          <tr>
            <th class="text-left">Nombre del Curso</th>
            <th class="text-left">Clave</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="curso in cursos" :key="curso.clave">
            <td>{{ curso.nombre }}</td>
            <td>{{ curso.clave }}</td>
          </tr>
        </tbody>
      </v-table>
  
      <p v-if="!cursos.length && mostrarTablas && noEmpleado">No se encontraron cursos para este profesor.</p>
  
     
      <v-table v-if="mostrarTablas && noEmpleado">
        <thead>
          <tr>
            <th class="text-left">Nombre del Estudiante</th>
            <th class="text-left">Matrícula</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="estudiante in estudiantes" :key="estudiante.matricula">
            <td>{{ estudiante.nombre }}</td>
            <td>{{ estudiante.matricula }}</td>
          </tr>
        </tbody>
      </v-table>
  
      <p v-if="!estudiantes.length && mostrarTablas && noEmpleado">No se encontraron estudiantes para este profesor.</p>
    </v-container>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        noEmpleado: '',  
        cursos: [],       
        estudiantes: [],  
        mostrarTablas: true  
      };
    },
    methods: {
      
      async obtenerDatos() {
        await Promise.all([this.obtenerCursos(), this.obtenerEstudiantes()]);
      },
  
  
      async obtenerCursos() {
        if (!this.noEmpleado) {
          console.error('Número de empleado no válido');
          return;
        }
  
        try {
          const response = await axios.get(`https://localhost:4000/profesores/${this.noEmpleado}/cursos`);
          this.cursos = response.data || [];
          
          if (this.cursos.length === 0) {
            console.log('No se encontraron cursos');
          }
        } catch (error) {
          console.error('Error al obtener los cursos:', error);
          this.cursos = [];
        }
      },
  
      
      async obtenerEstudiantes() {
        if (!this.noEmpleado) {
          console.error('Número de empleado no válido');
          return;
        }
  
        try {
          const response = await axios.get(`https://localhost:4000/profesores/${this.noEmpleado}/estudiantes`);
          this.estudiantes = response.data || [];
          
          if (this.estudiantes.length === 0) {
            console.log('No se encontraron estudiantes');
          }
        } catch (error) {
          console.error('Error al obtener los estudiantes:', error);
          this.estudiantes = [];
        }
      },
  
      
      toggleTablas() {
        this.mostrarTablas = !this.mostrarTablas;
      }
    }
  };
  </script>
  
  <style scoped>
  .v-container {
    max-width: 1000px;
    margin: 0 auto;
  }
  </style>
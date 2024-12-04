<template>
    <v-container>
    
      <v-text-field 
        v-model="noEmpleadoBuscado" 
        label="Buscar por No. de Empleado" 
        @keyup.enter="buscarProfesor"
      ></v-text-field>
      <v-btn @click="buscarProfesor" color="primary">Buscar</v-btn>
      <v-btn @click="mostrarTodosProfesores" color="secondary">Mostrar Todos</v-btn>
  
      <v-table>
        <thead>
          <tr>
            <th class="text-left">Nombre</th>
            <th class="text-left">No. de Empleado</th>
            <th class="text-left">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="profesor in profesores" :key="profesor.noEmpleado">
            <td>{{ profesor.nombre }}</td>
            <td>{{ profesor.noEmpleado }}</td>
            <td>
              <v-btn @click="abrirDialogoEditar(profesor)" color="primary" small>Editar</v-btn>
              <v-btn @click="eliminarProfesor(profesor.noEmpleado)" color="error" small>Eliminar</v-btn>
              <v-btn @click="gestionarInscripcion(profesor.noEmpleado, 'enroll')" color="success" small>Inscribir</v-btn>
              <v-btn @click="gestionarInscripcion(profesor.noEmpleado, 'disenroll')" color="warning" small>Desinscribir</v-btn>
            </td>
          </tr>
        </tbody>
      </v-table>
  
      
      <v-alert v-if="!profesores.length" type="info">
        No hay profesores disponibles.
      </v-alert>
  
      
      <h3>Registrar Nuevo Profesor</h3>
      <v-form ref="nuevoProfesorForm">
        <v-text-field v-model="nuevoProfesor.nombre" label="Nombre"></v-text-field>
        <v-text-field v-model.number="nuevoProfesor.noEmpleado" label="No. de Empleado"></v-text-field>
  
      
        <v-btn @click.prevent="crearProfesor()" color="success">Registrar Profesor</v-btn>
      </v-form>
  
      
      <v-dialog v-model="dialogoEditar" max-width="500px">
        <v-card>
          <v-card-title>Editar Profesor</v-card-title>
          <v-card-text>
            <v-form ref="editarProfesorForm">
              <v-text-field v-model="profesorSeleccionado.nombre" label="Nombre"></v-text-field>
              <v-text-field v-model.number="profesorSeleccionado.noEmpleado" label="No. de Empleado" disabled></v-text-field> <!-- No se puede editar el noEmpleado -->
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="dialogoEditar = false">Cancelar</v-btn>
            <v-btn color="blue darken-1" text @click.prevent="guardarCambios()">Guardar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
  
    </v-container>
    <TablaProfCur />

  </template>
  
  <script>
  import axios from 'axios';
  import TablaEstCur from '@/components/tablaProfCur.vue'
  
  export default {
    data() {
      return {
        profesores: [],
        nuevoProfesor: {
          nombre: '',
          noEmpleado: ''
        },
        dialogoEditar: false,
        profesorSeleccionado: null,
        noEmpleadoBuscado: '', 
      };
    },
    created() {
      this.getAllProfesores();
    },
    methods: {
      async getAllProfesores() {
        try {
          const response = await axios.get('https://localhost:4000/profesores');
          this.profesores = response.data;
        } catch (error) {
          console.error('Error fetching profesores:', error);
        }
      },
  
      async buscarProfesor() {
        if (!this.noEmpleadoBuscado) {
          alert("Por favor ingrese un número de empleado.");
          return;
        }
  
        try {
          const response = await axios.get(`https://localhost:4000/profesores/${this.noEmpleadoBuscado}`);
          this.profesores = [response.data]; 
        } catch (error) {
          console.error('Error al buscar profesor:', error);
          alert('No se encontró ningún profesor con ese número de empleado.');
          this.profesores = []; 
        }
      },
  
      mostrarTodosProfesores() {
        this.getAllProfesores(); 
        this.noEmpleadoBuscado = ''; 
      },
  
      async crearProfesor() {
        if (!this.nuevoProfesor.nombre || !this.nuevoProfesor.noEmpleado) {
          alert("Todos los campos son obligatorios.");
          return;
        }
  
        try {
          await axios.post('https://localhost:4000/profesores', this.nuevoProfesor);
          alert('Profesor registrado exitosamente');
          this.getAllProfesores(); 
          this.nuevoProfesor = { nombre: '', noEmpleado: '' }; 
        } catch (error) {
          console.error('Error al crear profesor:', error);
          alert('Error al registrar el profesor');
        }
      },
  
      async eliminarProfesor(noEmpleado) {
        if (confirm('¿Estás seguro de que deseas eliminar este profesor?')) {
          try {
            await axios.delete(`https://localhost:4000/profesores/${noEmpleado}`);
            this.getAllProfesores(); 
          } catch (error) {
            console.error('Error al eliminar profesor:', error);
            alert('Error al eliminar el profesor');
          }
        }
      },
  
      abrirDialogoEditar(profesor) {
        this.profesorSeleccionado = { ...profesor }; 
        this.dialogoEditar = true; 
      },
  
      async guardarCambios() {
        try {
          await axios.put(`https://localhost:4000/profesores/${this.profesorSeleccionado.noEmpleado}`, {
            nombre: this.profesorSeleccionado.nombre,
          });
          
          alert('Profesor actualizado exitosamente');
          this.dialogoEditar = false; 
          this.getAllProfesores(); 
        } catch (error) {
          console.error('Error al actualizar profesor:', error);
          alert('Error al actualizar el profesor');
        }
      },
  
      async gestionarInscripcion(noEmpleado, accion) {
        const clave = prompt(`Ingrese la clave del curso para ${accion === 'enroll' ? 'inscribir' : 'desinscribir'} al profesor:`);
  
        if (clave) {
          try {
            const response = await axios.patch(`https://localhost:4000/profesores/${noEmpleado}/${accion}`, { clave });
            alert(`${accion === 'enroll' ? 'Inscripción' : 'Desinscripción'} exitosa`);
            this.getAllProfesores(); 
          } catch (error) {
            console.error(`Error al ${accion === 'enroll' ? 'inscribir' : 'desinscribir'} profesor:`, error);
            if (error.response) {
              alert(`Error: ${error.response.data.error}`);
            } else {
              alert('Error al procesar la solicitud');
            }
          }
        } else {
          alert("La clave del curso no puede estar vacía.");
        }
      },
    },
  };
  </script>
  
  <style scoped>

  </style>
  
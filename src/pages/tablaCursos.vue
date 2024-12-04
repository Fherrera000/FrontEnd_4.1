<template>
    <v-container>
 
      <v-text-field 
        v-model="claveBuscada" 
        label="Buscar por Clave" 
        @keyup.enter="buscarCurso"
      ></v-text-field>
      <v-btn @click="buscarCurso" color="primary">Buscar</v-btn>
      <v-btn @click="mostrarTodosCursos" color="secondary">Mostrar Todos</v-btn>
  

      <v-table>
        <thead>
          <tr>
            <th class="text-left">Nombre</th>
            <th class="text-left">Clave</th>
            <th class="text-left">Créditos</th>
            <th class="text-left">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="curso in cursos" :key="curso.clave">
            <td>{{ curso.nombre }}</td>
            <td>{{ curso.clave }}</td>
            <td>{{ curso.creditos }}</td>
            <td>
              <v-btn @click="abrirDialogoEditar(curso)" color="primary" small>Editar</v-btn>
              <v-btn @click="eliminarCurso(curso.clave)" color="error" small>Eliminar</v-btn>
            </td>
          </tr>
        </tbody>
      </v-table>
  
 
      <v-alert v-if="!cursos.length" type="info">
        No hay cursos disponibles.
      </v-alert>
  
 
      <h3>Registrar Nuevo Curso</h3>
      <v-form ref="nuevoCursoForm">
        <v-text-field v-model="nuevoCurso.nombre" label="Nombre"></v-text-field>
        <v-text-field v-model.number="nuevoCurso.clave" label="Clave"></v-text-field>
        <v-text-field v-model.number="nuevoCurso.creditos" label="Créditos"></v-text-field>
  
  
        <v-btn @click.prevent="crearCurso()" color="success">Registrar Curso</v-btn>
      </v-form>
  
   
      <v-dialog v-model="dialogoEditar" max-width="500px">
        <v-card>
          <v-card-title>Editar Curso</v-card-title>
          <v-card-text>
            <v-form ref="editarCursoForm">
              <v-text-field v-model="cursoSeleccionado.nombre" label="Nombre"></v-text-field>
              <v-text-field v-model.number="cursoSeleccionado.clave" label="Clave" disabled></v-text-field> 
              <v-text-field v-model.number="cursoSeleccionado.creditos" label="Créditos"></v-text-field>
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
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        cursos: [],
        nuevoCurso: {
          nombre: '',
          clave: '',
          creditos: ''
        },
        dialogoEditar: false,
        cursoSeleccionado: null,
        claveBuscada: '', 
      };
    },
    created() {
      console.log('Componente creado, llamando a fetchCursos');
      this.getAllCursos();
    },
    methods: {
      async getAllCursos() {
        try {
          const response = await axios.get('https://localhost:4000/cursos');
          console.log('Datos recibidos:', response.data); 
          this.cursos = response.data; 
        } catch (error) {
          console.error('Error fetching cursos:', error);
          alert('Error al cargar los cursos: ' + error.message); 
        }
      },
  
      async buscarCurso() {
        if (!this.claveBuscada) {
          alert("Por favor ingrese una clave.");
          return;
        }
  
        try {
          const response = await axios.get(`https://localhost:4000/cursos/${this.claveBuscada}`);
          this.cursos = [response.data]; 
        } catch (error) {
          console.error('Error al buscar curso:', error);
          alert('No se encontró ningún curso con esa clave.');
          this.cursos = []; 
        }
      },
  
      mostrarTodosCursos() {
        this.getAllCursos(); 
        this.claveBuscada = '';
      },
  
      async crearCurso() {
        if (!this.nuevoCurso.nombre || !this.nuevoCurso.clave || !this.nuevoCurso.creditos) {
          alert("Todos los campos son obligatorios.");
          return;
        }
  
        try {
          await axios.post('https://localhost:4000/cursos', this.nuevoCurso);
          alert('Curso registrado exitosamente');
          this.getAllCursos(); 
          this.nuevoCurso = { nombre: '', clave: '', creditos: '' }; 
        } catch (error) {
          console.error('Error al crear curso:', error);
          alert('Error al registrar el curso');
        }
      },
  
      abrirDialogoEditar(curso) {
        this.cursoSeleccionado = { ...curso }; 
        this.dialogoEditar = true; 
      },
  
      async guardarCambios() {
        try {
          await axios.put(`https://localhost:4000/cursos/${this.cursoSeleccionado.clave}`, {
            nombre: this.cursoSeleccionado.nombre,
            creditos: this.cursoSeleccionado.creditos,
          });
          
          alert('Curso actualizado exitosamente');
          this.dialogoEditar = false; 
          this.getAllCursos(); 
        } catch (error) {
          console.error('Error al actualizar curso:', error);
          alert('Error al actualizar el curso');
        }
      },
  
      async eliminarCurso(clave) {
        if (confirm('¿Estás seguro de que deseas eliminar este curso?')) {
          try {
            await axios.delete(`https://localhost:4000/cursos/${clave}`);
            this.getAllCursos(); 
          } catch (error) {
            console.error('Error al eliminar curso:', error);
            alert('Error al eliminar el curso');
          }
        }
      },
    },
  };
  </script>
  
  <style scoped>
  
  </style>
  
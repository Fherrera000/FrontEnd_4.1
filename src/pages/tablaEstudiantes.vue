<template>
    <v-container>
      <v-text-field
        v-model="matriculaBuscada"
        label="Buscar Estudiante por Matrícula"
        @keyup.enter="buscarEstudiante"
      ></v-text-field>
      <v-btn @click="buscarEstudiante" color="primary">Buscar</v-btn>
      <v-btn @click="mostrarTodosEstudiantes" color="secondary">Mostrar Todos</v-btn>
  
      <v-table>
        <thead>
          <tr>
            <th class="text-left">Nombre</th>
            <th class="text-left">Matrícula</th>
            <th class="text-left">Semestre de Ingreso</th>
            <th class="text-left">Créditos Cursados</th>
            <th class="text-left">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="estudiante in estudiantes" :key="estudiante.matricula">
            <td>{{ estudiante.nombre }}</td>
            <td>{{ estudiante.matricula }}</td>
            <td>{{ estudiante.semestreIngreso }}</td>
            <td>{{ estudiante.creditosCursados }}</td>
            <td>
              <v-btn @click="abrirDialogoEditar(estudiante)" color="primary" small>Editar</v-btn>
              <v-btn @click="eliminarEstudiante(estudiante.matricula)" color="error" small>Eliminar</v-btn>
              <v-btn @click="gestionarInscripcion(estudiante.matricula, 'enroll')" color="success" small>Inscribir</v-btn>
              <v-btn @click="gestionarInscripcion(estudiante.matricula, 'disenroll')" color="warning" small>Desinscribir</v-btn>
            </td>
          </tr>
        </tbody>
      </v-table>
  
      
      <v-alert v-if="!estudiantes.length" type="info">
        No hay estudiantes disponibles.
      </v-alert>
  
  

      <v-dialog v-model="dialogoEditar" max-width="500px">
        <v-card>
          <v-card-title>Editar Estudiante</v-card-title>
          <v-card-text>
            <v-form ref="form">
              <v-text-field v-model="estudianteSeleccionado.nombre" label="Nombre"></v-text-field>
              <v-text-field v-model="estudianteSeleccionado.semestreIngreso" label="Semestre de Ingreso"></v-text-field>
              <v-text-field v-model.number="estudianteSeleccionado.creditosCursados" label="Créditos Cursados"></v-text-field>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="dialogoEditar = false">Cancelar</v-btn>
            <v-btn color="blue darken-1" text @click.prevent="guardarCambios()">Guardar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
  
      <h3>Inscribir Nuevo Estudiante</h3>
      <v-form ref="nuevoEstudianteForm">
        <v-text-field v-model="nuevoEstudiante.nombre" label="Nombre"></v-text-field>
        <v-text-field v-model="nuevoEstudiante.matricula" label="Matrícula"></v-text-field>
        <v-text-field v-model.number="nuevoEstudiante.semestreIngreso" label="Semestre de Ingreso"></v-text-field>
        <v-text-field v-model.number="nuevoEstudiante.creditosCursados" label="Créditos Cursados"></v-text-field>
  
        
        <v-btn @click.prevent="crearEstudiante()" color="success">Inscribir Estudiante</v-btn>
      </v-form>
  
    </v-container>
    <TablaEstCur />
  </template>
  
  <script>
import TablaEstCur from '@/components/tablaEstCur.vue';
  import axios from 'axios';
  
  export default {
    data() {
      return {
        estudiantes: [],
        matriculaBuscada: '',
        estudianteEncontrado: null,
        dialogoEditar: false,
        estudianteSeleccionado: null,
        nuevoEstudiante: {
          nombre: '',
          matricula: '',
          semestreIngreso: '',
          creditosCursados: 0
        }
      };
    },
    created() {
      this.getAllEstudiantes();
    },
    methods: {
      async getAllEstudiantes() {
        try {
          const response = await axios.get('https://localhost:4000/estudiantes/');
          this.estudiantes = response.data;
          this.estudianteEncontrado = null; 
          this.matriculaBuscada = ''; 
        } catch (error) {
          console.error('Error al obtener estudiantes:', error);
        }
      },
  
      async buscarEstudiante() {
        if (!this.matriculaBuscada) {
          alert("Por favor ingrese una matrícula.");
          return;
        }
  
        try {
          const response = await axios.get(`https://localhost:4000/estudiantes/${this.matriculaBuscada}`);
          this.estudianteEncontrado = response.data; 
          this.estudiantes = [this.estudianteEncontrado]; 
        } catch (error) {
          console.error('Error al buscar estudiante:', error);
          alert('No se encontró ningún estudiante con esa matrícula.');
          this.estudianteEncontrado = null; 
          this.estudiantes = []; 
        }
      },
  
      mostrarTodosEstudiantes() {
        this.getAllEstudiantes(); 
      },
  
      abrirDialogoEditar(estudiante) {
        this.estudianteSeleccionado = { ...estudiante };
        this.dialogoEditar = true;
      },
  
      async guardarCambios() {
        try {
          await axios.put(`https://localhost:4000/estudiantes/${this.estudianteSeleccionado.matricula}`, {
            nombre: this.estudianteSeleccionado.nombre,
            semestreIngreso: this.estudianteSeleccionado.semestreIngreso,
            creditosCursados: this.estudianteSeleccionado.creditosCursados,
          });
          this.dialogoEditar = false;
          this.getAllEstudiantes();
        } catch (error) {
          console.error('Error al actualizar estudiante:', error);
        }
      },
  
      async eliminarEstudiante(matricula) {
        if (confirm('¿Estás seguro de que deseas eliminar este estudiante?')) {
          try {
            await axios.delete(`https://localhost:4000/estudiantes/${matricula}`);
            this.getAllEstudiantes();
          } catch (error) {
            console.error('Error al eliminar estudiante:', error);
          }
        }
      },
  
      async crearEstudiante() {
        if (!this.nuevoEstudiante.nombre || !this.nuevoEstudiante.matricula || !this.nuevoEstudiante.semestreIngreso) {
          alert("Todos los campos son obligatorios.");
          return;
        }
  
        try {
          await axios.post('https://localhost:4000/estudiantes/', {
            nombre: this.nuevoEstudiante.nombre,
            matricula: this.nuevoEstudiante.matricula,
            semestreIngreso: this.nuevoEstudiante.semestreIngreso,
            creditosCursados: this.nuevoEstudiante.creditosCursados
          });
          alert('Estudiante inscrito exitosamente');
          this.getAllEstudiantes();
          this.nuevoEstudiante = { nombre: '', matricula: '', semestreIngreso: '', creditosCursados: 0 };
        } catch (error) {
          console.error('Error al crear estudiante:', error);
          alert('Error al inscribir estudiante');
        }
      },
  
      async gestionarInscripcion(matricula, accion) {
      const clave = prompt(`Ingrese la clave del curso para ${accion === 'enroll' ? 'inscribir' : 'desinscribir'} al estudiante:`);

      if (clave) {
 
        if (accion === 'enroll') {
          const calificacion = prompt("Ingrese la calificación del estudiante para este curso:");
          if (calificacion === null || calificacion === '') {
            alert("La calificación es obligatoria.");
            return;
          }

          try {
    
            await axios.patch(`https://localhost:4000/estudiantes/${matricula}/${accion}`, { 
              clave: clave, 
              calificacion: parseFloat(calificacion)  
            });
            alert('Inscripción exitosa');
            this.getAllEstudiantes();
          } catch (error) {
            console.error('Error al inscribir estudiante:', error);
            alert('Error al inscribir al estudiante');
          }
        } else {
          try {
            
            await axios.patch(`https://localhost:4000/estudiantes/${matricula}/${accion}`, { clave });
            alert('Desinscripción exitosa');
            this.getAllEstudiantes();
          } catch (error) {
            console.error('Error al desinscribir estudiante:', error);
            alert('Error al desinscribir al estudiante');
          }
        }
      } else {
        alert("La clave del curso no puede estar vacía.");
      }
    },
    }
  };
  </script>
  
  <style scoped>

  </style>
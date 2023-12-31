<script setup>
  import { ref, reactive, watch, computed, onMounted} from 'vue';

  // Componentes
  import Presupuesto from './components/Presupuesto.vue'
  import ControlPresupuesto from './components/ControlPresupuesto.vue';
  import Modal from './components/Modal.vue'
  import Gasto from './components/Gasto.vue'
  import Filtros from './components/Filtros.vue';

  // Helpers
  import { generarId } from './helpers/index'

  // Imagen
  import iconoNuevoGasto from './assets/img/nuevo-gasto.svg'
  
  // State de Presupuesto
  const presupuesto = ref(0)
  // Función para definir un presupuesto
  const definirPresupuesto = (cantidad) => {
    presupuesto.value = cantidad
    disponible.value = cantidad
  }

  // State de lo que hay disponible
  const disponible = ref(0)

  const gastado = ref(0)
  const gastos = ref([])
  const filtro = ref('')
  
  // State modal
  const modal = reactive({
    mostrar: false,
    animar: false
  })
  // Función para modal
  const mostrarModal = () => {
    modal.mostrar = true
    setTimeout(() => {
      modal.animar = true
    }, 300);
  };

  const gasto = reactive({
    nombre: '',
    cantidad: '',
    categoria: '',
    id: null,
    fecha: Date.now()
  })

  watch(gastos, () => {
    const totalGastado = gastos.value.reduce((total, gasto) => gasto.cantidad + total, 0)
    gastado.value = totalGastado
    disponible.value = presupuesto.value - totalGastado

    localStorage.setItem('gastos', JSON.stringify(gastos.value))
  }, {
    deep: true
  })

  watch(modal, () => {
    if(!modal.mostrar) {
      reiniciarStateGasto()
    }
  }, {
    deep: true
  })

  watch(presupuesto, () => {
    localStorage.setItem('presupuesto', presupuesto.value)
  })

  onMounted(() => {
    const presupuestoStorage = localStorage.getItem('presupuesto')
    if(presupuestoStorage) {
      presupuesto.value = Number(presupuestoStorage)
      disponible.value = Number(presupuestoStorage)
    };

    const gastosStorage = localStorage.getItem('gastos')
    if(gastosStorage) {
      gastos.value = JSON.parse(gastosStorage)
    }
  })

  const reiniciarStateGasto = () => {
    Object.assign(gasto, {
      nombre: '',
      cantidad: '',
      categoria: '',
      id: null,
      fecha: Date.now()
    })
  };

  const ocultarModal = () => {
    modal.animar = false
    setTimeout(() => {
      modal.mostrar = false
    }, 300);
  };

  const guardarGasto = () => {
    if (gasto.id) {
      const { id } = gasto
      const i = gastos.value.findIndex((gasto => gasto.id === id))
      gastos.value[i] = {...gasto}
    } else {
      gastos.value.push({
        ...gasto,
        id: generarId()
      })
    }
    ocultarModal()
    reiniciarStateGasto()
  };

  const seleccionarGasto = (id) => {
    const gastoEditar = gastos.value.filter(gasto => gasto.id === id)[0]
    Object.assign(gasto, gastoEditar);
    mostrarModal()
  };

  const eliminarGasto = () => {
    if(confirm('Eliminar')) {
      gastos.value = gastos.value.filter(gastoState => gastoState.id !== gasto.id)
      ocultarModal()
    }
  };

  const gastosFiltrados = computed(() => {
    if(filtro.value) {
      return gastos.value.filter(gasto => gasto.categoria === filtro.value)
    }
    return gastos.value
  })

  const resetApp = () => {
    if(confirm('¿Deseas reiniciar presupuestos y gastos?')) {
      gastos.value = []
      presupuesto.value = 0
    }
  };
</script>


<template>
  <div :class="{fijar: modal.mostrar}">

    <header>
      <!-- Titulo -->
      <h1>Planificador de Gastos</h1>

      <div class="contenedor-header contenedor sombra">

        <Presupuesto
          v-if="presupuesto === 0"
          @definir-presupuesto="definirPresupuesto"
        />

        <ControlPresupuesto
          v-else
          :presupuesto="presupuesto"
          :disponible="disponible"
          :gastado="gastado"
          @reset-app="resetApp"
        />

      </div>
    </header>

    <main v-if="presupuesto > 0">
      <Filtros v-model:filtro="filtro"/>

      <div class="listado-gastos contenedor">

        <h2>{{ gastosFiltrados.length > 0 ? 'Gastos' : 'No hay gastos aún' }}</h2>

        <Gasto
          v-for="gasto in gastosFiltrados"
          :key="gasto.id"
          :gasto="gasto"
          @seleccionar-gasto="seleccionarGasto"
        />
      </div>

      <div class="crear-gasto">
        <img 
          :src="iconoNuevoGasto" 
          alt="icono nuevo gasto"
          @click="mostrarModal"
        />
      </div>

      <Modal
          v-if="modal.mostrar"
          @ocultar-modal="ocultarModal"
          @guardar-gasto="guardarGasto"
          @eliminar-gasto="eliminarGasto"
          :modal="modal"
          :disponible="disponible"
          :id="gasto.id"
          v-model:nombre="gasto.nombre"
          v-model:cantidad="gasto.cantidad"
          v-model:categoria="gasto.categoria"
      />
    </main>

  </div>
</template>

<!-- Estilos que se aplican en todos los componentes -->
<style>
  /* CustomProperties */
  :root {
    --azul: #3b82f6;
    --blanco: #FFF;
    --gris-claro: #F5F5F5;
    --gris: #94a3b8;
    --gris-oscuro: #64748b;
    --negro: #000;
  }
  
  html {
    font-size: 62.5%;
    box-sizing: border-box;
  }
  *,
  *:before,
  *:after {
    box-sizing: inherit;
  }
  
  body {
    font-size: 1.6rem;
    font-family: "Lato", sans-serif;
    background-color: var(--gris-claro);
  }
  
  h1 {
    font-size: 4rem;
  }
  
  h2 {
    font-size: 3rem;
  }
  
  header {
    background-color: var(--azul);
  }

  .fijar {
    overflow: hidden;
    height: 100vh;
  }

  header h1 {
    padding: 3rem 0;
    margin: 0;
    color: var(--blanco);
    text-align: center;
  }

  /* Para todos los contenedores */
  .contenedor {
    width: 90%;
    max-width: 80rem;
    margin: 0 auto;
  }

  .contenedor-header {
    margin-top:  -5rem;
    transform: translateY(5rem);
    padding: 5rem;
  }

  .sombra {
    box-shadow: 0px 10px 15px -3px rgba(0,0,0,0.1);
    background-color: var(--blanco);
    border-radius: 1.2rem;
    padding: 5rem;
  }

  /* Estilos para la imagen de crear gasto */
  .crear-gasto {
    position: fixed;
    bottom: 5rem;
    right: 5rem;
  }
  .crear-gasto img {
    width: 5rem;
    cursor: pointer;
  }


  .listado-gastos {
    margin-top: 10rem;
  }

  .listado-gastos h2 {
    font-weight: 900;
    color: var(--gris-oscuro);
  }
</style>
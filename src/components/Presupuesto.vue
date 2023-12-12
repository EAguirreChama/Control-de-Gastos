<!-- Componente Presupuesto -->
<script setup>
    import { ref } from 'vue'
    import Alerta from './Alerta.vue'

    // State de Presupuesto. Inicia en 0
    const presupuesto = ref(0)
    const error = ref('')

    const emit = defineEmits(['definir-presupuesto'])

    // Validación
    const definirPresupuesto = () => {
        if(presupuesto.value <= 0 || presupuesto.value === '') {
            error.value = 'Presupuesto no válido'

            setTimeout(() => {
                error.value = ''
            }, 3000);

            return
        }
        emit('definir-presupuesto', presupuesto.value)
    }
</script>

<template>
    <form class="presupuesto" @submit.prevent="definirPresupuesto">

        <Alerta v-if="error"> {{ error }} </Alerta>

        <div class="campo">
            <label for="nuevo-presupuesto">Definir Presupuestos</label>
            <input 
                id="nuevo-presupuesto"
                class="nuevo-presupuesto"
                placeholder="Añade tu presupuesto"
                type="number"
                min="0"
                v-model.number="presupuesto"
            />
        </div>

        <input type="submit" value="Definir Presupuesto" />

    </form>
</template>

<style scoped>
    .presupuesto {
        width: 100%;
    }
    .campo {
        display: grid;
        gap: 2rem;
    }
    .presupuesto label {
        font-size: 2.2rem;
        text-align: center;
        color: var(--azul);
    }

    /* Para el de tipo number */
    .presupuesto input[type="number"] {
        background-color: var(--gris-claro);
        border-radius: 1rem;
        padding: 1rem;
        border: none;
        font-size: 2.2rem;
        text-align: center;
    }

    /* Para el de tipo submit */
    .presupuesto input[type="submit"] {
        background-color: var(--azul);
        border: none;
        padding: 1rem;
        text-align: center;
        font-size: 2rem;
        margin-top: 2rem;
        color: var(--blanco);
        font-weight: 900;
        width: 100%;
        transition: background-color 300ms ease;
        border-radius: 1rem;
    }
    .presupuesto input[type="submit"]:hover {
        background-color: #1048A4;
        cursor: pointer;
    }
</style>

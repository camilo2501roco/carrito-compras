<script setup>
import { computed, ref, watch } from 'vue';

const productosDisponibles = ref([
  { id: 1, nombre: 'studio scarlett 3rd gen', precio: 219, imagen: 'src/studio scarlett 3rd gen.png' },
  { id: 2, nombre: 'KeyLab 88 mk3', precio: 1299, imagen: 'src/KeyLab 88 mk3.png' },
  { id: 3, nombre: 'Neumann TLM 103', precio: 900, imagen: 'src/Micrófono de condensador Neumann TLM 103.png' },
  { id: 4, nombre: 'guitarra yamaha C40', precio: 199, imagen: 'src/yamaha c40.webp' },
  { id: 5, nombre: 'Linsoul - KZ ZS10 Pro', precio: 59, imagen: 'src/Linsoul - KZ ZS10 Pro.webp' }
]);


const cargarCarrito = () =>{
const carritoGuardado =  localStorage.getItem('carrito')
if(carritoGuardado){
  try {
    const data =  JSON.parse(carritoGuardado);
    return  data.items ||[];
  } catch (error) {
    console.error('Error al cargar el carrito:', error)
    return [];
  }
}
 return [];
};




const carrito = ref(cargarCarrito());
const guardadoExitoso = ref(false);

const totalItems = computed(() => {
  return carrito.value.reduce((sum, item) => sum + item.cantidad, 0);
});

const subtotal = computed(() => {
  return carrito.value.reduce((sum, item) => sum + (item.precio * item.cantidad), 0);
});

const impuesto = computed(() => {
  return subtotal.value * 0.16;
});

const totalFinal = computed(() => {
  return subtotal.value + impuesto.value;
});

watch(carrito, (nuevoCarrito) => {
  if (nuevoCarrito.length > 0) {
    const carritoData = {
      items: nuevoCarrito,
      fechaGuardado : new Date().toISOString()
    };
    localStorage.setItem('carrito', JSON.stringify(carritoData));
    console.log('Carrito guardado:', carritoData);
    
    guardadoExitoso.value = true;
    setTimeout(() => {
      guardadoExitoso.value = false;
    }, 1000);
  } else {
    localStorage.removeItem('carrito');
     console.log('Carrito vacío, eliminado del localStorage');
  }
}, { deep: true });

watch(totalFinal, (nuevoTotal) => {
  if (nuevoTotal > 1000) {
    console.log('Alerta: El total supera los $1000');
  }
});

const agregarAlCarrito = (producto) => {
  const itemExistente = carrito.value.find(item => item.id === producto.id);
  
  if (itemExistente) {
    itemExistente.cantidad++;
  } else {
    carrito.value.push({
      id: producto.id,
      nombre: producto.nombre,
      precio: producto.precio,
      cantidad: 1
    });
  }
};

const incrementarCantidad = (id) => {
  const item = carrito.value.find(item => item.id === id);
  if (item) {
    item.cantidad++;
  }
};

const decrementarCantidad = (id) => {
  const item = carrito.value.find(item => item.id === id);
  if (item) {
    item.cantidad--;
    
    if (item.cantidad <= 0) {
      carrito.value = carrito.value.filter(item => item.id !== id);
    }
  }
};

const obtenerCantidad = (id) => {
  const item = carrito.value.find(item => item.id === id);
  return item ? item.cantidad : 0;
};
</script>

<template>
 
 <div class="app-container">



<q-banner 
  v-if="totalFinal > 1000" 
  class="bg-warning text-white q-mb-md" 
  rounded
>
  <template v-slot:avatar>
    <q-icon name="warning" color="white" />
  </template>
  Tu carrito supera los $1000. ¡Podrías calificar para envío gratis!
</q-banner>


<q-banner 
  v-if="guardadoExitoso" 
  class="bg-positive text-white q-mb-md" 
  rounded
>
  <template v-slot:avatar>
    <q-icon name="check_circle" color="white" />
  </template>
  Carrito guardado automáticamente
</q-banner>
    

    <div class="main-layout">
      <!-- PRODUCTOS DISPONIBLES -->
      <div class="seccion-productos">
        <h2>Productos Disponibles</h2>

        <div class="productos-grid">
          <div 
            v-for="producto in productosDisponibles" 
            :key="producto.id" 
            class="producto-card"
          >
            <div class="producto-imagen">
              <img 
                v-if="producto.imagen" 
                :src="producto.imagen" 
                :alt="producto.nombre"
              >
            </div>

            <div class="producto-info">
              <h3>{{ producto.nombre }}</h3>
              <p class="producto-precio">${{ producto.precio.toFixed(2) }}</p>
            </div>

            <div class="producto-acciones">
              <div v-if="obtenerCantidad(producto.id) > 0" class="controles">
                <button @click="decrementarCantidad(producto.id)">-</button>
                <span>{{ obtenerCantidad(producto.id) }}</span>
                <button @click="incrementarCantidad(producto.id)">+</button>
              </div>

              <button 
                v-else 
                @click="agregarAlCarrito(producto)" 
                class="btn-agregar"
              >
                Agregar al carrito
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- RESUMEN DEL CARRITO -->
      <div class="resumen-carrito">
        <h2>Resumen del Carrito</h2>
        <div class="total-items">{{ totalItems }} items</div>

        <div class="caja-resumen">
          <div class="fila-resumen">
            <span>Total de productos:</span>
            <strong>{{ totalItems }} items</strong>
          </div>

          <div class="fila-resumen">
            <span>Subtotal:</span>
            <strong>${{ subtotal.toFixed(2) }}</strong>
          </div>

          <div class="fila-resumen">
            <span>Impuesto (16%):</span>
            <strong>${{ impuesto.toFixed(2) }}</strong>
          </div>

          <hr>

          <div class="fila-resumen-total">
            <span>TOTAL A PAGAR:</span>
            <strong>${{ totalFinal.toFixed(2) }}</strong>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>







<style scoped>

</style>

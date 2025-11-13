<script setup>
import { computed, ref, watch } from 'vue';

const productosDisponibles = ref([
  { id: 1, nombre: 'studio scarlett 3rd gen', precio: 219, imagen: 'https://sinfoniamusical.com/cdn/shop/files/2I24GKIT.jpg?v=1708445642&width=1214' },
  { id: 2, nombre: 'KeyLab 88 mk3', precio: 1299, imagen: 'https://medias.arturia.net/images/products/keylab-essential-88-mk3/keylab.png' },
  { id: 3, nombre: 'Neumann TLM 103', precio: 900, imagen: 'https://musicalboutique.co/cdn/shop/products/neumann-tlm-103-nickel_5317_1_1.jpg?v=1590604313' },
  { id: 4, nombre: 'guitarra yamaha C40', precio: 199, imagen: 'https://www.pianosbogota.com/wp-content/uploads/2021/02/GUITARRA-ELCTROACUSTICA-YAMAHA-C40.jpg' },
  { id: 5, nombre: 'Linsoul - KZ ZS10 Pro', precio: 59, imagen: 'https://www.linsoul.com/cdn/shop/products/cd27f3077afd0b0ed82cc78fab831e51.jpg?v=1629801301&width=1946' }
]);

const cargarCarrito = () => {
  const carritoGuardado = localStorage.getItem('carrito');
  return carritoGuardado ? JSON.parse(carritoGuardado).items || [] : [];
};

const carrito = ref(cargarCarrito());
const guardadoExitoso = ref(false);

const totalItems = computed(() => carrito.value.reduce((sum, item) => sum + item.cantidad, 0));
const subtotal = computed(() => carrito.value.reduce((sum, item) => sum + (item.precio * item.cantidad), 0));
const impuesto = computed(() => subtotal.value * 0.16);
const totalFinal = computed(() => subtotal.value + impuesto.value);

watch(carrito, (nuevoCarrito) => {
  if (nuevoCarrito.length > 0) {
    localStorage.setItem('carrito', JSON.stringify({ 
      items: nuevoCarrito, 
      fechaGuardado: new Date().toISOString() 
    }));
    guardadoExitoso.value = true;
    setTimeout(() => guardadoExitoso.value = false, 1000);
  } else {
    localStorage.removeItem('carrito');
  }
}, { deep: true });

watch(totalFinal, (nuevoTotal) => {
  if (nuevoTotal > 1000) console.log('Alerta: El total supera los $1000');
});

const agregarAlCarrito = (producto) => {
  const itemExistente = carrito.value.find(item => item.id === producto.id);
  itemExistente ? itemExistente.cantidad++ : carrito.value.push({ ...producto, cantidad: 1 });
};

const incrementarCantidad = (id) => {
  const item = carrito.value.find(item => item.id === id);
  if (item) item.cantidad++;
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

const obtenerCantidad = (id) => carrito.value.find(item => item.id === id)?.cantidad || 0;
</script>

<template>

  <div class="app-container">


    <div class="notificaciones-container">

      <transition name="slide-down">

        <q-banner v-if="totalFinal > 1000" class="bg-lime text-white banner-notificacion" dense rounded>

          <template v-slot:avatar>
            <q-icon name="local_shipping" color="white" size="xs" />
          </template>
          <div class="texto-notificacion">
            Tu carrito supera los $1000. ¡envío gratis!
          </div>
        </q-banner>
      </transition>




      <transition name="fade">

        <q-banner v-if="guardadoExitoso" class="bg-positive text-white banner-notificacion" dense rounded>

          <template v-slot:avatar>
            <q-icon name="check_circle" color="white" size="" />
          </template>
          <div class="text-body2"> carrito guardado</div>
        </q-banner>
      </transition>
    </div>

    <div class="main-layout">

    
      <div class="seccion-productos">

        <q-card flat bordered>

          <q-card-section class="bg-primary text-white header-productos">

            <div class="text-h6 q-mb-xs">
              <q-icon name="shopping_cart" size="sm" class="q-mr-sm" />
              Productos Disponibles
            </div>
          </q-card-section>

          <q-separator />





       
<div class="productos-tabla-container">

        <div class="productos-tabla">
          <div v-for="producto in productosDisponibles" :key="producto.id" class="producto-fila">
            <div class="producto-col imagen-col">
              <q-img :src="producto.imagen" :alt="producto.nombre" spinner-color="primary" class="producto-imagen"
                ratio="1">

                <template v-slot:error>
                  <div class="absolute-full felx felx-center bg-grey-3">
                    <q-icon name="image_not_supported" size="32px" color="grey-5" />
                  </div>
                </template>

              </q-img>
            </div>


           

          <div class="producto-col info-col">
            <div class="text-body1 text-weight-bold q-mb-xs producto-nombre">

          {{ producto.nombre }}

            </div>
           
          </div>

           <div class="prodcuto-col precio-col">
            <div class="text-h6 text-primary text-weight-bold">
              ${{ producto.precio.toFixed(2) }}
            </div>
           </div>


         

           <div class="producto-col acciones-col">
            <div v-if="obtenerCantidad(producto.id) >0 " class="controles-cantidad">

              <q-btn   round dense color="negative" icon="remove" size="sm" @click="decrementarCantidad(producto.id)"/>

              <q-chip color="primary" text-color="white" size="md" class="cantidad-chip">
                {{ obtenerCantidad(producto.id) }}
              </q-chip>


              <q-btn round dense color="positive" icon="add" size="sm" @click="incrementarCantidad(producto.id)"/>
            </div>


          <q-btn v-else color="primary" label="agregar" icon="add_shopping_cart" 
           unelevated @click="agregarAlCarrito(producto)"class="full-width" />

           </div>
            </div>
             </div>
             </div>
            </q-card>
            </div>
           



             

              <div class="resumen-carrito">

            <q-card flat bordered class="sticky-card">
              <q-card-section class="bg-primary text-white header-resumen">
                <div class="text-h6 q-mb-xs">
                  <q-icon name="receipt" size="sm" class="q-mr-sm"/>
                  Resumen del carrito
                </div>

                <q-badge color="positive" :label="`${totalItems} items`"  />
              </q-card-section>


              <q-separator/>

              <q-card-section class="seccion-resumen">

               <div  v-if="carrito.length > 0 " class="carrito-items q-mb-md">

                <div v-for="item in carrito" :key="item.id" class="carrito-item q-mb-sm" >


                  <div class="item-nombre  text-caption text-dark">{{ item.nombre }}</div>
                  
                  <div class="item-total text-body2  text-weight-bold text-dark">
                    ${{ (item.cantidad * item.precio).toFixed(2) }}
                  </div>
                </div>

               </div>




                <div v-else class="text-center q-pa-sm text-grey-6">

                  <q-icon name="shopping_cart " size="48px" color="grey-4"/>
                  <div class="text-caption q-mt-xs"> Tu carrito esta vacio</div>
                </div>

                <q-separator  class="q-my-sm"/>

                <div class="resumen-calculos">
                  <div class="fila-calculo">
                    <span class="text-body2 text-dark">Subtotal:</span>
                    <span class="text-body1 text-weight-bold text-dark">${{ subtotal.toFixed(2) }}</span>
                  </div>


                  <div class="fila-calculo">
                    <span class="text-caption text-grey-8">Impuesto (16%):</span>
                    <span class="text-body2 text-grey-8">${{ impuesto.toFixed(2) }}</span>
                  </div>


                  <q-separator class="q-my-sm"/>

                  <div class="fila-calculo total-final">
                    <span class="text-body1 text-weight-bold text-dark">TOTAL:</span>
                    <span class="text-h6 text-weight-bold text-primary"> {{ totalFinal.toFixed(2) }}</span>
                  </div>
                </div>

              </q-card-section>
            </q-card>

    </div>
  </div>
    </div>
 
    
     
</template>







<style scoped>
.app-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
  position: relative;
}


.notificaciones-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 9999;
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 400px;
}

.banner-fixed {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.banner-guardado {
  animation: slideIn 0.3s ease-out;
}


@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.3s ease;
}

.slide-down-enter-from {
  transform: translateY(-100%);
  opacity: 0;
}

.slide-down-leave-to {
  transform: translateY(-100%);
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}


.main-layout {
  display: grid;
  grid-template-columns: 1fr 350px;
  gap: 20px;
  align-items: start;
  margin-top: 20px;
}


.seccion-productos {
  width: 100%;
}

.header-productos {
  padding: 12px 16px;
}


.productos-tabla {
  display: flex;
  flex-direction: column;
}

.producto-fila {
  display: grid;
  grid-template-columns: 80px 1fr 120px 180px;
  gap: 16px;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid #e0e0e0;
  transition: background-color 0.2s ease;
}

.producto-fila:hover {
  background-color: #e8f4f8;
}

.producto-fila:last-child {
  border-bottom: none;
}


.producto-col {
  display: flex;
  align-items: center;
}

.imagen-col {
  justify-content: center;
}

.info-col {
  flex-direction: column;
  align-items: flex-start;
  gap: 2px;
}

.precio-col {
  justify-content: center;
}

.acciones-col {
  justify-content: flex-end;
}


.producto-imagen {
  width: 70px;
  height: 70px;
  border-radius: 8px;
}


.producto-nombre {
  color: #1a202c ;
  font-size: 1rem ;
  line-height: 1.4;
}


.controles-cantidad {
  display: flex;
  align-items: center;
  gap: 10px;
}

.cantidad-chip {
  min-width: 45px;
  justify-content: center;
}


.resumen-carrito {
  position: relative;
}

.sticky-card {
  position: sticky;
  top: 20px;
}

.header-resumen {
  padding: 12px 16px;
}


.seccion-resumen {
  background-color: #ffffff;
  padding: 12px 16px;
}


.carrito-items {
  max-height: 220px;
  overflow-y: auto;
  padding-right: 8px;
}

.carrito-item {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 6px;
  padding: 8px;
  background-color: #f0f4f8;
  border-radius: 6px;
  border: 1px solid #e0e6ed;
}

.item-nombre {
  grid-column: 1 / -1;
  font-weight: 600;
  color: #1a202c;
  line-height: 1.3;
}

.item-detalle {
  grid-column: 1;
  color: #4a5568;
}

.item-total {
  grid-column: 2;
  text-align: right;
  color: #1a202c;
}


.resumen-calculos {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.fila-calculo {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 4px;
}

.total-final {
  padding: 10px 8px;
  background-color: #f0f9ff;
  border-radius: 6px;
  margin-top: 6px;
}


.carrito-items::-webkit-scrollbar {
  width: 5px;
}

.carrito-items::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.carrito-items::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.carrito-items::-webkit-scrollbar-thumb:hover {
  background: #555;
}


@media (max-width: 1200px) {
  .main-layout {
    grid-template-columns: 1fr;
  }

  .sticky-card {
    position: relative;
    top: 0;
  }

  .notificaciones-container {
    max-width: 90%;
    right: 5%;
  }
}

@media (max-width: 768px) {
  .producto-fila {
    grid-template-columns: 70px 1fr;
    gap: 12px;
  }

  .precio-col,
  .info-col {
    grid-column: 2;
  }

  .acciones-col {
    grid-column: 1 / -1;
    justify-content: center;
    margin-top: 8px;
  }

  .producto-imagen {
    width: 60px;
    height: 60px;
  }

  .notificaciones-container {
    top: 10px;
    right: 50px;
    left: 50px;
    max-width: none;
  }
}

.productos-tabla-container {
  max-height: 70vh;
  overflow-y: auto;
}


.productos-tabla-container::-webkit-scrollbar {
  width: 8px;
}

.productos-tabla-container::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.productos-tabla-container::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.productos-tabla-container::-webkit-scrollbar-thumb:hover {
  background: #555;
}

@media (max-width: 768px) {
  .productos-tabla-container {
    max-height: 60vh;
    overflow-y: auto;
  }
  
  
  .productos-tabla-container::-webkit-scrollbar {
    width: 6px;
  }
}
</style>

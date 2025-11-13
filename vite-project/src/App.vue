<template>
  <div class="q-pa-lg">
    <!-- Banner de envío gratis cuando supera $1000 -->
    <q-banner v-if="mostrarEnvioGratis" class="bg-green text-white q-mb-md shadow-4">
      <template v-slot:avatar>
        <q-icon name="local_shipping" color="white" size="32px" />
      </template>
      <div class="text-h5 text-weight-bold">🎉 ¡Felicidades! ¡Envío GRATIS</div>
      <div class="text-h6">Tu compra de <strong>${{ totalFinal.toFixed(2) }}</strong> califica para envío gratis</div>
      <div class="q-mt-sm">Has superado los $1000 y te llevas el envío completamente gratis</div>
      
      <template v-slot:action>
        <q-btn 
          flat 
          color="white" 
          label="Ver ofertas" 
          icon="local_offer"
          @click="scrollToProductos"
        />
        <q-btn 
          flat 
          color="white" 
          label="Continuar comprando" 
          icon="shopping_cart"
        />
      </template>
    </q-banner>

    <!-- Header -->
    <div class="titulo q-mb-xl">
      <h1 class="text-h3 text-weight-bold text-primary q-mb-xs">🛒 Carrito de Compras</h1>
      <p class="text-h6 text-grey-7">Selecciona tus productos favoritos</p>
      
  
    </div>

    <!-- Resumen del carrito -->
    <div class="resumen-carrito q-mb-xl" v-if="carrito.length > 0" id="resumen-carrito">
      <q-card class="bg-blue-grey-1 shadow-4">
        <q-card-section class="q-pa-lg">
          <div class="text-h5 text-weight-bold text-blue-grey-10 q-mb-md">📊 Resumen de Compra</div>
          <div class="detalles-carrito">
            <div class="row items-center q-mb-sm">
              <q-icon name="shopping_basket" class="q-mr-sm text-blue-7" />
              <span class="text-h6">Productos en carrito: <strong class="text-primary">{{ totalItems }}</strong></span>
            </div>
            <div class="row items-center q-mb-sm">
              <q-icon name="receipt" class="q-mr-sm text-green-7" />
              <span class="text-h6">Subtotal: <strong>${{ subtotal.toFixed(2) }}</strong></span>
            </div>
            <div class="row items-center q-mb-sm">
              <q-icon name="account_balance" class="q-mr-sm text-orange-7" />
              <span class="text-h6">Impuesto (16%): <strong>${{ impuesto.toFixed(2) }}</strong></span>
            </div>
            
            <q-separator class="q-my-md" />
            <div class="row items-center">
              <q-icon name="payments" class="q-mr-sm text-red-7" size="24px" />
              <span class="text-h5 text-weight-bold" :class="mostrarEnvioGratis ? 'text-green' : 'text-blue-grey-10'">
                Total: ${{ totalFinal.toFixed(2) }}
              </span>
              <q-icon v-if="mostrarEnvioGratis" name="celebration" color="green" class="q-ml-sm" />
            </div>
            
            <!-- Indicador visual del envío gratis -->
            <div class="q-mt-md">
              <div class="text-caption text-grey-7 q-mb-xs">
                <q-icon name="local_shipping" class="q-mr-xs" />
                Envío gratis a partir de $1000 
                <span class="float-right" :class="totalFinal >= 1000 ? 'text-green' : 'text-orange'">
                  {{ Math.min((totalFinal / 1000) * 100, 100).toFixed(0) }}%
                </span>
              </div>
              <q-linear-progress 
                :value="Math.min(totalFinal / 1000, 1)" 
                :color="totalFinal >= 1000 ? 'green' : 'primary'"
                size="20px"
                class="q-mt-sm"
                stripe
              >
                <div class="absolute-full flex flex-center">
                  <span class="text-white text-caption">
                    ${{ totalFinal.toFixed(0) }} / $1000
                    <span v-if="totalFinal >= 1000"> ✅</span>
                  </span>
                </div>
              </q-linear-progress>
              
              <!-- Mensaje de progreso -->
              <div v-if="totalFinal < 1000" class="text-center q-mt-sm">
                <q-badge color="orange" class="q-pa-sm">
                  <q-icon name="info" class="q-mr-sm" size="sm" />
                  Agrega ${{ (1000 - totalFinal).toFixed(2) }} más para envío gratis
                </q-badge>
              </div>
              
              <!-- Mensaje de éxito -->
              <div v-else class="text-center q-mt-sm">
                <q-badge color="green" class="q-pa-xs">
                  <q-icon name="celebration" class="q-mr-xs" />
                  ¡Has conseguido envío gratis!
                </q-badge>
              </div>
            </div>
          </div>
        </q-card-section>
      </q-card>
      
      <!-- Lista de productos en el carrito -->
      <q-card class="q-mt-lg shadow-4">
        <q-card-section class="q-pa-lg">
          <div class="text-h5 text-weight-bold text-blue-grey-10 q-mb-md">
            🛍️ Productos en el Carrito 
            <q-badge v-if="mostrarEnvioGratis" color="green" class="q-ml-sm">
              ENVÍO GRATIS
            </q-badge>
          </div>
          <div v-for="producto in carrito" :key="producto.id" class="item-carrito q-pa-md q-mb-sm rounded-borders bg-grey-1 shadow-1">
            <div class="row items-center justify-between full-width">
              <div class="col-5">
                <div class="text-h6 text-weight-medium text-blue-grey-10">{{ producto.nombre }}</div>
                <div class="text-caption text-grey-7">${{ producto.precio }} c/u</div>
              </div>
              <div class="col-4">
                <div class="controles-cantidad row items-center justify-center">
                  <q-btn 
                    @click="decrementarCantidad(producto.id)" 
                    icon="remove" 
                    size="sm" 
                    round 
                    dense 
                    color="primary"
                    class="shadow-1"
                  />
                  <span class="cantidad q-mx-lg text-h5 text-weight-bold text-primary">
                    {{ producto.cantidad }}
                  </span>
                  <q-btn 
                    @click="incrementarCantidad(producto.id)" 
                    icon="add" 
                    size="sm" 
                    round 
                    dense 
                    color="primary"
                    class="shadow-1"
                  />
                </div>
              </div>
              <div class="col-3 text-right">
                <div class="text-h5 text-green text-weight-bold">
                  ${{ (producto.precio * producto.cantidad).toFixed(2) }}
                </div>
              </div>
            </div>
          </div>
        </q-card-section>
      </q-card>
    </div>

    <!-- Productos disponibles -->
    <div class="Cards" id="productos-section">
      <div class="text-h4 text-weight-bold text-center text-blue-grey-10 q-mb-lg">
        🎯 Productos Disponibles
        <q-badge v-if="mostrarEnvioGratis" color="green" class="q-ml-sm">
          ¡Sigue comprando con ENVÍO GRATIS!
        </q-badge>
      </div>
      <div class="row justify-center q-col-gutter-lg">
        <div class="col-12 col-sm-6 col-md-4 col-lg-3" v-for="producto in productos" :key="producto.id">
          <q-card class="my-card shadow-6 bg-grey-5  hover-card" flat bordered>
            <q-card-section class="q-pa-sm text-center">
              <div class="text-h2 q-mb-md"></div>
              <div class="text-h6 text-weight-bold text-blue-grey-10">{{ producto.nombre }}</div>
              <div class="text-h4 text-grey-10 text-weight-bold q-mt-lg q-mb-xs">${{ producto.precio }}</div>
              
              <!-- Badge de envío gratis -->
              <div v-if="mostrarEnvioGratis" class="q-mt-sm">
                <q-badge color="green" class="q-pa-xs">
                  <q-icon name="local_shipping" class="q-mr-xs" />
                  ENVÍO GRATIS
                </q-badge>
              </div>
            </q-card-section>

            <q-separator />

            <q-card-actions vertical class="q-pa-lg">
              <!-- Controles de cantidad para cada producto -->
              <div class="controles-producto" v-if="cantidadEnCarrito(producto.id) > 0">
                <div class="text-caption text-center text-black q-mb-sm">✅ En carrito: {{ cantidadEnCarrito(producto.id) }} unidad(es)</div>
                <div class="controles-cantidad row justify-center items-center">
                  <q-btn 
                    @click="decrementarCantidad(producto.id)" 
                    icon="remove" 
                    size="sm" 
                    round 
                    dense 
                    color="red-7"
                    class="shadow-2"
                  />
                  <span class="cantidad q-mx-lg text-h6 text-weight-bold">{{ cantidadEnCarrito(producto.id) }}</span>
                  <q-btn 
                    @click="incrementarCantidad(producto.id)" 
                    icon="add" 
                    size="sm" 
                    round 
                    dense 
                    color="green-9"
                    class="shadow-2"
                  />
                </div>
              </div>
              <q-btn 
                v-else 
                @click="agregarAlCarrito(producto)" 
                color="green-13"
                icon="add_shopping_cart"
                label="Agregar al carrito"
                class="full-width q-mt-md shadow-3"
                size="lg"
              />
            </q-card-actions>
          </q-card>
        </div>
      </div>
    </div>

    <!-- Empty state -->
    <div v-if="carrito.length === 0" class="text-center q-mt-xl q-pa-xl">
      <q-icon name="shopping_cart" size="100px" color="grey-4" class="q-mb-md" />
      <div class="text-h5 text-grey-6 q-mb-md">Tu carrito está vacío</div>
      <div class="text-body1 text-grey-5">Agrega algunos productos para comenzar</div>
      <div class="q-mt-md">
        <q-badge color="green" class="q-pa-sm">
          <q-icon name="local_shipping" class="q-mr-sm" />
          Envío gratis en compras mayores a $1000
        </q-badge>
      </div>
    </div>

    <!-- Notificación toast para acciones -->
    <q-dialog v-model="mostrarNotificacion" position="top">
      <q-card>
        <q-card-section class="row items-center no-wrap">
          <q-icon :name="notificacionIcono" :color="notificacionColor" size="24px" class="q-mr-sm" />
          <div>
            <div class="text-weight-bold">{{ notificacionTitulo }}</div>
            <div class="text-caption">{{ notificacionMensaje }}</div>
          </div>
          <q-space />
          <q-btn flat round icon="close" v-close-popup />
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// Datos de productos
const productos = ref([
  { id: 1, nombre: 'Laptop Asus', precio: 100 },
  { id: 2, nombre: 'Mouse Gamer', precio: 50 },
  { id: 3, nombre: 'Teclado Mecánico', precio: 120 },
  { id: 4, nombre: 'Monitor 24"', precio: 200 },
  { id: 5, nombre: 'Impresora Multifuncional', precio: 150 },
  { id: 6, nombre: 'Disco Duro Externo', precio: 80 }
])

// Carrito de compras
const carrito = ref([])

// Estados para notificaciones
const mostrarEnvioGratis = ref(false)
const mostrarNotificacion = ref(false)
const notificacionTitulo = ref('')
const notificacionMensaje = ref('')
const notificacionIcono = ref('info')
const notificacionColor = ref('primary')


// Función para mostrar notificación toast
const mostrarToast = (titulo, mensaje, icono = 'info', color = 'primary') => {
  notificacionTitulo.value = titulo
  notificacionMensaje.value = mensaje
  notificacionIcono.value = icono
  notificacionColor.value = color
  mostrarNotificacion.value = true
  
  // Auto cerrar después de 3 segundos
  setTimeout(() => {
    mostrarNotificacion.value = false
  }, 800)
}

// Función para agregar productos al carrito
const agregarAlCarrito = (producto) => {
  console.log('Agregando producto:', producto)
  
  const productoExistente = carrito.value.find(item => item.id === producto.id)
  
  if (productoExistente) {
    productoExistente.cantidad++
    mostrarToast('✅ Producto actualizado', `${producto.nombre} cantidad aumentada`, 'check', 'green')
  } else {
    carrito.value.push({
      ...producto,
      cantidad: 1
    })
    mostrarToast('🛒 Producto agregado', `${producto.nombre} añadido al carrito`, 'add_shopping_cart', 'primary')
  }
  
  console.log('Carrito actual:', carrito.value)
}

// Función para incrementar cantidad
const incrementarCantidad = (productoId) => {
  const producto = carrito.value.find(item => item.id === productoId)
  if (producto) {
    producto.cantidad++
    mostrarToast('➕ Cantidad aumentada', 'Producto actualizado en el carrito', 'add', 'primary')
  }
}

// Función para decrementar cantidad
const decrementarCantidad = (productoId) => {
  const producto = carrito.value.find(item => item.id === productoId)
  if (producto) {
    if (producto.cantidad > 1) {
      producto.cantidad--
      mostrarToast('➖ Cantidad disminuida', 'Producto actualizado en el carrito', 'remove', 'orange')
    } else {
      // Si la cantidad es 1, eliminar del carrito
      const productoEliminado = productos.value.find(p => p.id === productoId)
      carrito.value = carrito.value.filter(item => item.id !== productoId)
      mostrarToast('🗑️ Producto eliminado', `${productoEliminado.nombre} removido del carrito`, 'delete', 'red')
    }
  }
}

// Función para obtener la cantidad de un producto en el carrito
const cantidadEnCarrito = (productoId) => {
  const producto = carrito.value.find(item => item.id === productoId)
  return producto ? producto.cantidad : 0
}

// Función para hacer scroll a los productos
const scrollToProductos = () => {
  const element = document.getElementById('productos-section')
  if (element) {
    element.scrollIntoView({ behavior: 'smooth' })
  }
}

// COMPUTED PROPERTIES
const totalItems = computed(() => {
  return carrito.value.reduce((total, producto) => total + producto.cantidad, 0)
})

const subtotal = computed(() => {
  return carrito.value.reduce((total, producto) => 
    total + (producto.precio * producto.cantidad), 0
  )
})

const impuesto = computed(() => {
  return subtotal.value * 0.16
})

const totalFinal = computed(() => {
  return subtotal.value + impuesto.value
})

const totalConEnvio = computed(() => {
  return mostrarEnvioGratis.value
})

// WATCHERS
// Observar cambios en el carrito (deep watch)
watch(carrito, (nuevoCarrito) => {
  console.log('Carrito actualizado:', nuevoCarrito)
  // Guardar en localStorage
  localStorage.setItem('carrito', JSON.stringify(nuevoCarrito))
}, { deep: true })



// Cargar carrito desde localStorage al inicializar
const cargarCarrito = () => {
  const carritoGuardado = localStorage.getItem('carrito')
  if (carritoGuardado) {
    try {
      carrito.value = JSON.parse(carritoGuardado)
      console.log('Carrito cargado desde localStorage:', carrito.value)
    } catch (error) {
      console.error('Error al cargar el carrito:', error)
      carrito.value = []
    }
  }
}

// Inicializar
onMounted(() => {
  cargarCarrito()
})
</script>

<style>
.titulo {
  text-align: center;
  margin-bottom: 2rem;
}

.Cards {
  padding: 1rem 0;
}

.my-card {
  border-radius: 16px;
  transition: all 0.3s ease;
  overflow: hidden;
}

.hover-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 25px rgba(0,0,0,0.15);
}

.resumen-carrito {
  margin: 2rem auto;
  max-width: 800px;
}

.detalles-carrito {
  margin-bottom: 1.5rem;
}

.detalles-carrito p {
  margin: 0.5rem 0;
}

.lista-carrito {
  border-top: 1px solid #e0e0e0;
  padding-top: 1rem;
}

.item-carrito {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0.5rem 0;
  padding: 1rem;
  background-color: #fff;
  border-radius: 12px;
  border: 1px solid #e0e0e0;
  transition: all 0.2s ease;
}

.item-carrito:hover {
  background-color: #f8f9fa;
  border-color: #2196f3;
}

.controles-cantidad {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.cantidad {
  font-weight: bold;
  font-size: 1.25rem;
  min-width: 40px;
  text-align: center;
}

.controles-producto {
  margin-top: 1rem;
  text-align: center;
}

/* Responsive */
@media (max-width: 600px) {
  .item-carrito {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  .Cards .row {
    flex-direction: column;
    align-items: center;
  }
  
  .Cards .col {
    max-width: 100%;
    width: 100%;
  }
}
</style>
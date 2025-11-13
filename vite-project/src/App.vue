<template>
  <div class="q-pa-lg">
    <!-- Banner de envío gratis cuando supera $1000 -->
    <q-banner v-if="mostrarEnvioGratis" class="bg-green text-white q-mb-md shadow-4 envio-gratis-banner">
      <template v-slot:avatar>
        <q-icon name="local_shipping" color="white" class="banner-icon" />
      </template>
      <div class="text-h5 text-weight-bold banner-title">🎉 ¡Felicidades! ¡Envío GRATIS</div>
      <div class="text-h6 banner-subtitle">Tu compra de <strong>${{ totalFinal.toFixed(2) }}</strong> califica para envío gratis</div>
      <div class="q-mt-sm banner-text">Has superado los $1000 y te llevas el envío completamente gratis</div>
      <template v-slot:action>
        <div class="banner-actions">
          <q-btn 
            flat 
            color="white" 
            label="Ver ofertas" 
            icon="local_offer"
            class="q-mb-xs"
            @click="scrollToProductos"
          />
          <q-btn 
            flat 
            color="white" 
            label="Continuar comprando" 
            icon="shopping_cart"
            class="q-mb-xs"
          />
        </div>
      </template>
    </q-banner>
    <!-- Header -->
    <div class="titulo q-mb-xl">
      <h1 class="text-h3 text-h4-sm text-h5-xs text-weight-bold text-primary q-mb-xs">🛒 Carrito de Compras</h1>
      <p class="text-h6 text-subtitle1-sm text-body1-xs text-grey-7">Selecciona tus productos favoritos</p>
    </div>
    <!-- Resumen del carrito -->
    <div class="resumen-carrito q-mb-xl" v-if="carrito.length > 0" id="resumen-carrito">
      <q-card class="bg-blue-grey-1 shadow-4">
        <q-card-section class="q-pa-lg q-pa-md-sm q-pa-xs-xs">
          <div class="text-h5 text-h6-sm text-subtitle1-xs text-weight-bold text-blue-grey-10 q-mb-md">📊 Resumen de Compra</div>
          <div class="detalles-carrito">
            <div class="row items-center q-mb-sm">
              <q-icon name="shopping_basket" class="q-mr-sm text-blue-7 resumen-icon" />
              <span class="text-h6 text-subtitle1-sm text-body1-xs">Productos en carrito: <strong class="text-primary">{{ totalItems }}</strong></span>
            </div>
            <div class="row items-center q-mb-sm">
              <q-icon name="receipt" class="q-mr-sm text-green-7 resumen-icon" />
              <span class="text-h6 text-subtitle1-sm text-body1-xs">Subtotal: <strong>${{ subtotal.toFixed(2) }}</strong></span>
            </div>
            <div class="row items-center q-mb-sm">
              <q-icon name="account_balance" class="q-mr-sm text-orange-7 resumen-icon" />
              <span class="text-h6 text-subtitle1-sm text-body1-xs">Impuesto (16%): <strong>${{ impuesto.toFixed(2) }}</strong></span>
            </div>
            <q-separator class="q-my-md" />
            <div class="row items-center">
              <q-icon name="payments" class="q-mr-sm text-red-7 total-icon" />
              <span class="text-h5 text-h6-sm text-subtitle1-xs text-weight-bold" :class="mostrarEnvioGratis ? 'text-green' : 'text-blue-grey-10'">
                Total: ${{ totalFinal.toFixed(2) }}
              </span>
              <q-icon v-if="mostrarEnvioGratis" name="celebration" color="green" class="q-ml-sm" />
            </div>
            <!-- Indicador visual del envío gratis -->
            <div class="q-mt-md">
              <div class="text-caption text-grey-7 q-mb-xs envio-text">
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
                class="q-mt-sm progress-bar"
                stripe
              >
                <div class="absolute-full flex flex-center">
                  <span class="text-white text-caption progress-text">
                    ${{ totalFinal.toFixed(0) }} / $1000
                    <span v-if="totalFinal >= 1000"> ✅</span>
                  </span>
                </div>
              </q-linear-progress>
              <!-- Mensaje de progreso -->
              <div v-if="totalFinal < 1000" class="text-center q-mt-sm">
                <q-badge color="orange" class="q-pa-sm badge-mensaje">
                  <q-icon name="info" class="q-mr-sm" size="sm" />
                  Agrega ${{ (1000 - totalFinal).toFixed(2) }} más para envío gratis
                </q-badge>
              </div>
              <!-- Mensaje de éxito -->
              <div v-else class="text-center q-mt-sm">
                <q-badge color="green" class="q-pa-xs badge-mensaje">
                  <q-icon name="celebration" class="q-mr-xs" />
                  ¡Has conseguido envío gratis!
                </q-badge>
              </div>
            </div>
          </div>
        </q-card-section>
      </q-card>
      <!-- Lista de productos en el carrito -->
      <q-card class="q-mt-lg shadow-4 ">
        <q-card-section class="q-pa-lg q-pa-md-sm q-pa-xs-xs" >
          <div class="text-h5 text-h6-sm text-subtitle1-xs text-weight-bold text-blue-grey-10 q-mb-md">
            🛍️ Productos en el Carrito 
            <q-badge v-if="mostrarEnvioGratis" color="black" class="q-ml-sm envio-badge">
              ENVÍO GRATIS
            </q-badge>
          </div>
          <div v-for="producto in carrito" :key="producto.id" class="item-carrito q-pa-md q-pa-sm-sm q-pa-xs-xs q-mb-sm rounded-borders bg-gree shadow-1">
            <div class="row items-center justify-between full-width item-carrito-content">
              <div class="col-12 col-sm-5 item-info">
                <div class="text-h6 text-subtitle1-sm text-body1-xs text-weight-medium text-blue-grey-10">{{ producto.nombre }}</div>
                <div class="text-caption text-grey-7">${{ producto.precio }} c/u</div>
              </div>
              <div class="col-12 col-sm-4 q-mt-sm q-mt-none-sm">
                <div class="controles-cantidad row items-center justify-center justify-start-sm justify-center-xs">
                  <q-btn 
                    @click="decrementarCantidad(producto.id)" 
                    icon="remove" 
                    size="sm" 
                    round 
                    dense 
                    color="primary"
                    class="shadow-1 btn-cantidad"
                  />
                  <span class="cantidad q-mx-lg text-h5 text-h6-sm text-subtitle1-xs text-weight-bold text-primary">
                    {{ producto.cantidad }}
                  </span>
                  <q-btn 
                    @click="incrementarCantidad(producto.id)" 
                    icon="add" 
                    size="sm" 
                    round 
                    dense 
                    color="primary"
                    class="shadow-1 btn-cantidad"
                  />
                </div>
              </div>
              <div class="col-12 col-sm-3 text-right text-center-sm q-mt-sm q-mt-none-sm">
                <div class="text-h5 text-h6-sm text-subtitle1-xs text-green text-weight-bold item-total">
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
      <div class="text-h4 text-h5-sm text-h6-xs text-weight-bold text-center text-blue-grey-10 q-mb-lg productos-titulo">
        🎯 Productos Disponibles
        <q-badge v-if="mostrarEnvioGratis" color="green" class="q-ml-sm envio-badge">
          ¡Sigue comprando con ENVÍO GRATIS!
        </q-badge>
      </div>
      <div class="row justify-center q-col-gutter-lg productos-grid">
        <div class="col-12 col-sm-6 col-md-4 col-lg-3" v-for="producto in productos" :key="producto.id">
          <q-card class="my-card shadow-6 bg-grey-5 hover-card" flat bordered>
            <q-card-section class="q-pa-sm text-center card-content">
              <div class="text-h2 q-mb-md product-icon"></div>
              <div class="text-h6 text-subtitle1-sm text-body1-xs text-weight-bold text-blue-grey-10 product-name">{{ producto.nombre }}</div>
              <div class="text-h4 text-h5-sm text-subtitle1-xs text-grey-10 text-weight-bold q-mt-lg q-mb-xs product-price">${{ producto.precio }}</div>
              <!-- Badge de envío gratis -->
              <div v-if="mostrarEnvioGratis" class="q-mt-sm">
                <q-badge color="green" class="q-pa-xs envio-badge">
                  <q-icon name="local_shipping" class="q-mr-xs" />
                  ENVÍO GRATIS
                </q-badge>
              </div>
            </q-card-section>
            <q-separator />
            <q-card-actions vertical class="q-pa-lg card-actions">
              <!-- Controles de cantidad para cada producto -->
              <div class="controles-producto" v-if="cantidadEnCarrito(producto.id) > 0">
                <div class="text-caption text-center text-black q-mb-sm product-in-cart">✅ En carrito: {{ cantidadEnCarrito(producto.id) }} unidad(es)</div>
                <div class="controles-cantidad row justify-center items-center product-controls">
                  <q-btn 
                    @click="decrementarCantidad(producto.id)" 
                    icon="remove" 
                    size="sm" 
                    round 
                    dense 
                    color="red-7"
                    class="shadow-2 btn-producto"
                  />
                  <span class="cantidad q-mx-lg text-h6 text-subtitle1-sm text-body1-xs text-weight-bold product-quantity">{{ cantidadEnCarrito(producto.id) }}</span>
                  <q-btn 
                    @click="incrementarCantidad(producto.id)" 
                    icon="add" 
                    size="sm" 
                    round 
                    dense 
                    color="green-9"
                    class="shadow-2 btn-producto"
                  />
                </div>
              </div>
              <q-btn 
                v-else 
                @click="agregarAlCarrito(producto)" 
                color="green-13"
                icon="add_shopping_cart"
                label="Agregar al carrito"
                class="full-width q-mt-md shadow-3 btn-agregar"
                size="lg"
              />
            </q-card-actions>
          </q-card>
        </div>
      </div>
    </div>
    <!-- Empty state -->
    <div v-if="carrito.length === 0" class="text-center q-mt-xl q-pa-xl empty-state">
      <q-icon name="shopping_cart" class="empty-icon" color="grey-4" />
      <div class="text-h5 text-subtitle1-sm text-body1-xs text-grey-6 q-mb-md empty-title">Tu carrito está vacío</div>
      <div class="text-body1 text-grey-5 empty-text">Agrega algunos productos para comenzar</div>
      <div class="q-mt-md">
        <q-badge color="green" class="q-pa-sm empty-badge">
          <q-icon name="local_shipping" class="q-mr-sm" />
          Envío gratis en compras mayores a $1000
        </q-badge>
      </div>
    </div>
    <!-- Notificación toast para acciones -->
    <q-dialog v-model="mostrarNotificacion" position="top" class="notificacion-toast">
      <q-card class="toast-card">
        <q-card-section class="row items-center no-wrap toast-content">
          <q-icon :name="notificacionIcono" :color="notificacionColor" class="toast-icon" />
          <div class="toast-text">
            <div class="text-weight-bold toast-title">{{ notificacionTitulo }}</div>
            <div class="text-caption toast-message">{{ notificacionMensaje }}</div>
          </div>
          <q-space />
          <q-btn flat round icon="close" v-close-popup class="toast-close" />
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

<style scoped>
/* Estilos base */
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
  height: 100%;
  display: flex;
  flex-direction: column;
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

/* Estilos responsivos específicos */
.envio-gratis-banner {
  padding: 16px;
}

.banner-icon {
  font-size: 32px;
}

.banner-title {
  font-size: 1.5rem;
}

.banner-subtitle {
  font-size: 1.25rem;
}

.banner-text {
  font-size: 1rem;
}

.banner-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.resumen-icon, .total-icon {
  font-size: 24px;
}

.envio-text {
  font-size: 0.8rem;
}

.progress-bar {
  height: 20px;
}

.progress-text {
  font-size: 0.8rem;
}

.badge-mensaje {
  font-size: 0.9rem;
  padding: 8px 12px;
}

.envio-badge {
  font-size: 0.8rem;
}

.item-carrito-content {
  flex-wrap: wrap;
}

.item-info {
  margin-bottom: 12px;
}

.btn-cantidad {
  width: 32px;
  height: 32px;
}

.item-total {
  font-size: 1.5rem;
}

.productos-titulo {
  margin-bottom: 24px;
}

.productos-grid {
  margin: 0 -12px;
}

.card-content {
  padding: 12px;
  flex-grow: 1;
}

.product-icon {
  font-size: 3rem;
  margin-bottom: 16px;
}

.product-name {
  font-size: 1.25rem;
}

.product-price {
  font-size: 2rem;
}

.card-actions {
  padding: 24px;
}

.product-in-cart {
  font-size: 0.8rem;
}

.product-controls {
  gap: 8px;
}

.btn-producto {
  width: 32px;
  height: 32px;
}

.product-quantity {
  font-size: 1.25rem;
  margin: 0 16px;
}

.btn-agregar {
  font-size: 1rem;
  padding: 12px;
}

.empty-state {
  padding: 48px 24px;
}

.empty-icon {
  font-size: 100px;
  margin-bottom: 16px;
}

.empty-title {
  font-size: 1.5rem;
  margin-bottom: 16px;
}

.empty-text {
  font-size: 1rem;
}

.empty-badge {
  font-size: 0.9rem;
  padding: 8px 12px;
}

.notificacion-toast {
  z-index: 9999;
}

.toast-card {
  max-width: 400px;
  width: 90vw;
}

.toast-content {
  padding: 12px;
}

.toast-icon {
  font-size: 24px;
  margin-right: 8px;
}

.toast-text {
  flex-grow: 1;
}

.toast-title {
  font-size: 1rem;
}

.toast-message {
  font-size: 0.8rem;
}

.toast-close {
  margin-left: 8px;
}

/* Media Queries para responsividad */
@media (max-width: 1023px) {
  .banner-title {
    font-size: 1.25rem;
  }
  
  .banner-subtitle {
    font-size: 1.1rem;
  }
  
  .product-price {
    font-size: 1.75rem;
  }
}

@media (max-width: 768px) {
  .envio-gratis-banner {
    padding: 12px;
  }
  
  .banner-icon {
    font-size: 28px;
  }
  
  .banner-title {
    font-size: 1.1rem;
  }
  
  .banner-subtitle {  
    font-size: 1rem;
  }
  
  .banner-text {
    font-size: 0.9rem;
  }
  
  .banner-actions {
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .resumen-icon, .total-icon {
    font-size: 20px;
  }
  
  .item-carrito {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
    padding: 16px 12px;
  }
  
  .item-info {
    width: 100%;
    margin-bottom: 8px;
  }
  
  .controles-cantidad {
    justify-content: center;
  }
  
  .item-total {
    font-size: 1.25rem;
  }
  
  .productos-grid {
    margin: 0 -8px;
  }
  
  .card-content {
    padding: 8px;
  }
  
  .product-icon {
    font-size: 2.5rem;
    margin-bottom: 12px;
  }
  
  .product-name {
    font-size: 1.1rem;
  }
  
  .product-price {
    font-size: 1.5rem;
  }
  
  .card-actions {
    padding: 16px;
  }
  
  .empty-icon {
    font-size: 80px;
  }
  
  .empty-title {
    font-size: 1.25rem;
  }
  
  .empty-text {
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .envio-gratis-banner {
    padding: 8px;
  }
  
  .banner-icon {
    font-size: 24px;
  }
  
  .banner-title {
    font-size: 1rem;
  }
  
  .banner-subtitle {
    font-size: 0.9rem;
  }
  
  .banner-text {
    font-size: 0.8rem;
  }
  
  .banner-actions {
    flex-direction: column;
    width: 100%;
  }
  
  .resumen-icon, .total-icon {
    font-size: 18px;
  }
  
  .envio-text {
    font-size: 0.7rem;
  }
  
  .progress-bar {
    height: 16px;
  }
  
  .progress-text {
    font-size: 0.7rem;
  }
  
  .badge-mensaje {
    font-size: 0.8rem;
    padding: 6px 10px;
  }
  
  .item-carrito {
    padding: 12px 8px;
  }
  
  .btn-cantidad {
    width: 28px;
    height: 28px;
  }
  
  .cantidad {
    font-size: 1.1rem;
    min-width: 30px;
  }
  
  .item-total {
    font-size: 1.1rem;
  }
  
  .productos-titulo {
    margin-bottom: 16px;
  }
  
  .productos-grid {
    margin: 0 -4px;
  }
  
  .card-content {
    padding: 6px;
  }
  
  .product-icon {
    font-size: 2rem;
    margin-bottom: 8px;
  }
  
  .product-name {
    font-size: 1rem;
  }
  
  .product-price {
    font-size: 1.25rem;
  }
  
  .card-actions {
    padding: 12px;
  }
  
  .product-in-cart {
    font-size: 0.7rem;
  }
  
  .btn-producto {
    width: 28px;
    height: 28px;
  }
  
  .product-quantity {
    font-size: 1.1rem;
    margin: 0 12px;
  }
  
  .btn-agregar {
    font-size: 0.9rem;
    padding: 10px;
  }
  
  .empty-state {
    padding: 32px 16px;
  }
  
  .empty-icon {
    font-size: 60px;
  }
  
  .empty-title {
    font-size: 1.1rem;
  }
  
  .empty-text {
    font-size: 0.8rem;
  }
  
  .empty-badge {
    font-size: 0.8rem;
    padding: 6px 10px;
  }
  
  .toast-card {
    width: 95vw;
  }
  
  .toast-content {
    padding: 8px;
  }
  
  .toast-icon {
    font-size: 20px;
  }
  
  .toast-title {
    font-size: 0.9rem;
  }
  
  .toast-message {
    font-size: 0.7rem;
  }
}
@media (max-width: 360px) {
  .banner-title {
    font-size: 0.9rem;
  }
  .banner-subtitle {
    font-size: 0.8rem;
  }
  .product-name {
    font-size: 0.9rem;
  }
  .product-price {
    font-size: 1.1rem;
  } 
  .btn-agregar {
    font-size: 0.8rem;
    padding: 8px;
  }
}
</style>
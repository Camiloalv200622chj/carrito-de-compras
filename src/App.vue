<template>
  <q-layout view="lHh Lpr fFf" class="app-root">
    <transition name="slide-down">
      <div v-if="mostrarNotificacion" class="notificacion-custom" :class="notificacion.tipo">
        <q-icon :name="notificacion.icono" size="32px" class="q-mr-md" />
        <div class="notificacion-texto">{{ notificacion.mensaje }}</div>
      </div>
    </transition>

    <q-header elevated class="header bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title class="row items-center no-wrap">
          <q-icon name="storefront" size="28px" class="q-mr-md" />
          <span class="text-weight-bold">Mi Tienda</span>
        </q-toolbar-title>

        <q-btn flat round dense icon="shopping_cart" aria-label="carrito">
          <q-badge v-if="totalItems > 0" color="amber-10" text-color="black" floating rounded>
            {{ totalItems }}
          </q-badge>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-page-container class="page-container">
      <q-page class="page-content">
        <div class="content-area row no-wrap items-stretch">

          <div class="products col-12 col-md-8">
            <q-card flat bordered class="glass-card full-card">
              <q-card-section class="q-pb-sm">
                <div class="text-h6 text-primary text-weight-bold row items-center">
                  <q-icon name="devices" size="28px" class="q-mr-sm" />
                  <span>Productos Disponibles</span>
                </div>
              </q-card-section>

              <q-card-section class="products-content">
                <div class="row q-col-gutter-sm justify-start items-stretch">
                  <div v-for="producto in productos" :key="producto.id" class="col-12 col-sm-6 col-md-4 col-lg-4">
                    <q-card class="product-card column items-center" bordered>

                      <div class="product-image">
                        <q-img
                          :src="producto.imagen"
                          :alt="producto.nombre"
                          class="product-img"
                          fit="cover"
                        >
                          <template v-slot:error>
                            <div class="absolute-full flex flex-center bg-grey-3 text-grey-8">
                              <q-icon :name="producto.icono" size="40px" />
                            </div>
                          </template>
                        </q-img>
                      </div>
                      
                      <q-card-section class="text-center product-info">
                        <div class="text-subtitle2 text-weight-bold text-primary product-name">
                          {{ producto.nombre }}
                        </div>
                        <div class="text-h6 text-weight-medium text-green-8 product-price">
                          {{ formatCurrency(producto.precio) }}
                        </div>
                      </q-card-section>
                      

                      <q-card-section class="quantity-section">
                        <div class="quantity-selector row items-center justify-center">
                          <div class="text-caption text-grey-7 q-mr-sm">Cantidad:</div>
                          <div class="row items-center no-wrap quantity-controls">
                            <q-btn 
                              round 
                              dense 
                              color="grey-6" 
                              icon="remove" 
                              size="sm"
                              :disable="getProductQuantity(producto.id) <= 1"
                              @click="decrementQuantity(producto.id)"
                            />
                            <div class="quantity-display q-mx-sm">
                              {{ getProductQuantity(producto.id) }}
                            </div>
                            <q-btn 
                              round 
                              dense 
                              color="primary" 
                              icon="add" 
                              size="sm"
                              @click="incrementQuantity(producto.id)"
                            />
                          </div>
                        </div>
                      </q-card-section>

                      <q-card-actions class="product-actions">
                        <q-btn 
                          color="primary" 
                          icon="add_shopping_cart" 
                          label="Agregar al Carrito" 
                          class="full-width"
                          size="md"
                          @click="agregarAlCarrito(producto)" 
                        />
                      </q-card-actions>
                    </q-card>
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>


          <div class="cart col-12 col-md-4">
            <q-card flat bordered class="glass-card full-card">
              <q-card-section class="q-pb-sm">
                <div class="text-h6 text-primary text-weight-bold row items-center">
                  <q-icon name="receipt_long" size="28px" class="q-mr-sm" />
                  <span>Resumen del Carrito</span>
                </div>
              </q-card-section>

              <q-card-section class="cart-content">
                <div v-if="carrito.length > 0" class="cart-body">

                  <div class="cart-items-container" :class="{ 'scrollable': carrito.length > 3 }">
                    <q-list bordered separator class="rounded-borders">
                      <q-item v-for="item in carrito" :key="item.id" class="cart-item">
                        <q-item-section avatar>
                          <q-avatar size="40px" rounded class="bg-grey-3">
                            <q-img
                              :src="item.imagen"
                              :alt="item.nombre"
                              class="cart-item-img"
                            >
                              <template v-slot:error>
                                <q-icon :name="item.icono" size="20px" color="grey-7" />
                              </template>
                            </q-img>
                          </q-avatar>
                        </q-item-section>

                        <q-item-section>
                          <q-item-label class="text-weight-medium text-primary text-caption">
                            {{ item.nombre }}
                          </q-item-label>
                          <q-item-label caption class="text-caption text-grey-7">
                            {{ formatCurrency(item.precio) }} × {{ item.cantidad }}
                          </q-item-label>
                        </q-item-section>

                        <q-item-section side class="text-subtitle2 text-weight-medium text-green-8">
                          {{ formatCurrency(item.precio * item.cantidad) }}
                        </q-item-section>

                        <q-item-section side>
                          <q-btn 
                            dense 
                            flat 
                            round 
                            color="negative" 
                            icon="remove" 
                            size="xs"
                            @click="eliminarDelCarrito(item.id)" 
                          />
                        </q-item-section>
                      </q-item>
                    </q-list>
                  </div>


                  <div class="breakdown-section">
                    <div class="breakdown-content">
                      <div class="row justify-between q-py-xs">
                        <div class="text-body2 text-grey-7">Productos</div>
                        <div class="text-body2 text-weight-medium">{{ totalItems }}</div>
                      </div>

                      <div class="row justify-between q-py-xs">
                        <div class="text-body2 text-grey-7">Subtotal</div>
                        <div class="text-body2 text-weight-medium">{{ formatCurrency(subtotal) }}</div>
                      </div>

                      <div class="row justify-between q-py-xs">
                        <div class="text-body2 text-grey-7">Impuesto (16%)</div>
                        <div class="text-body2 text-weight-medium">{{ formatCurrency(impuesto) }}</div>
                      </div>

                      <q-separator class="q-my-sm" />

                      <div class="row justify-between total-row q-py-sm">
                        <div class="text-h6 text-weight-bold text-primary">Total</div>
                        <div class="text-h6 text-weight-bold text-green-8">{{ formatCurrency(totalFinal) }}</div>
                      </div>
                      
                      <q-btn 
                        color="primary" 
                        label="Proceder al Pago" 
                        class="full-width payment-btn"
                        size="md"
                        :disable="carrito.length === 0"
                        icon="payment"
                      />
                    </div>
                  </div>
                </div>

                <div v-else class="empty-state">
                  <q-icon name="shopping_basket" size="60px" color="grey-4" />
                  <div class="text-subtitle1 text-grey-6 q-mt-md">Carrito vacío</div>
                  <div class="text-caption text-grey-5 q-mt-xs text-center">Selecciona la cantidad y agrega productos</div>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { useQuasar } from 'quasar'

export default {
  name: 'CarritoCompras',
  
  setup() {
    const $q = useQuasar()


    const productos = ref([
      { 
        id: 1, 
        nombre: 'Pc HP', 
        precio: 3200000,
        imagen: 'https://images.unsplash.com/photo-1496181133206-80ce9b88a853?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8bGFwdG9wfGVufDB8fDB8fHww&auto=format&fit=crop&w=500&q=60'
      },
      { 
        id: 2, 
        nombre: 'Mouse Gamer', 
        precio: 120000,
        imagen: 'https://images.unsplash.com/photo-1527864550417-7fd91fc51a46?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8bW91c2UlMjBnYW1lcnxlbnwwfHwwfHx8MA%3D%3D&auto=format&fit=crop&w=500&q=60'
      },
      { 
        id: 3, 
        nombre: 'Teclado Mecánico', 
        precio: 350000,
        imagen: 'https://images.unsplash.com/photo-1541140532154-b024d705b90a?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8dGVjbGFkbyUyMG1lY2FuaWNvfGVufDB8fDB8fHww&auto=format&fit=crop&w=500&q=60'
      },
      { 
        id: 4, 
        nombre: 'Camara Cannon"', 
        precio: 8500000,
        imagen: 'https://www.alkosto.com/medias/013803351422-001-750Wx750H?context=bWFzdGVyfGltYWdlc3wyMzIyMnxpbWFnZS93ZWJwfGFHVTBMMmd5T1M4eE5ETTFOakV6TURRek1UQXdOaTh3TVRNNE1ETXpOVEUwTWpKZk1EQXhYemMxTUZkNE56VXdTQXwxMjFiZmU1Njc5ZDMyZDQ5ZjU1MjJhNzlmYWZlNTk4OGZlNTRiMDM0ZTNjZjkxNGRkNjNiMWU0MTdkNjZmMDA2'
      },
      { 
        id: 5, 
        nombre: 'Auriculares Bluetooth', 
        precio: 280000,
        imagen: 'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8aGVhZHBob25lc3xlbnwwfHwwfHx8MA%3D%3D&auto=format&fit=crop&w=500&q=60'
      },
      { 
        id: 6, 
        nombre: 'Camara de seguridad', 
        precio: 75000,
        imagen: 'https://vta.vtexassets.com/arquivos/ids/176264/VTA-84723_1.jpg?v=638823948723300000'
      }
    ])

    const carrito = ref([])
    const cantidadesProductos = ref({})
    

    const mostrarNotificacion = ref(false)
    const notificacion = ref({
      mensaje: '',
      tipo: 'success',
      icono: 'check_circle'
    })
    
    const mostrarAlerta = (mensaje, tipo = 'success', icono = 'check_circle', duracion = 3000) => {
      notificacion.value = { mensaje, tipo, icono }
      mostrarNotificacion.value = true
      
      setTimeout(() => {
        mostrarNotificacion.value = false
      }, duracion)
    }


    const formatCurrency = (amount) => {
      return new Intl.NumberFormat('es-CO', {
        style: 'currency',
        currency: 'COP',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(amount)
    }


    onMounted(() => {
      const guardado = localStorage.getItem('carrito')
      if (guardado) {
        try {
          carrito.value = JSON.parse(guardado)
        } catch (e) {
          console.error('Error loading cart from localStorage:', e)
          carrito.value = []
        }
      }
      

      productos.value.forEach(producto => {
        cantidadesProductos.value[producto.id] = 1
      })
    })


    const getProductQuantity = (productId) => {
      return cantidadesProductos.value[productId] || 1
    }


    const incrementQuantity = (productId) => {
      if (!cantidadesProductos.value[productId]) {
        cantidadesProductos.value[productId] = 1
      }
      cantidadesProductos.value[productId]++
    }


    const decrementQuantity = (productId) => {
      if (cantidadesProductos.value[productId] && cantidadesProductos.value[productId] > 1) {
        cantidadesProductos.value[productId]--
      }
    }


    const agregarAlCarrito = (producto) => {
      const cantidad = getProductQuantity(producto.id)
      const existente = carrito.value.find(p => p.id === producto.id)
      
      if (existente) {
        existente.cantidad += cantidad
      } else {
        carrito.value.push({ 
          ...producto, 
          cantidad: cantidad 
        })
      }
      

      cantidadesProductos.value[producto.id] = 1
      

      mostrarAlerta(
        `✅ ${cantidad} ${producto.nombre}${cantidad > 1 ? 's' : ''} agregado${cantidad > 1 ? 's' : ''} al carrito`,
        'success',
        'check_circle',
        2000
      )
    }

    const eliminarDelCarrito = (id) => {
      const producto = carrito.value.find(p => p.id === id)
      carrito.value = carrito.value.filter(p => p.id !== id)
      
      if (producto) {
        mostrarAlerta(
          `🗑️ ${producto.nombre} eliminado del carrito`,
          'error',
          'remove_shopping_cart',
          1500
        )
      }
    }


    const totalItems = computed(() =>
      carrito.value.reduce((acc, item) => acc + item.cantidad, 0)
    )

    const subtotal = computed(() =>
      carrito.value.reduce((acc, item) => acc + item.precio * item.cantidad, 0)
    )

    const impuesto = computed(() => {
      return Math.round(subtotal.value * 0.16 * 100) / 100
    })

    const totalFinal = computed(() => {
      return Math.round((subtotal.value + impuesto.value) * 100) / 100
    })


    watch(carrito, (nuevo) => {
      localStorage.setItem('carrito', JSON.stringify(nuevo))
    }, { deep: true })


    let alertaMostrada = false
    
    watch(totalFinal, (nuevo, viejo) => {

      if (nuevo > 1000000 && (!viejo || viejo <= 1000000)) {
        if (!alertaMostrada) {
          alertaMostrada = true
          
          mostrarAlerta(
            '🎉 ¡El total supera $1.000.000! Tienes ENVÍO GRATIS 🚚',
            'envio-gratis',
            'local_shipping',
            5000
          )
        }
      }
      

      if (nuevo <= 1000000) {
        alertaMostrada = false
      }
    })

    return {
      productos,
      carrito,
      totalItems,
      subtotal,
      impuesto,
      totalFinal,
      getProductQuantity,
      incrementQuantity,
      decrementQuantity,
      agregarAlCarrito,
      eliminarDelCarrito,
      formatCurrency,
      mostrarNotificacion,
      notificacion
    }
  }
}
</script>

<style scoped>

* {
  box-sizing: border-box;
}

.app-root {
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
}

.page-container {
  height: calc(100vh - 64px);
  overflow: hidden;
}

.page-content {
  height: 100%;
  width: 100%;
  padding: 16px;
  overflow: hidden;
}

.content-area {
  width: 100%;
  margin: 0;
  gap: 16px;
  overflow: hidden;
}


.full-card {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.glass-card {
  background: rgba(255, 255, 255, 0.98);
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(0, 0, 0, 0.05);
}


.products-content {
  flex: 1;
  overflow: hidden;
  padding: 8px !important;
  display: flex;
  flex-direction: column;
}


.products-content > .row {
  flex: 1;
  overflow: hidden;
  align-content: flex-start;
}


.product-card {
  border-radius: 10px;
  transition: all 0.2s ease;
  margin-bottom: 8px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.product-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.12);
}


.product-image {
  width: 100%;
  height: 180px;
  overflow: hidden;
  flex-shrink: 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);

}

.product-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.product-info {
  padding: 4px 12px !important;
  flex-shrink: 0;
  min-height: 60px;
}

.product-name {
  line-height: 1.2;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 2px;
}

.product-price {
  font-size: 1.1rem;
}

.quantity-section {
  padding: 2px 12px !important;
  flex-shrink: 0;
  min-height: 40px;
}

.product-actions {
  padding: 4px 12px 8px 12px !important;
  flex-shrink: 0;
  margin-top: auto;
}


.quantity-selector {
  width: 100%;
  padding: 0 2px;
}

.quantity-controls {
  background: rgba(0, 0, 0, 0.03);
  border-radius: 20px;
  padding: 2px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.quantity-display {
  min-width: 24px;
  text-align: center;
  font-weight: 600;
  font-size: 0.9rem;
  color: #1976d2;
}


.cart-content {
  height: 100%;
  padding: 8px 16px 16px 16px !important;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.cart-body {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.cart-items-container {
  flex: 1;
  min-height: 0;
  overflow: hidden;
  margin-bottom: 16px;
}

.cart-items-container.scrollable {
  overflow-y: auto;
  max-height: calc(100% - 180px);
}

.cart-item {
  border-radius: 6px;
  padding: 6px 10px;
  min-height: 45px;
}

.cart-item:hover {
  background-color: rgba(0, 0, 0, 0.02);
}

.cart-item-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}


.breakdown-section {
  flex-shrink: 0;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  padding: 12px;
  border: 1px solid rgba(0, 0, 0, 0.05);
}

.payment-btn {
  margin-top: 8px;
  height: 42px;
  font-weight: 600;
  border-radius: 6px;
  font-size: 0.9rem;
}


.empty-state {
  height: 100%;
  text-align: center;
  padding: 30px 16px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}


@media (max-width: 1023px) {
  .content-area {
    flex-direction: column;
    height: auto;
    min-height: 100%;
  }
  
  .products, .cart {
    height: 50vh;
    min-height: 350px;
  }
  
  .product-card {
    height: 300px;
  }
  
  .product-image {
    height: 100px;
  }
  
  .breakdown-section {
    position: sticky;
    bottom: 0;
    background: white;
    box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
  }
}

@media (max-width: 599px) {
  .page-content {
    padding: 8px;
  }
  
  .content-area {
    gap: 12px;
  }
  
  .product-card {
    height: 280px;
  }
  
  .product-image {
    height: 90px;
  }
  
  .quantity-selector {
    flex-direction: column;
    gap: 6px;
  }
  
  .products-content {
    padding: 4px !important;
  }
  
  .product-info {
    padding: 2px 8px !important;
    min-height: 50px;
  }
  
  .quantity-section {
    padding: 1px 8px !important;
    min-height: 35px;
  }
}


:deep(.q-btn) {
  opacity: 1 !important;
  visibility: visible !important;
}

:deep(.q-btn--disabled) {
  opacity: 0.6 !important;
}

:deep(.q-card__section) {
  padding: 12px;
}


.products-content,
.products-content > .row,
.product-grid {
  overflow: hidden !important;
}


.notificacion-custom {
  position: fixed;
  top: 80px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
  display: flex;
  align-items: center;
  padding: 16px 32px;
  border-radius: 12px;
  font-size: 1.1rem;
  font-weight: 600;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
  min-width: 400px;
  max-width: 600px;
}

.notificacion-custom.success {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  color: white;
  border: 2px solid #34d399;
}

.notificacion-custom.error {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  color: white;
  border: 2px solid #f87171;
}

.notificacion-custom.envio-gratis {
  background: linear-gradient(135deg, #8b5cf6 0%, #7c3aed 100%);
  color: white;
  border: 2px solid #a78bfa;
  font-size: 1.3rem;
  padding: 20px 40px;
  animation: pulse 1s infinite;
}

.notificacion-texto {
  flex: 1;
  text-align: center;
}


.slide-down-enter-active {
  animation: slideDown 0.4s ease-out;
}

.slide-down-leave-active {
  animation: slideUp 0.3s ease-in;
}

@keyframes slideDown {
  from {
    transform: translate(-50%, -100px);
    opacity: 0;
  }
  to {
    transform: translate(-50%, 0);
    opacity: 1;
  }
}

@keyframes slideUp {
  from {
    transform: translate(-50%, 0);
    opacity: 1;
  }
  to {
    transform: translate(-50%, -100px);
    opacity: 0;
  }
}

@keyframes pulse {
  0%, 100% {
    transform: translateX(-50%) scale(1);
  }
  50% {
    transform: translateX(-50%) scale(1.02);
  }
}


@media (max-width: 599px) {
  .notificacion-custom {
    min-width: 90%;
    max-width: 90%;
    font-size: 0.95rem;
    padding: 12px 16px;
    top: 70px;
  }
  
  .notificacion-custom.envio-gratis {
    font-size: 1.05rem;
    padding: 16px 20px;
  }
}
</style>

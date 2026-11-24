<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

type Product = {
  id: string
  name: string
  category: string
  price: number
  compareAtPrice?: number
  image: string
  colors: string[]
  sizes: string[]
  stock: number
  badge?: string
  description?: string
}
type CartLine = { product: Product; size: string; quantity: number }

const demoProducts: Product[] = [
  { id: 'p-01', name: 'Vestido Alba', category: 'Vestidos', price: 42990, compareAtPrice: 54990, image: 'https://images.unsplash.com/photo-1539008835657-9e8e9680c956?auto=format&fit=crop&w=1000&q=85', colors: ['#c58272', '#262421'], sizes: ['XS', 'S', 'M', 'L'], stock: 8, badge: 'MÁS VENDIDO', description: 'Vestido midi de caída suave, pensado para acompañarte todo el día.' },
  { id: 'p-02', name: 'Blusa Lino', category: 'Blusas', price: 32990, image: 'https://images.unsplash.com/photo-1564257577-6a2b7b0f8b2b?auto=format&fit=crop&w=1000&q=85', colors: ['#e9e1d3', '#97a08e'], sizes: ['XS', 'S', 'M', 'L', 'XL'], stock: 12, description: 'Lino liviano con una silueta relajada y versátil.' },
  { id: 'p-03', name: 'Pantalón Nube', category: 'Pantalones', price: 38990, image: 'https://images.unsplash.com/photo-1594633312681-425c7b97ccd1?auto=format&fit=crop&w=1000&q=85', colors: ['#d4c8b5', '#373634'], sizes: ['XS', 'S', 'M', 'L'], stock: 6, badge: 'NUEVO', description: 'Tiro alto y pierna recta. Un esencial para combinar sin esfuerzo.' },
  { id: 'p-04', name: 'Top Brisa', category: 'Tops', price: 24990, image: 'https://images.unsplash.com/photo-1551163943-3f6a855d1153?auto=format&fit=crop&w=1000&q=85', colors: ['#d8a79c', '#f0e9df'], sizes: ['XS', 'S', 'M', 'L'], stock: 10, description: 'Un top delicado con textura sutil y detalles cuidados.' },
  { id: 'p-05', name: 'Chaqueta Siena', category: 'Chaquetas', price: 69990, image: 'https://images.unsplash.com/photo-1548624149-fc7c8f4b3e1f?auto=format&fit=crop&w=1000&q=85', colors: ['#766b59', '#b69379'], sizes: ['S', 'M', 'L', 'XL'], stock: 4, badge: 'EDICIÓN LIMITADA', description: 'Una capa ligera para esos días que empiezan frescos.' },
  { id: 'p-06', name: 'Falda Serena', category: 'Faldas', price: 35990, image: 'https://images.unsplash.com/photo-1583496661160-fb5886a0aaaa?auto=format&fit=crop&w=1000&q=85', colors: ['#a77d70', '#2d302c'], sizes: ['XS', 'S', 'M', 'L'], stock: 7, description: 'Falda de movimiento fluido, fácil de llevar de día o de noche.' },
  { id: 'p-07', name: 'Camisa Clara', category: 'Blusas', price: 37990, image: 'https://images.unsplash.com/photo-1598554747436-c9293d6a588f?auto=format&fit=crop&w=1000&q=85', colors: ['#f1eee8', '#aeb6a4'], sizes: ['S', 'M', 'L', 'XL'], stock: 9, description: 'Camisa atemporal en algodón suave con calce ligeramente amplio.' },
  { id: 'p-08', name: 'Vestido Oliva', category: 'Vestidos', price: 47990, image: 'https://images.unsplash.com/photo-1612336307429-8a898d10e223?auto=format&fit=crop&w=1000&q=85', colors: ['#8e9278', '#322f2c'], sizes: ['XS', 'S', 'M', 'L'], stock: 5, badge: 'NUEVO', description: 'Silueta elegante y cómoda, con un tono natural que combina con todo.' },
]

const products = ref<Product[]>(demoProducts)
const cart = ref<CartLine[]>(JSON.parse(localStorage.getItem('amara-cart') || '[]'))
const category = ref('Todo')
const query = ref('')
const selectedProduct = ref<Product | null>(null)
const selectedSize = ref('')
const cartOpen = ref(false)
const sendingOrder = ref(false)
const notice = ref('')
const orderName = ref('')
const orderContact = ref('')

const categories = computed(() => ['Todo', ...new Set(products.value.map((p) => p.category))])
const visibleProducts = computed(() => products.value.filter((p) =>
  (category.value === 'Todo' || p.category === category.value) &&
  p.name.toLowerCase().includes(query.value.toLowerCase()),
))
const cartCount = computed(() => cart.value.reduce((n, line) => n + line.quantity, 0))
const subtotal = computed(() => cart.value.reduce((n, line) => n + line.product.price * line.quantity, 0))
const money = (value: number) => new Intl.NumberFormat('es-CL', { style: 'currency', currency: 'CLP', maximumFractionDigits: 0 }).format(value)

onMounted(async () => {
  try {
    const response = await fetch(`${import.meta.env.VITE_API_BASE_URL || ''}/api/products`)
    if (!response.ok) throw new Error('API no disponible')
    const data = await response.json() as Product[]
    if (data.length) products.value = data
  } catch {
    // Demo de respaldo para que el catálogo también funcione sin iniciar la API.
  }
})

function focusSearch() {
  document.getElementById('search')?.focus()
}
function openQuickAdd(product: Product) {
  selectedProduct.value = product
  selectedSize.value = ''
}
function addToCart() {
  const product = selectedProduct.value
  if (!product || !selectedSize.value) return
  const existing = cart.value.find((line) => line.product.id === product.id && line.size === selectedSize.value)
  if (existing) existing.quantity += 1
  else cart.value.push({ product, size: selectedSize.value, quantity: 1 })
  persistCart()
  selectedProduct.value = null
  notice.value = 'Agregado a tu bolsa'
  window.setTimeout(() => notice.value = '', 2200)
}
function changeQuantity(line: CartLine, delta: number) {
  line.quantity += delta
  if (line.quantity <= 0) cart.value = cart.value.filter((item) => item !== line)
  persistCart()
}
function persistCart() {
  localStorage.setItem('amara-cart', JSON.stringify(cart.value))
}
async function placeOrder() {
  if (!cart.value.length || !orderName.value.trim() || !orderContact.value.trim()) return
  sendingOrder.value = true
  try {
    const response = await fetch(`${import.meta.env.VITE_API_BASE_URL || ''}/api/orders`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ customerName: orderName.value, contact: orderContact.value, items: cart.value.map((line) => ({ productId: line.product.id, size: line.size, quantity: line.quantity })) }),
    })
    const result = await response.json()
    if (!response.ok) throw new Error(result.error || 'No se pudo registrar el pedido')
    cart.value = []
    persistCart()
    orderName.value = ''
    orderContact.value = ''
    cartOpen.value = false
    notice.value = `Pedido ${result.id} registrado. Pago pendiente; lo integramos más adelante.`
    window.setTimeout(() => notice.value = '', 5000)
  } catch (error) {
    notice.value = error instanceof Error ? error.message : 'No se pudo registrar el pedido'
    window.setTimeout(() => notice.value = '', 3500)
  } finally {
    sendingOrder.value = false
  }
}
</script>

<template>
  <div class="announcement">ENVÍO GRATIS EN COMPRAS SOBRE $80.000 <span>✦</span> HECHO PARA DURAR</div>
  <header class="site-header">
    <a class="wordmark" href="#inicio" aria-label="Amara inicio">AMARA<span>®</span></a>
    <nav class="main-nav"><a href="#coleccion">NUEVA COLECCIÓN</a><a href="#coleccion">PRENDAS</a><a href="#historia">NUESTRA ESENCIA</a></nav>
    <div class="header-actions"><button class="search-toggle" @click="focusSearch" aria-label="Buscar">⌕</button><button class="bag-button" @click="cartOpen = true">BOLSA <span>{{ cartCount }}</span></button></div>
  </header>

  <main id="inicio">
    <section class="hero">
      <img class="hero-image" src="https://images.unsplash.com/photo-1483985988355-763728e1935b?auto=format&fit=crop&w=2200&q=88" alt="Mujeres caminando con looks de temporada" />
      <div class="hero-shade"></div>
      <div class="hero-copy"><p class="eyebrow">COLECCIÓN PRIMAVERA · VERANO</p><h1>Vestir a tu<br /><em>manera.</em></h1><p>Prendas para los días que se sienten tuyos.</p><a class="hero-link" href="#coleccion">DESCUBRE LA COLECCIÓN <span>↘</span></a></div>
      <div class="hero-index">01 / 03</div>
      <div class="hero-caption">SANTIAGO, CHILE · EST. 2025</div>
    </section>

    <section class="intro" id="historia"><p class="eyebrow">MENOS, PERO MEJOR</p><h2>Un guardarropa que<br /><em>se siente como tú.</em></h2><p>Diseñamos piezas versátiles, cómodas y hechas para acompañarte temporada tras temporada.</p></section>

    <section class="collection" id="coleccion">
      <div class="collection-heading"><div><p class="eyebrow">PIEZAS PARA TODOS TUS DÍAS</p><h2>La colección <span>01</span></h2></div><label class="search-box"><span>⌕</span><input id="search" v-model="query" placeholder="Buscar prendas" aria-label="Buscar prendas" /></label></div>
      <div class="filters"><button v-for="item in categories" :key="item" :class="{ active: category === item }" @click="category = item">{{ item }}</button><span class="result-count">{{ visibleProducts.length }} PRENDAS</span></div>
      <div v-if="visibleProducts.length" class="product-grid">
        <article v-for="(product, index) in visibleProducts" :key="product.id" class="product-card">
          <button class="product-visual" @click="openQuickAdd(product)" :aria-label="`Ver ${product.name}`">
            <img :src="product.image" :alt="product.name" :loading="index > 3 ? 'lazy' : 'eager'" />
            <span v-if="product.badge" class="product-badge">{{ product.badge }}</span>
            <span class="quick-add">VER PRENDA <b>↗</b></span>
          </button>
          <div class="product-info"><div><h3>{{ product.name }}</h3><p>{{ product.category }}</p></div><div class="price-wrap"><span v-if="product.compareAtPrice" class="old-price">{{ money(product.compareAtPrice) }}</span><strong>{{ money(product.price) }}</strong></div></div>
          <div class="swatches"><span v-for="(color, i) in product.colors" :key="color" :style="{ backgroundColor: color }" :aria-label="`Color ${i + 1}`"></span><small>{{ product.colors.length }} colores</small></div>
        </article>
      </div>
      <div v-else class="empty-results">No encontramos prendas con ese nombre. Prueba otra búsqueda.</div>
      <div class="collection-footer"><span>01 — 08</span><span>HECHO PARA VIVIRLO</span><span>✳</span></div>
    </section>

    <section class="note-banner"><div class="note-image"></div><div class="note-content"><p class="eyebrow">ELIGE CON INTENCIÓN</p><h2>Lo cotidiano,<br /><em>extraordinario.</em></h2><p>Materiales agradables, siluetas cómodas y detalles que importan. Una forma más consciente de vestirte cada día.</p><a href="#coleccion">CONOCE NUESTRAS PRENDAS <span>↗</span></a></div></section>
  </main>
  <footer><a class="wordmark" href="#inicio">AMARA<span>®</span></a><p>Diseñado en Chile. Hecho para acompañarte.</p><div class="footer-bottom"><span>© 2025 AMARA STUDIO</span><span>INSTAGRAM&nbsp;&nbsp; · &nbsp;&nbsp;CONTACTO</span><span>ES / CL</span></div></footer>

  <div v-if="selectedProduct" class="modal-backdrop" @click.self="selectedProduct = null" @keydown.esc="selectedProduct = null">
    <section class="quick-modal" role="dialog" aria-modal="true" :aria-label="selectedProduct.name">
      <button class="close-button" @click="selectedProduct = null" aria-label="Cerrar">×</button>
      <img :src="selectedProduct.image" :alt="selectedProduct.name" />
      <div class="quick-details"><p class="eyebrow">{{ selectedProduct.category }}</p><h2>{{ selectedProduct.name }}</h2><strong>{{ money(selectedProduct.price) }}</strong><p class="quick-description">{{ selectedProduct.description }}</p><p class="size-label">ELIGE TU TALLA <span>GUÍA DE TALLAS ↗</span></p><div class="sizes"><button v-for="size in selectedProduct.sizes" :key="size" :class="{ chosen: selectedSize === size }" :disabled="selectedProduct.stock <= 0" @click="selectedSize = size">{{ size }}</button></div><p class="stock-note">{{ selectedProduct.stock > 0 ? `${selectedProduct.stock} unidades disponibles` : 'Agotado' }}</p><button class="add-button" :disabled="!selectedSize || selectedProduct.stock <= 0" @click="addToCart">{{ selectedSize ? 'AGREGAR A LA BOLSA' : 'SELECCIONA UNA TALLA' }} <span>↗</span></button></div>
    </section>
  </div>

  <div v-if="cartOpen" class="drawer-backdrop" @click.self="cartOpen = false">
    <aside class="cart-drawer" aria-label="Tu bolsa">
      <div class="drawer-heading"><div><p class="eyebrow">TU SELECCIÓN</p><h2>Tu bolsa <span>({{ cartCount }})</span></h2></div><button class="close-button" @click="cartOpen = false" aria-label="Cerrar">×</button></div>
      <div v-if="!cart.length" class="cart-empty"><span>✳</span><h3>Tu bolsa está esperando.</h3><p>Encuentra esa prenda que se siente como tú.</p><button class="add-button" @click="cartOpen = false">EXPLORAR PRENDAS ↗</button></div>
      <template v-else>
        <div class="cart-lines"><article v-for="line in cart" :key="line.product.id + line.size" class="cart-line"><img :src="line.product.image" :alt="line.product.name" /><div class="line-info"><h3>{{ line.product.name }}</h3><p>Talla {{ line.size }}</p><strong>{{ money(line.product.price) }}</strong><div class="quantity"><button @click="changeQuantity(line, -1)" aria-label="Restar">−</button><span>{{ line.quantity }}</span><button @click="changeQuantity(line, 1)" aria-label="Sumar">+</button></div></div><button class="remove-line" @click="changeQuantity(line, -line.quantity)" aria-label="Eliminar">×</button></article></div>
        <div class="cart-checkout"><p class="subtotal"><span>Subtotal</span><strong>{{ money(subtotal) }}</strong></p><p class="shipping-copy">Envío e impuestos se calculan al confirmar.</p><label class="form-label">TU NOMBRE<input v-model="orderName" placeholder="Nombre y apellido" /></label><label class="form-label">EMAIL O TELÉFONO<input v-model="orderContact" placeholder="Para coordinar tu pedido" /></label><button class="add-button" :disabled="sendingOrder || !orderName.trim() || !orderContact.trim()" @click="placeOrder">{{ sendingOrder ? 'REGISTRANDO…' : 'CONFIRMAR PEDIDO · PAGO PENDIENTE' }} <span>↗</span></button><p class="payment-note">Los pagos se habilitarán próximamente. Por ahora registraremos tu pedido sin cobrar.</p></div>
      </template>
    </aside>
  </div>
  <Transition name="toast"><div v-if="notice" class="toast-message">{{ notice }}</div></Transition>
</template>

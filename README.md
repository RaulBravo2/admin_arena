# AMARA storefront (Vue 3 + TypeScript)

Vitrina inicial para una tienda de ropa de mujer. Incluye catálogo, filtros por categoría, búsqueda, selector de talla, carrito persistente en el navegador y registro de pedidos sin cobrar. La marca y las fotografías son demostrativas y deben cambiarse por las definitivas.

## Desarrollo local

```bash
npm install
npm run dev
```

Vite reenvía `/api` a `http://127.0.0.1:3001`. Inicia también la API en el repositorio `api_arena`. El frontend tiene productos de respaldo para que se pueda revisar aunque la API esté apagada.

Para apuntar a una API desplegada en otro dominio, crea `.env.local` con `VITE_API_BASE_URL=https://tu-api.example.com`.

## Compilar

```bash
npm run build
```

Los pagos aún no están conectados intencionalmente. El flujo actual crea pedidos con estado pendiente de pago.

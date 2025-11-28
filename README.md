# Clase-Práctica - Equipo 3
## 🛒 Sesión Práctica: Comparación de Precios con MockAPI + Postman

## 🎯 Objetivo
Crear una API simulada con MockAPI para manejar productos y precios, y utilizar Postman para realizar el ciclo completo de gestión: **Crear, Consultar, Actualizar y Visualizar/Comparar precios gráficamente**.

---

## ⭐ PARTE 1 — MockAPI (Configuración del Backend)

### 1. Crear proyecto
1. Ve a: [https://mockapi.io](https://mockapi.io)
2. Crea una cuenta o inicia sesión.
3. Pulsa el botón **+ New Project**.
4. **Name:** `ComparadorPrecios`
5. **API Prefix:** `/Api/Tiendas`
6. Pulsa **Create**.
7. Haz clic sobre el proyecto recién creado para entrar.

### 2. Crear recurso "products"
1. Haz clic en el botón **New Resource**.
2. En "Resource Name" escribe: `products`.
3. Configura los campos (elimina los existentes si no coinciden):
    * `product` → Tipo: **Faker.js** → Valor: `commerce.product`
    * `price` → Tipo: **Faker.js** → Valor: `commerce.price`
    * `store` → Tipo: **Faker.js** → Valor: `company.name` (o puedes dejarlo como string vacío si prefieres llenarlo manual).
4. Haz clic en el botón **Create**.
5. Da clic en la url de encima y cópiala.

### 3. Generar datos de prueba
1. Regresa a MockApi.
2. En la fila de tu recurso "products", cambia el número (por defecto dice 0) y arrastra la barra hasta llegar a **10**.
3. Esto creará 10 productos aleatorios para empezar.

---

## ⭐ PARTE 2 — Postman (Consumo y Gestión)

### 4. Crear Colección
1. Abre Postman.
2. Ve a **Workspaces** → **Create Workspace** → **Blank Workspace** → **Next** → Nombre: "Taller API" → **Create**.
3. En el panel izquierdo, haz clic en **Collections**.
4. Clic en el botón **+** (o "Create new collection").
5. Renombra la colección como: `ComparadorPrecios`.

### 5. Request GET: Obtener productos
1. Dentro de tu colección, clic en los tres puntos `...` → **Add Request**.
2. Renómbralo a: `GET - Ver Productos`.
3. Método: **GET**.
4. **URL:** Pega la URL que copiaste de MockAPI.
5. Clic en **Send**.
6. **Verificación:** Deberías ver una lista JSON con los productos generados.

### 6. Request POST: Agregar productos para la COMPARACIÓN
Aquí agregaremos dos productos idénticos en tiendas diferentes para la prueba final.

#### Paso A: Crear el Producto 1
1. Add Request → Nombre: `POST - Producto Caro`.
2. Método: **POST**.
3. **URL:** La misma URL de MockAPI.
4. Pestaña **Body** → **raw** → **JSON**.
5. Pega el siguiente código:

```json
{
  "Product": "Laptop A",
  "Price": 550,
  "Store": "Tienda A"
}
```
6. Clic en **Send**.

#### Paso B: Crear el Producto 2 (Mismo producto, mejor precio)
1. Crea otro Request → Nombre: `POST - Producto Barato`.
2. Método: **POST**.
3. **URL:** La misma URL.
4. Pestaña **Body** → **raw** → **JSON**.
5. Pega el siguiente código (nota que el precio y la tienda cambian):

```json
{
  "Product": "Laptop A",
  "Price": 490,
  "Store": "Tienda B"
}
```
6. Clic en **Send**.

### 7. Request PUT: Actualizar precio
1. Add Request → Nombre: `PUT - Actualizar Precio`.
2. Método: **PUT**.
3. **URL:** Tu URL + `/ID` (ej. .../products/1).
4. **Body** → **raw** → **JSON**:

```json
{
  "Price": 450
}
```
5. Clic en **Send**.

---

## ⭐ PARTE 3 — Visualización y Gráfica de Barras

Usaremos la herramienta **Visualizer** de Postman. Esto nos permitirá ver una gráfica comparativa de precios directamente en la respuesta.

1. Ve a tu petición `GET - Ver Productos`.
2. Haz clic en la pestaña **Scripts** que está debajo de la barra de URL.
3. Pega el siguiente código en el editor:

```javascript
// ==========================================
// CONFIGURACIÓN DE BÚSQUEDA
// ==========================================
// Escribe aquí el nombre del producto que quieres comparar.
// Si lo dejas vacío (""), mostrará TODOS los productos.
// Ejemplo: "Laptop A" o "Ball"
var PRODUCTO_BUSCADO = "Laptop A";
// ==========================================
function obtenerValor(objeto, nombrePropiedad) {
    if (!objeto) return null;

    var llaveEncontrada = Object.keys(objeto).find(key =>
        key.toLowerCase() === nombrePropiedad.toLowerCase()
    );
    return llaveEncontrada ? objeto[llaveEncontrada] : null;
}
var respuesta = pm.response.json();
var listaCompleta = Array.isArray(respuesta) ? respuesta : [respuesta];
var productosFiltrados = listaCompleta.filter(function(item) {
    if (PRODUCTO_BUSCADO === "") return true;

    var nombreItem = obtenerValor(item, "product") || "";
    return nombreItem.toLowerCase().includes(PRODUCTO_BUSCADO.toLowerCase());
});
// 3. PREPARAR DATOS PARA LA GRÁFICA
var nombresData = JSON.stringify(productosFiltrados.map(p => {
    var nombre = obtenerValor(p, "product") || "Sin Nombre";
    var tienda = obtenerValor(p, "store") || "Tienda X";
    return nombre + " (" + tienda + ")";
}));
var preciosData = JSON.stringify(productosFiltrados.map(p => {
    var precio = obtenerValor(p, "price");
    return Number(precio) || 0;
}));
var coloresData = JSON.stringify(productosFiltrados.map(p => {
    var precio = Number(obtenerValor(p, "price")) || 0;
    return precio > 500 ? 'rgba(255, 99, 132, 0.7)' : 'rgba(75, 192, 192, 0.7)';
}));
var template = `
<div style="background-color: #f4f4f4; padding: 20px; font-family: 'Segoe UI', sans-serif;">
<h2 style="text-align: center; color: #333;">
📊  Comparativa: ${PRODUCTO_BUSCADO === "" ? "Todos los Productos" : PRODUCTO_BUSCADO}
</h2>

<div style="background: white; padding: 15px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
<canvas id="myChart" height="400"></canvas>
</div>
</div>

<script src="[https://cdn.jsdelivr.net/npm/chart.js](https://cdn.jsdelivr.net/npm/chart.js)"></script>

<script>
var ctx = document.getElementById('myChart').getContext('2d');

// Datos recibidos desde Postman
var etiquetas = {{{nombres}}};
var datos = {{{precios}}};
var colores = {{{colores}}};
// Configuración de la gráfica
var myChart = new Chart(ctx, {
type: 'bar',
data: {
labels: etiquetas,
datasets: [{
label: 'Precio ($)',
data: datos,
backgroundColor: colores,
borderColor: '#666',
borderWidth: 1,
borderRadius: 4
}]
},
options: {
responsive: true,
maintainAspectRatio: false,
plugins: {
legend: { display: false },
title: {
display: true,
text: etiquetas.length === 0 ? ' ⚠️  No se encontraron productos con ese nombre' : ''
}
},
scales: {
y: {
beginAtZero: true,
ticks: { callback: function(value) { return '$' + value; } }
}
}
}
});
</script>
`;
pm.visualizer.set(template, {
nombres: nombresData,
precios: preciosData,
colores: coloresData
});
```

### ¿Cómo ver la gráfica?
1. Después de pegar el código, haz clic en **Send** nuevamente en el GET - Ver Productos.
2. En la parte inferior (donde ves la respuesta JSON), busca y haz clic en la pestaña que dice **Visualize**.
3. ¡Ahí verás tu gráfica de barras ordenada! Podrás ver visualmente la diferencia entre el "Laptop A" de la Tienda A y el de la Tienda B.

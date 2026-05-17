# Información inicial Odoo Express
## 1. ¿Qué es el módulo de Inventario?

Es un **sistema de seguimiento de partida doble**, basado en el mismo principio de la contabilidad financiera. Nada se pierde, todo se mueve entre ubicaciones.
Este módulo es importante para lo operativo, alimenta directamente a la contabilidad (Costos de Ventas, Activos Corrientes) y asegura que los estados financieros reflejen la realidad física del almacén en tiempo real.

----------

## 2. Flujo Estándar del Módulo: Desde el Producto hasta el Reporte

### A. Estructura Inicial: Almacenes y Ubicaciones

-   **Almacén (Warehouse):** Es el edificio o recinto físico (Ej. "Central Santa Cruz").
    
-   **Ubicaciones (Locations):** Son zonas específicas dentro del almacén. Odoo las divide en:
    
    -   _Ubicaciones Internas:_ Donde guardamos la mercadería real.
        
    -   _Ubicaciones Virtuales (Desecho/Producción):_ Para pérdidas o transformación.
        
    -   _Ubicaciones de Socios (Proveedores/Clientes):_ Puntos de origen y destino externos.
        

> 📸 **Captura sugerida aquí:** Ir a _Inventario > Configuración > Almacenes_ o _Ubicaciones_. Mostrar la vista de lista con la estructura jerárquica de ubicaciones (Ej: `WH/Existencias`). 🎥 **Enlace a Video sugerido:** `[Video: Configuración de Estructura de Almacenes en Odoo 17](https://www.odoo.com/slides/inventory-24)` (Enlace oficial de Odoo Academy).

### B. El Producto: Tipos y Configuración Crítica

Se debe entender los tipos de producto en Odoo 17:

1.  **Almacenable (Storable):** El sistema lleva un control estricto de cantidades físicas, genera movimientos de existencias y activa la valoración contable.
    
2.  **Consumible (Consumable):** Se recibe físicamente pero el sistema asume que la existencia siempre es suficiente. No genera alertas de stock.
    
3.  **Servicio (Service):** Intangible. No genera movimientos de almacén.
    

**Categorías de Productos y Mapeo Contable (Clave para tu sistema):** La categoría define las reglas del juego. Para que la contabilidad se genere de forma idéntica a Enterprise, la categoría debe configurarse así:

-   **Método de coste:** _FIFO (First In, First Out / PEPS - Primero en Entrar, Primero en Salir)_ o _Precio Medio Ponderado (PMP)_.
    
-   **Valoración de inventario:** _Automatizado_. Esto es lo que activa los asientos contables en tiempo real al mover inventario.
    
-   **Propiedades de la cuenta de inventario (Cuentas Contables):**
    
    -   _Cuenta de valoración de inventario:_ Cuenta de Activo (Mercaderías/Inventario Central).
        
    -   _Cuenta de entrada de inventario (Transitoria):_ Cuenta de pasivo transitorio (Bienes recibidos no facturados). Se llena al recibir el producto de una Compra.
        
    -   _Cuenta de salida de inventario (Transitoria):_ Cuenta de activo/pasivo transitorio (Bienes entregados no facturados). Se llena al validar la entrega de una Venta.
        
    -   _Cuenta de ingresos / Gastos:_ Cuentas estándar del estado de resultados (Ventas / Costo de Ventas).
        

> 📸 **Captura sugerida aquí:** Entrar a _Inventario > Configuración > Categorías de productos_. Seleccionar una categoría y tomar captura mostrando claramente la sección "Valoración de inventario" en "Automatizado" y las cuentas contables asociadas mapeadas para Bolivia.

### C. Métodos de Remoción (Estrategias de Salida)

Configurados a nivel de categoría o ubicación, determinan qué unidades salen primero:

-   **FIFO (PEPS):** El primer producto que ingresó (el más antiguo) es el primero en ser seleccionado para la venta. Evita la obsolescencia.
    
-   **LIFO (UEPS):** El último en entrar es el primero en salir (Poco usado, restringido por normativas contables según el caso).
    
-   **FEFO (First Expired, First Out):** Basado en **fechas de caducidad**. Odoo seleccionará automáticamente el lote cuya fecha de vencimiento esté más próxima.
    

### D. Flujo Operativo: Compra, Recepción, Venta y Entrega

#### 1. Abastecimiento (Compra y Recepción)

Al validar una _Orden de Compra (PO)_ y hacer la **Recepción**, Odoo genera un documento tipo `WH/IN`. Al validarlo:

-   Las existencias físicas suben.
    
-   **Efecto Contable Automático:** Se debita la _Cuenta de valoración de inventario_ (Activo ↑) y se acredita la _Cuenta transitoria de entrada de inventario_ (Pasivo ↑).
    

#### 2. Despacho (Venta y Entrega)

Al confirmar un _Pedido de Venta (SO)_, se genera una **Orden de Entrega** tipo `WH/OUT`. Al validar el despacho:

-   Las existencias físicas bajan.
    
-   **Efecto Contable Automático:** Se debita la _Cuenta transitoria de salida de inventario_ (Activo ↓) y se acredita la _Cuenta de valoración de inventario_ (Activo ↓).
    

#### 3. ¿En qué momento se registra el Costo de Ventas (Efectivo)?

El costo real del producto se calcula con el método de coste elegido (ej. FIFO) en el momento del despacho. Sin embargo, el asiento definitivo de **Costo de Ventas contra Cuentas Transitorias** se consolida en el sistema cuando se valida la **Factura de Venta**. Al validar la factura, la cuenta transitoria se cierra y se convierte en el gasto real (Costo de Mercadería Vendida).

> 📸 **Captura sugerida aquí:** Un diagrama del flujo de pantallas de Odoo 17: Compra -> Recepción (Botón Recibir) -> Venta -> Entrega (Botón Validar). 🎥 **Enlace a Video sugerido:** `[Video: Flujo de Recepciones y Entregas en Odoo 17](https://www.odoo.com/slides/slide/receipts-and-deliveries-2815)`

### E. Trazabilidad: Lotes y Fechas de Caducidad

Para productos que requieren control estricto (alimentos, químicos, tecnología):

-   **Lote (Lot):** Un número de identificación único asignado a un conjunto de productos que se produjeron o recibieron juntos.
    
-   **Fechas de Caducidad:** Al activar "Fechas de caducidad" en la configuración, el producto almacenable exigirá registrar:
    
    -   _Fecha de caducidad:_ El producto es peligroso/inútil.
        
    -   _Fecha de fin de vida:_ Fin de su consumo óptimo.
        
    -   _Fecha de alerta:_ Cuándo el sistema debe notificar al equipo de almacén.
        

> 📸 **Captura sugerida aquí:** Ficha de un producto en la pestaña "Inventario", mostrando la sección "Trazabilidad" con la opción "Por lotes" activa y los campos de fechas habilitados.

### F. Reportes Clave: Valoración de Inventario

El reporte más importante para tu equipo de implementación es la **Valoración de Inventario (Inventory Valuation)**.

-   Muestra la cantidad de productos a una fecha determinada, multiplicada por su costo (según FIFO o PMP).
    
-   _Validación Crucial de Auditoría:_ El total de este reporte **DEBE** coincidir exactamente con el saldo de la cuenta de Activo "Mercaderías" en el Balance General. Si hay diferencias, es porque el equipo hizo un asiento manual en la cuenta o modificó un costo de forma indebida.
    

----------

## 3. Configuraciones Importantes a tener en cuenta en Odoo 17

Para lograr que el flujo anterior funcione sin errores, el implementador debe activar las siguientes casillas en **Inventario > Configuración > Ajustes**:

1.  **Rutas multietapa (Multi-Step Routes):** Obligatorio si el cliente no hace recepciones directas, sino que usa control de calidad o recepción en dos pasos (Recibir + Almacenar).
    
2.  **Ubicaciones de almacenamiento (Storage Locations):** Permite activar la gestión de pasillos, estantes y ubicaciones virtuales.
    
3.  **Lotes y números de serie (Lots & Serial Numbers):** Abre la pestaña de trazabilidad en los productos.
    
4.  **Fechas de caducidad (Expiration Dates):** Habilita las alertas y bloqueos por vencimiento.
    

----------

## 4. Anexos (Casos Especiales y Flujos Extras)

### Caso Especial 1: Inventario Físico / Ajustes de Inventario

Cuando hay diferencias entre el sistema y la realidad, no se hacen compras o ventas ficticias. Se utiliza la pantalla **Ajustes de Inventario**.

-   _Flujo en Odoo 17:_ Se va a _Inventario > Operaciones > Ajustes de inventario_. Odoo 17 unificó esto en una sola pantalla tipo hoja de cálculo donde modificas la "Cantidad contada" y aplicas el cambio.
    
-   _Impacto Contable:_ Si faltan productos, Odoo hace un asiento automático enviando el valor del costo del producto a una cuenta de "Pérdidas por Inventario".
    

### Caso Especial 2: Costos en Destino (Landed Costs)

Muy común en Bolivia para importaciones desde Arica o Iquique. Si un producto cuesta $10, pero el flete y el desaduanamiento costaron $2 por unidad:

-   Se utiliza el módulo de **Costos en Destino** para cargar esos $2 directamente al valor del inventario (haciendo que el producto ahora valga $12 en la Valoración de Inventario).
    
-   _Nota técnica:_ Solo aplica para productos con método de coste FIFO o PMP y Valoración Automatizada.

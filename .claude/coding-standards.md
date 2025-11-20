# Estándares de Codificación XP

Este documento contiene los estándares de código que sigo como agente XP. Estos estándares son esenciales para mantener el código simple, legible y mantenible según los valores de Extreme Programming.

## Nombres

- Pronunciables en inglés, sin abreviaturas técnicas (abreviaturas solo permitidas en expresiones lambda de alcance reducido)
- Evita prefijos/sufijos redundantes (`I`, `Impl`, `Abstract`)
- Concretos; evita palabras-comodín (helper, util, service…) salvo que el dominio lo exija
- Sin información de tipo; el IDE ya la muestra
- Sin alias ni sinónimos para el mismo concepto: un concepto, un nombre
- Combinan bien con la gramática del lenguaje: `isPaidInvoice`, `sumOfNumbersIn(expression)`
- Preferible usar "not" en el nombre a usar el operador de negación
- Distingue sustantivos (clases, módulos) de verbos (métodos, funciones)
- Sufijos genéricos admitidos: DTO, Repository, Factory, Mapper, UseCase, Service
- Evita comentarios si puedes usar un nombre autoexplicativo. Comentarios solo en casos extremos que el código no pueda explicar

### Ejemplos

```typescript
// ⚠️ PEOR
const d = 42; // días
const usrCtrl = new UsrCtrl();
const IUserRepository = ...;
const fetchUserInfo = ...;

// ✅ MEJOR
const daysUntilExpiration = 42;
const userAuthenticator = new UserAuthenticator();
const UserRepository = ...;
const findUser = ...;
```

## Funciones

1. **Optimiza para simplicidad y una sola responsabilidad (SRP)**. Cada función debe "hacer exactamente lo que su nombre indica". Las funciones de 10-15 líneas son una métrica para detectar si tienen más de una responsabilidad; si no la tienen, no importa el tamaño

2. **Nomenclatura impecable**. Los nombres de funciones deben ser **verbos** que describan la acción con precisión

3. **Firma mínima necesaria**. Reduce **aridad** (0–3 parámetros es el ideal). Si se excede, agrúpalos en un objeto con un nombre adecuado y tipado

4. **Evita parámetros de configuración**. No uses `boolean` (ni banderas, ni flags) que cambien comportamientos. Prefiere **funciones específicas** (`show()/hide()`, `switchToReadMode()/switchToWriteMode()`)

5. **Parámetros opcionales con moderación**. Máximo **uno**; si puedes evitarlos, mejor. Recuerda la explosión combinatoria y su impacto en tests

6. **Control de flujo simple**. Usa **cláusulas de guarda** para casos límite y **sal pronto**; reduce indentación y complejidad ciclomática

7. **Condiciones legibles**:
   - Abstrae **expresiones booleanas combinadas** en **variables explicativas** o **funciones** (`const isOnSale = ...`)
   - **Prioriza condiciones afirmativas**; si la negación mejora la claridad, encapsúlala en el nombre (`doesNotExist()`)

8. **Separa control de flujo y lógica de negocio**. Extrae iteraciones y bifurcaciones; delega cálculos/mutaciones en funciones u objetos dedicados

9. **Prefiere estilo declarativo cuando mejore la lectura**. Usa `map/filter/reduce/forEach` con criterio; no es dogma. Si un `for` es más claro, úsalo

10. **Fomenta funciones puras y transparencia referencial**. Evita efectos secundarios cuando sea posible

11. **Aplica CQS (Command–Query Separation)**:
    - **Comandos**: mutan estado y **no devuelven** valor
    - **Consultas**: **devuelven** valor y **no mutan** estado
    - Conoce **excepciones justificadas** (p. ej., crear y devolver `Id` al insertar)

12. **Equilibrio rendimiento–legibilidad**. Optimiza solo cuando sea necesario. Prioriza claridad

13. **No añadas comentarios a la función**

14. **No mutes colecciones**. Evita push, pop, shift, unshift, splice. Prefiere operaciones inmutables que devuelvan nuevos arrays

### Ejemplos

**Parámetros booleanos:**
```typescript
// ⚠️ PEOR
function render(showDetails: boolean) {
  if (showDetails) { /* ... */ }
  else { /* ... */ }
}

// ✅ MEJOR
function renderWithDetails() { /* ... */ }
function renderSummary() { /* ... */ }
```

**CQS (Command-Query Separation):**
```typescript
// ⚠️ PEOR - Query que muta estado
function totalWithDiscount(percentage: number): number {
  this.appliedDiscount = percentage;
  return this.total * (1 - percentage / 100);
}

// ✅ MEJOR - Comando y Query separados
function applyDiscount(percentage: number): void { 
  this.appliedDiscount = percentage; 
}
function calculateTotal(): number { 
  return this.baseTotal * (1 - this.appliedDiscount / 100); 
}
```

## Clases y Módulos

1. **Ámbito mínimo para máxima cohesión**. Si un método o función solo usa una constante y nadie más la usa, ponla dentro de la función

2. **Constructores simples**. Si tienen lógica de validación en la creación, muévela a un método factoría y haz el constructor privado. Si no hay validación compleja, no crear método factoría innecesario

3. **Organización de clase**: Constructor(es) públicos primero (si existen), seguido del constructor privado, luego la API pública, finalmente métodos privados. En TypeScript, las propiedades pueden auto-inyectarse en el constructor

4. **Encapsulación por defecto**. Usa `private` para métodos y no exportes funciones a menos que sea necesario. Limita la accesibilidad

5. **Aplica Ley de Demeter y Tell, Don't Ask**

6. **Evita modelos anémicos**. Las clases deben encapsular comportamiento, salvo en la frontera de la aplicación donde puedes usar DTOs

7. **Objetos completos en construcción**. Evita setters y limita los getters. Los objetos deben estar completamente inicializados al crearse

8. **Nunca uses singletons**. Si una instancia tiene estado global, debe ser gestionada por el módulo factoría de la aplicación

9. **Composición sobre herencia**. Evita la herencia, prioriza la composición

10. **Tipos específicos de dominio**. Construye clases con comportamiento, especialmente en el dominio de la aplicación

### Ejemplos

**Tell, Don't Ask y Ley de Demeter:**
```typescript
// ⚠️ PEOR - Ask (preguntamos y decidimos fuera)
if (order.customer().address().city() === "Madrid") {
  order.applyDiscount(10);
}

// ✅ MEJOR - Tell (le decimos qué hacer)
order.applyDiscountForCity("Madrid", 10);

// ⚠️ PEOR - Violación Ley de Demeter (cadena larga)
const city = user.account().settings().location().city();

// ✅ MEJOR - Ley de Demeter (un solo punto)
const city = user.city();
```

**Modelo Anémico vs Modelo Rico:**
```typescript
// ⚠️ PEOR - Modelo anémico
class Order {
  items: Item[];
  total: number;
}
// Lógica fuera
const total = order.items.reduce((sum, item) => sum + item.price, 0);

// ✅ MEJOR - Modelo rico
class Order {
  private items: Item[];
  
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}
```

## Tests

**Nomenclatura:**
- Nombres en inglés
- Representan reglas de negocio, no detalles de implementación
- Descriptivos: qué se está probando y qué se espera
- Evita nombres técnicos o acoplados a la implementación

**Estructura AAA (Arrange-Act-Assert):**
- **Arrange**: Preparar el contexto y datos necesarios
- **Act**: Ejecutar la acción a probar
- **Assert**: Verificar el resultado esperado
- Separar visualmente las tres secciones (línea en blanco entre ellas)

**Ejemplos:**

```typescript
// ⚠️ PEOR - Acoplado a implementación, no describe negocio
test('calculatePrice returns 90', () => {
  const result = calculatePrice(100, 10);
  expect(result).toBe(90);
});

// ✅ MEJOR - Describe regla de negocio
test('calculates price with discount applied to given product', () => {
  const originalPrice = 100;
  const discountPercentage = 10;

  const finalPrice = calculateDiscountedPrice(originalPrice, discountPercentage);

  expect(finalPrice).toBe(90);
});
```

---

**Nota**: Estos estándares se aplican en conjunto con las prácticas XP definidas en [claude.md](claude.md), especialmente durante el refactoring continuo y el diseño simple.
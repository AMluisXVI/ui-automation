# UI Automation — Urban Routes

[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python)](.)
[![Selenium](https://img.shields.io/badge/Selenium-WebDriver-43B02A?style=flat-square&logo=selenium)](.)
[![pytest](https://img.shields.io/badge/pytest-0A9EDC?style=flat-square&logo=pytest)](.)
[![POM](https://img.shields.io/badge/POM-Page%20Object%20Model-2E5FA3?style=flat-square)](.)
[![CDP](https://img.shields.io/badge/CDP-Network%20Interception-FF6C37?style=flat-square)](.)
[![Status](https://img.shields.io/badge/Status-Complete-green?style=flat-square)](.)

🇺🇸 [English](#-english) · 🇪🇸 [Español](#-español)

---

## 🇺🇸 English

### The Problem

**The manual tests on Urban Routes found bugs. But running them manually every deploy takes too long. And the SMS verification step blocks automation entirely.**

You can automate address entry. You can automate clicking buttons. But when the app sends a real SMS code to a real phone number, Selenium can't read it. Most automation projects stop here — they test "everything except auth" and leave a gap in coverage.

Urban Routes had 11 manual steps in the taxi order flow. Step 4 was a blocker: "Enter SMS code." To automate the full E2E flow, you need to intercept that code from the network layer — not type it manually.

### Why This Matters

Manual E2E testing of the taxi order flow takes ~15 minutes per run. That's 15 minutes every time a deploy happens. With automation, the same flow runs in under 2 minutes — and runs overnight, on weekends, on every commit.

But partial automation is almost useless. Testing address entry without testing the SMS flow means the most fragile part of the app — authentication — is never validated. The CDP interception technique solves this: **it reads the OTP directly from Chrome's network logs** so the automation can complete the full flow without human intervention.

### The Approach — Scope & Cuts

I automated **the full taxi order flow** — 11 steps, 9 test assertions:

| Step | Action | Validation |
|---|---|---|
| 1 | Open app | Page loads |
| 2 | Enter From / To addresses | Fields retain input |
| 3 | Select "Comfort" fare | Tariff selected |
| 4 | Enter phone → request SMS → **intercept code via CDP** → enter code | Code entered |
| 5 | Add bank card | Card form submits |
| 6 | Write driver message | Message retained |
| 7 | Select blanket + tissues | Checkboxes checked |
| 8 | Select 2 ice creams | Counter updates |
| 9 | Submit order | Car search modal appears |
| 10 | Wait for driver assignment | Driver info panel appears |

**What was cut:**
- Only 1 flow (Comfort fare) — not all tariff types
- No cross-browser — Chrome only. Selenium with CDP is Chrome-specific.
- No mobile — web-only flow
- No negative scenarios — scope was happy-path E2E to establish the baseline

> 11 steps, 9 assertions, 1 end-to-end flow. Not exhaustive — but the most technically complex flow (SMS auth) is covered, which is where most automation fails.

### Tools — Chosen by Need

| Tool | Why, Not Just What |
|---|---|
| **Selenium WebDriver** | The industry standard for browser automation. Handles dynamic elements, waits, and page transitions. |
| **pytest** | Lightweight test runner with clear assertions and verbose output. |
| **Chrome DevTools Protocol (CDP)** | Chosen because the app uses real SMS verification. CDP intercepts the network response containing the OTP — no workaround needed. |
| **Page Object Model (POM)** | Separates page logic (locators, actions) from test logic (assertions). Makes tests readable and maintainable. |

CDP was the critical choice. Without it, the SMS step would require manual input or a fake phone number backend — neither is a real solution. Intercepting the code from the network layer means **the automation handles auth exactly like a real user does**, just faster.

### How It Was Broken Down

9 test functions, each covering one UI action. They run in sequence because the flow is sequential (you can't submit an order before entering addresses):

1. **`test_set_route`** — Address fields. Foundation for everything else.
2. **`test_select_plan`** — Tariff selection. One click, one assertion.
3. **`test_fill_phone_number`** — Phone entry + SMS request. The CDP interception happens here.
4. **`test_fill_card`** — Card form. Tests that the payment UI works.
5. **`test_comment_for_driver`** — Message field. Text input validation.
6. **`test_order_blanket_and_handkerchiefs`** — Checkbox extras.
7. **`test_order_2_ice_creams`** — Counter interaction (2 clicks).
8. **`test_car_search_model_appears`** — Post-submit modal. The critical assertion.
9. **`test_driver_info_appears`** — Final state. Driver assigned.

Each test is small and focused. The Page Object Model keeps locators and actions separate from assertions.

### Project Structure — Separation of Concerns

```
qa-project-Urban-Routes/
├── data.py     # Test data: URL, addresses, phone, card, driver message
├── main.py     # Page object class + pytest test cases
└── README.md
```

| File | Responsibility |
|---|---|
| `data.py` | All test data in one place. Change an address? Edit here, not in tests. |
| `main.py` | `UrbanRoutesPage` class (locators + actions) + test functions (assertions). Two concerns, one file, clearly separated. |

The OTP interception via CDP is a single function — `retrieve_phone_code()` — that reads Chrome's performance logs, finds the SMS network response, and extracts the numeric code. It's the most technically significant part of the project.

### The Results

**All 9 tests pass:**
```
PASSED  test_set_route
PASSED  test_select_plan
PASSED  test_fill_phone_number
PASSED  test_fill_card
PASSED  test_comment_for_driver
PASSED  test_order_blanket_and_handkerchiefs
PASSED  test_order_2_ice_creams
PASSED  test_car_search_model_appears
PASSED  test_driver_info_appears
```

| Flow Step | Covered |
|---|---|
| Route entry | ✅ |
| Tariff selection | ✅ |
| Phone + SMS (with CDP) | ✅ |
| Card payment | ✅ |
| Driver message | ✅ |
| Extras (blanket, tissues, ice cream) | ✅ |
| Order submission | ✅ |
| Driver assignment | ✅ |

The full E2E flow is automated. It runs in under 2 minutes. It handles real SMS verification without manual intervention. It can be integrated into a CI/CD pipeline.

---

## 🇪🇸 Español

### El Problema

**Las pruebas manuales de Urban Routes encontraron bugs. Pero ejecutarlas manualmente en cada deploy toma demasiado. Y el paso de verificación SMS bloquea la automatización por completo.**

Podés automatizar la entrada de direcciones. Podés automatizar los clics en botones. Pero cuando la app envía un código SMS real a un número de teléfono real, Selenium no puede leerlo. La mayoría de los proyectos de automatización se detienen acá — prueban "todo excepto auth" y dejan un gap en la cobertura.

Urban Routes tenía 11 pasos manuales en el flujo de pedido de taxi. El paso 4 era un bloqueador: "Ingresar código SMS." Para automatizar el flujo E2E completo, necesitás interceptar ese código desde la capa de red — no tipearlo manualmente.

### Por Qué Importa

Una prueba E2E manual del flujo de pedido de taxi toma ~15 minutos por ejecución. Son 15 minutos cada vez que hay un deploy. Con automatización, el mismo flujo se ejecuta en menos de 2 minutos — y se ejecuta de noche, los fines de semana, en cada commit.

Pero la automatización parcial es casi inútil. Probar la entrada de direcciones sin probar el flujo SMS significa que la parte más frágil de la app — la autenticación — nunca se valida. La técnica de intercepción CDP resuelve esto: **lee el OTP directamente de los logs de red de Chrome** para que la automatización complete el flujo completo sin intervención humana.

### El Enfoque — Alcance y Recortes

Automaticé **el flujo completo de pedido de taxi** — 11 pasos, 9 aserciones:

| Paso | Acción | Validación |
|---|---|---|
| 1 | Abrir app | La página carga |
| 2 | Ingresar direcciones Origen / Destino | Los campos retienen el valor |
| 3 | Seleccionar tarifa "Comfort" | Tarifa seleccionada |
| 4 | Ingresar teléfono → pedir SMS → **interceptar código via CDP** → ingresar código | Código ingresado |
| 5 | Agregar tarjeta bancaria | Formulario de tarjeta enviado |
| 6 | Escribir mensaje para el conductor | Mensaje retenido |
| 7 | Seleccionar manta + pañuelos | Checkboxes marcados |
| 8 | Seleccionar 2 helados | Contador actualizado |
| 9 | Confirmar pedido | Modal de búsqueda aparece |
| 10 | Esperar asignación de conductor | Panel de info del conductor aparece |

**Lo que se recortó:**
- Solo 1 flujo (tarifa Comfort) — no todos los tipos de tarifa
- Sin cross-browser — solo Chrome. Selenium con CDP es específico de Chrome.
- Sin mobile — flujo web solamente
- Sin escenarios negativos — el alcance fue happy-path E2E para establecer la línea base

> 11 pasos, 9 aserciones, 1 flujo E2E. No es exhaustivo — pero el flujo más complejo técnicamente (auth SMS) está cubierto, que es donde la mayoría de la automatización falla.

### Stack — Elegido por Necesidad

| Herramienta | Por Qué, No Solo Qué |
|---|---|
| **Selenium WebDriver** | El estándar de la industria para automatización de navegadores. Maneja elementos dinámicos, waits, y transiciones. |
| **pytest** | Test runner liviano con aserciones claras y salida verbose. |
| **Chrome DevTools Protocol (CDP)** | Elegido porque la app usa verificación SMS real. CDP intercepta la respuesta de red que contiene el OTP — sin workarounds. |
| **Page Object Model (POM)** | Separa la lógica de página (localizadores, acciones) de la lógica de prueba (aserciones). Tests legibles y mantenibles. |

CDP fue la elección crítica. Sin esto, el paso SMS requeriría entrada manual o un backend de número falso — ninguna es una solución real. Interceptar el código desde la capa de red significa que **la automatización maneja auth exactamente como un usuario real**, solo que más rápido.

### Cómo se Fragmentó

9 funciones de prueba, cada una cubriendo una acción UI. Se ejecutan en secuencia porque el flujo es secuencial:

1. **`test_set_route`** — Campos de dirección. Fundación para todo lo demás.
2. **`test_select_plan`** — Selección de tarifa. Un clic, una aserción.
3. **`test_fill_phone_number`** — Entrada de teléfono + solicitud SMS. La intercepción CDP ocurre acá.
4. **`test_fill_card`** — Formulario de tarjeta. Valida que la UI de pago funciona.
5. **`test_comment_for_driver`** — Campo de mensaje. Validación de entrada de texto.
6. **`test_order_blanket_and_handkerchiefs`** — Extras tipo checkbox.
7. **`test_order_2_ice_creams`** — Interacción con contador (2 clics).
8. **`test_car_search_model_appears`** — Modal post-confirmación. La aserción crítica.
9. **`test_driver_info_appears`** — Estado final. Conductor asignado.

Cada test es chico y enfocado. El Page Object Model mantiene localizadores y acciones separados de las aserciones.

### Estructura del Proyecto — Separación de Responsabilidades

```
qa-project-Urban-Routes/
├── data.py     # Datos de prueba: URL, direcciones, teléfono, tarjeta, mensaje
├── main.py     # Page Object (localizadores + acciones) + funciones de prueba (aserciones)
└── README.md
```

| Archivo | Responsabilidad |
|---|---|
| `data.py` | Todos los datos de prueba en un lugar. ¿Cambiás una dirección? Editá acá, no en los tests. |
| `main.py` | `UrbanRoutesPage` (localizadores + acciones) + funciones de prueba (aserciones). Dos preocupaciones, un archivo, claramente separadas. |

La intercepción OTP via CDP es una función — `retrieve_phone_code()` — que lee los logs de rendimiento de Chrome, encuentra la respuesta de red del SMS, y extrae el código numérico. Es la parte técnicamente más significativa del proyecto.

### Los Resultados

**Los 9 tests pasan:**
```
PASSED  test_set_route
PASSED  test_select_plan
PASSED  test_fill_phone_number
PASSED  test_fill_card
PASSED  test_comment_for_driver
PASSED  test_order_blanket_and_handkerchiefs
PASSED  test_order_2_ice_creams
PASSED  test_car_search_model_appears
PASSED  test_driver_info_appears
```

| Paso del flujo | Cubierto |
|---|---|
| Ingreso de ruta | ✅ |
| Selección de tarifa | ✅ |
| Teléfono + SMS (con CDP) | ✅ |
| Pago con tarjeta | ✅ |
| Mensaje al conductor | ✅ |
| Extras (manta, pañuelos, helado) | ✅ |
| Confirmación de pedido | ✅ |
| Asignación de conductor | ✅ |

El flujo E2E completo está automatizado. Se ejecuta en menos de 2 minutos. Maneja verificación SMS real sin intervención manual. Listo para integrarse en un pipeline CI/CD.

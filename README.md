# 🚕 Urban Routes — UI Test Automation

**TripleTen QA Engineering Bootcamp · Cohort 23**
👤 Luis Manco · Medellín, Colombia

[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python)](.)
[![Selenium](https://img.shields.io/badge/Selenium-WebDriver-43B02A?style=flat-square&logo=selenium)](.)
[![pytest](https://img.shields.io/badge/pytest-0A9EDC?style=flat-square&logo=pytest)](.)
[![POM](https://img.shields.io/badge/POM-Page%20Object%20Model-2E5FA3?style=flat-square)](.)
[![Status](https://img.shields.io/badge/Status-Complete-green?style=flat-square)](.)

🇺🇸 [English](#-english) · 🇪🇸 [Español](#-español)

---

## 🇺🇸 English

### Project Description

This project contains automated end-to-end UI tests for the **Urban Routes** taxi booking application. Tests are written in Python using Selenium WebDriver and the **Page Object Model (POM)** design pattern, covering the complete taxi order flow — from address entry to driver assignment confirmation.

A key technical feature is the interception of the SMS phone verification code directly from Chrome's network logs using the **Chrome DevTools Protocol (CDP)**, eliminating the need for manual input during automation.

### Tech Stack

| Tool | Purpose |
|------|---------|
| Python 3 | Primary language |
| Selenium WebDriver | Browser automation |
| pytest | Test runner |
| ChromeDriver | Chrome browser driver |
| Chrome DevTools Protocol (CDP) | Network log interception for OTP retrieval |
| Page Object Model (POM) | Test architecture pattern |

### Project Structure

```
qa-project-Urban-Routes/
├── data.py     # Test data: URL, addresses, phone, card, driver message
├── main.py     # Page object class + pytest test cases
└── README.md   # This document
```

### Test Flow

The automated tests cover the full taxi order sequence:

```
1. Open Urban Routes app
2. Enter From / To addresses
3. Select "Comfort" fare
4. Enter phone number → request SMS code
5. Intercept OTP code via CDP → enter confirmation code
6. Add bank card (number + CVV)
7. Write message for driver
8. Select extras: blanket, tissues, 2 ice creams
9. Submit order
10. Assert: car search modal appears
11. Assert: driver info panel appears
```

### Test Cases

| Test | What It Validates |
|------|------------------|
| `test_set_route` | Both address fields accept and retain input |
| `test_select_plan` | Comfort tariff can be selected |
| `test_fill_phone_number` | Phone field accepts number and code request fires |
| `test_fill_card` | Card form submits successfully |
| `test_comment_for_driver` | Message field accepts and retains text |
| `test_order_blanket_and_handkerchiefs` | Both checkboxes can be checked |
| `test_order_2_ice_creams` | Ice cream button is interactable after 2 clicks |
| `test_car_search_model_appears` | Car search modal appears after order submit |
| `test_driver_info_appears` | Driver info panel loads within timeout |

### OTP Interception via CDP

The SMS verification step uses a real code sent by the server. The `retrieve_phone_code()` function intercepts it directly from Chrome's performance logs using the Chrome DevTools Protocol.

**How it works:**
1. Chrome is launched with `'performance'` logging enabled
2. After triggering the SMS request, it scans the log buffer for a network response matching `api/v1/number?number`
3. It calls the CDP command `Network.getResponseBody` to read the response payload
4. It strips all non-digit characters and returns the numeric OTP

### How to Run

```bash
# 1. Clone the repository
git clone https://github.com/AMluisXVI/ui-automation.git
cd UI-Test-Automation-Framework-for-Urban-Routes

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate  # macOS / Linux
# venv\Scripts\activate   # Windows

# 3. Install dependencies
pip install selenium pytest

# 4. Run all tests
pytest main.py -v
```

**Expected output:**
```
PASSED  main.py::TestUrbanRoutes::test_set_route
PASSED  main.py::TestUrbanRoutes::test_select_plan
PASSED  main.py::TestUrbanRoutes::test_fill_phone_number
PASSED  main.py::TestUrbanRoutes::test_fill_card
PASSED  main.py::TestUrbanRoutes::test_comment_for_driver
PASSED  main.py::TestUrbanRoutes::test_order_blanket_and_handkerchiefs
PASSED  main.py::TestUrbanRoutes::test_order_2_ice_creams
PASSED  main.py::TestUrbanRoutes::test_car_search_model_appears
PASSED  main.py::TestUrbanRoutes::test_driver_info_appears
```

### Key Concepts Applied

| Concept | Where Used |
|---------|-----------|
| Page Object Model | `UrbanRoutesPage` separates locators and actions from test logic |
| Separation of concerns | `data.py` / page object / test class — three distinct layers |
| Explicit waits | `WebDriverWait` + `presence_of_element_located` |
| CDP network interception | `retrieve_phone_code()` reads OTP from Chrome performance logs |
| Multi-selector strategy | ID, CSS Selector, XPath, Class Name, Name |

---

## 🇪🇸 Español

### Descripción del Proyecto

Este proyecto contiene pruebas automatizadas E2E de UI para la aplicación de reserva de taxis **Urban Routes**. Las pruebas están escritas en Python usando Selenium WebDriver con el patrón **Page Object Model (POM)**, cubriendo el flujo completo de pedido de taxi — desde la entrada de direcciones hasta la confirmación de asignación del conductor.

Una característica técnica clave es la intercepción del código de verificación SMS directamente desde los logs de red de Chrome usando el **Chrome DevTools Protocol (CDP)**, eliminando la necesidad de entrada manual durante la automatización.

### Stack Tecnológico

| Herramienta | Propósito |
|-------------|-----------|
| Python 3 | Lenguaje principal |
| Selenium WebDriver | Automatización del navegador |
| pytest | Ejecutor de pruebas |
| ChromeDriver | Controlador del navegador Chrome |
| CDP | Intercepción de logs de red para OTP |
| Page Object Model (POM) | Patrón de arquitectura de pruebas |

### Estructura del Proyecto

```
qa-project-Urban-Routes/
├── data.py     # Datos de prueba: URL, direcciones, teléfono, tarjeta
├── main.py     # Page Object + clases de prueba pytest
└── README.md   # Este documento
```

### Flujo de Prueba

```
1. Abrir Urban Routes
2. Ingresar direcciones Origen / Destino
3. Seleccionar tarifa "Comfort"
4. Ingresar número de teléfono → solicitar código SMS
5. Interceptar código OTP via CDP → ingresar código
6. Agregar tarjeta bancaria
7. Escribir mensaje para el conductor
8. Seleccionar extras: manta, pañuelos, 2 helados
9. Confirmar pedido
10. Verificar: modal de búsqueda de conductor aparece
11. Verificar: panel de información del conductor aparece
```

### Cómo Ejecutar

```bash
# 1. Clonar el repositorio
git clone https://github.com/AMluisXVI/ui-automation.git
cd UI-Test-Automation-Framework-for-Urban-Routes

# 2. Crear y activar entorno virtual
python -m venv venv
source venv/bin/activate

# 3. Instalar dependencias
pip install selenium pytest

# 4. Ejecutar todas las pruebas
pytest main.py -v
```

### Conceptos Clave Aplicados

| Concepto | Dónde se Usó |
|----------|-------------|
| Page Object Model | `UrbanRoutesPage` separa localizadores y acciones de la lógica de prueba |
| Separación de responsabilidades | `data.py` / page object / test class — tres capas distintas |
| Esperas explícitas | `WebDriverWait` + `presence_of_element_located` |
| Intercepción CDP | `retrieve_phone_code()` lee OTP de logs de rendimiento de Chrome |
| Estrategia multi-localizador | ID, CSS Selector, XPath, Class Name, Name |

---

> 📚 Project developed as part of the **TripleTen QA Engineering Bootcamp** · Sprint 9 · Cohort 23

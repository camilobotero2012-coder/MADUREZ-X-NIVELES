# 📊 Diagramas Visuales y Matriz de Flujos

## 1️⃣ DIAGRAMA DE FLUJO COMPLETO (Detallado)

```
┌──────────────────────────────────────────────────────────────┐
│                   USUARIO NUEVO LLEGA                        │
│                   (Sin registro previo)                       │
└──────────────────────┬───────────────────────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────┐
        │ App.js carga las rutas       │
        │ SessionProvider lo envuelve  │
        │ AuthContext valida sesión    │
        └──────────────┬───────────────┘
                       │
           ┌───────────┴───────────┐
           │                       │
        NO │                    SÍ │
       (sin│                (con   │
      sesión)              sesión)│
           │                       │
           ▼                       ▼
    ┌─────────────┐       ┌──────────────┐
    │isAuth = F   │       │ isAuth = T   │
    └──────┬──────┘       └──────┬───────┘
           │                     │
           │                     ▼
           │            ┌────────────────┐
           │            │Navigate /survey│
           │            │(automática)    │
           │            └────────────────┘
           │
           ▼
    ┌─────────────────────┐
    │ Usuario navega a:   │
    │ - /landing          │
    │ - /register         │
    │ - intenta /survey   │
    └────────┬────────────┘
             │
    ┌────────┴─────────────────────┐
    │                              │
    ▼ /landing u /register         ▼ /survey (intento)
    
    ✓ SE MUESTRA                   ❌ BLOQUEADO
    FORMULARIO                     🚫 REDIRIGE /register
                                   ⚠️  "Primero registrate"
    
         │
         ▼
    Usuario llena:
    - Nombre empresa
    - Industria
    - Tamaño
    - País
    - Nombre usuario
    - Email
         │
         ▼
    ┌──────────────────┐
    │ Valida localmente │
    │ (todos campos OK?)│
    └────┬─────────────┘
         │
    ┌────┴─────────────┐
    │                  │
    NO │              SÍ│
    (error)        (ok)│
    │                  │
    ▼                  ▼
┌────────────┐   ┌──────────────────┐
│Muestra     │   │POST /api/companies│
│error en    │   │/register         │
│formulario  │   │{company, user}   │
└────────────┘   └────┬─────────────┘
                      │
                ┌─────┴──────┐
                │            │
             NO │         SÍ │
           (error)      (ok) │
                │            │
                ▼            ▼
          ┌──────────┐  ┌─────────────────┐
          │Muestra   │  │Backend guarda:  │
          │error API │  │ - Empresa ✓     │
          │"Intenta  │  │ - Usuario ✓     │
          │después"  │  │ - Genera IDs    │
          └──────────┘  │ - Retorna {id}  │
                        └────┬────────────┘
                             │
                             ▼
                  ┌───────────────────────┐
                  │ Frontend recibe:      │
                  │ - company.id = "xxxx" │
                  │ - user.id = "yyyy"    │
                  └───────┬───────────────┘
                          │
                          ▼
              ┌─────────────────────────┐
              │ Crea objeto session:    │
              │ {                       │
              │   isAuthenticated: true │
              │   companyId: "xxxx"     │
              │   userId: "yyyy"        │
              │   companyName: "..."    │
              │   userName: "..."       │
              │   email: "..."          │
              │   industry: "..."       │
              │   country: "..."        │
              │ }                       │
              └───────┬─────────────────┘
                      │
                      ▼
        ┌──────────────────────────┐
        │localStorage.setItem(     │
        │'session', JSON.stringify)│
        └───────┬──────────────────┘
                │
                ▼
      ┌───────────────────────┐
      │ setSession(sessionData)│
      │ login(sessionData)     │
      │ setIsAuthenticated(true)
      └───────┬───────────────┘
              │
              ▼
    ┌──────────────────────┐
    │navigate('/survey')   │
    │(automática)          │
    │Mostrar: "Bienvenido" │
    └──────────┬───────────┘
               │
               ▼
    ┌────────────────────────┐
    │ USUARIO EN ENCUESTA    │
    │ Survey.jsx cargado     │
    │ ✓ Tiene acceso         │
    │ ✓ Datos en sesión      │
    │ ✓ companyId, userId    │
    │ ✓ Puede guardar respuestas
    └───────────────────────┘
```

---

## 2️⃣ MATRIZ DE DECISIÓN (Decision Tree)

```
╔════════════════════════════════════════════════════════════════╗
║                    ¿QUÉ SUCEDE EN CADA CASO?                  ║
╚════════════════════════════════════════════════════════════════╝

┌─ USUARIO NUEVO (Sin registro)
│  ├─ Va a /landing
│  │  └─ ✓ Ve Landing page normalmente
│  │
│  ├─ Va a /register
│  │  └─ ✓ Ve formulario para registrarse
│  │
│  ├─ Va a /survey
│  │  ├─ AuthContext valida: ¿localStorage tiene 'session'?
│  │  │  └─ NO
│  │  ├─ ProtectedRoute revisa: ¿isAuthenticated = true?
│  │  │  └─ NO
│  │  ├─ ProtectedRoute redirige: navigate('/register')
│  │  └─ ❌ BLOQUEADO - Redirigido a /register
│  │       Mensaje: "Primero debes registrar tu empresa"
│  │
│  └─ Intenta guardar respuesta POST /survey/responses
│     └─ Backend valida: ¿company_id existe?
│        └─ NO
│        └─ 404 "Empresa no encontrada"
│
├─ USUARIO EN REGISTRO
│  └─ PresionaButton "Registrar"
│     ├─ Valida localmente
│     │  ├─ ✓ Todos campos obligatorios lleados
│     │  └─ ✗ Falta campos → Muestra errores
│     ├─ POST /api/companies/register
│     ├─ Espera respuesta
│     └─ Si ✓ success → auto login + navigate /survey
│
├─ USUARIO REGISTRADO (Con sesión válida)
│  ├─ Va a /landing
│  │  ├─ useEffect detecta: ¿isAuthenticated = true?
│  │  │  └─ SÍ
│  │  └─ navigate('/survey', {replace: true})
│  │     (No ve Landing, va directo a encuesta)
│  │
│  ├─ Va a /register
│  │  └─ ✓ Puede verla (no hay protección)
│  │     Podría volver a registrar (considerar: ¿debo impedir esto?)
│  │
│  ├─ Va a /survey
│  │  ├─ ProtectedRoute valida: ¿isAuthenticated = true?
│  │  │  └─ SÍ
│  │  └─ ✓ ACCESO PERMITIDO
│  │     Renderiza Survey.jsx con datos de sesión
│  │
│  ├─ Responde preguntas
│  │  └─ POST /survey/responses
│  │     ├─ Envía: {company_id, user_id, question_id, answer}
│  │     ├─ Backend valida company_id → ✓ Existe
│  │     └─ Guarda respuesta en BD
│  │
│  └─ Click "Cerrar sesión"
│     ├─ logout()
│     ├─ localStorage.removeItem('session')
│     ├─ setIsAuthenticated(false)
│     ├─ navigate('/landing')
│     └─ ✓ Vuelve a Landing sin sesión
│
└─ EDGE CASES
   ├─ localStorage corrupto
   │  └─ useEffect intenta parse → Error
   │     └─ localStorage.removeItem() → sesión limpia
   │
   ├─ SessionID falso
   │  ├─ localStorage.setItem('session', fakeData)
   │  └─ POST /survey/responses {company_id: 'fake'}
   │     └─ Backend: 404 "No existe" → Frontend: logout + redirige
   │
   ├─ Sesión muy antigua (>30 días)
   │  └─ Validación: fecha (logout) + redirige /register
   │
   └─ Múltiples tabs del navegador
      ├─ Tab 1: Registra → localStorage se actualiza
      ├─ Tab 2: Detecta cambio → Actualiza estado
      └─ Ambas tabs sincronizadas
```

---

## 3️⃣ CICLO DE VIDA DE SESIÓN

```
┌────────────────────────────────────────────────────────────┐
│          CICLO DE VIDA DE LA SESIÓN DEL USUARIO            │
└────────────────────────────────────────────────────────────┘

1. ANTES DE REGISTRARSE
   ├─ localStorage: vacío ∅
   ├─ AuthContext: {isAuthenticated: false, session: null}
   ├─ Rutas protegidas: BLOQUEADAS
   └─ Estado: "SIN_AUTENTICACION"

2. USUARIO VA A /register
   ├─ Rellena formulario
   ├─ LocalStorage: vacío ∅
   ├─ Estado: "RELLENANDO_FORMULARIO"
   └─ Botón "Registrar" esperando click

3. USUARIO HACE CLICK EN "REGISTRAR"
   ├─ Frontend valida form
   ├─ POST /api/companies/register
   ├─ Estado: "REGISTRANDO" (loading = true)
   └─ Esperando respuesta del backend

4. BACKEND PROCESA REGISTRO
   ├─ Crea empresa (BD)
   ├─ Crea usuario (BD)
   ├─ Genera company_id
   ├─ Genera user_id
   └─ Retorna response con IDs

5. FRONTEND RECIBE RESPUESTA
   ├─ Crea objeto session con:
   │  ├─ companyId
   │  ├─ userId
   │  ├─ companyName
   │  ├─ userName
   │  ├─ email
   │  └─ otros datos...
   ├─ localStorage.setItem('session', JSON.stringify(session))
   ├─ AuthContext.login(session)
   ├─ isAuthenticated = true
   └─ Estado: "REGISTRADO"

6. FRONTEND REDIRIGE A /survey
   ├─ ProtectedRoute se evalúa
   ├─ ¿isAuthenticated = true? SÍ ✓
   ├─ Renderiza Survey.jsx
   └─ Estado: "EN_ENCUESTA"

7. USUARIO COMPLETA ENCUESTA
   ├─ Para cada respuesta:
   │  ├─ POST /api/survey/responses
   │  ├─ Envía {company_id, user_id, question_id, answer}
   │  └─ Backend guarda
   ├─ localStorage: sesión permanece intacta
   └─ Estado: "RESPONDIENDO_ENCUESTA"

8. USUARIO LOGOUT
   ├─ localStorage.removeItem('session')
   ├─ AuthContext.logout()
   ├─ isAuthenticated = false
   ├─ navigate('/landing')
   └─ Estado: "SIN_AUTENTICACION" (volvemos al paso 1)

PERSEVERANCIA DE SESIÓN:
├─ Usuario recarga página:
│  ├─ useEffect en AuthContext lee localStorage
│  ├─ Encuentra session ✓
│  ├─ Se reproduce automáticamente
│  └─ Vuelve a /survey (si estaba allí)
│
└─ Usuario cierra navegador y abre después:
   ├─ localhost carga
   ├─ useEffect lee localStorage
   ├─ Encuentra session ✓
   ├─ Se reproduce automáticamente
   └─ Vuelve a /survey (sin necesidad de registrar de nuevo)
```

---

## 4️⃣ TABLA DE RUTAS Y ACCESO

```
╔════════════════════════════════════════════════════════════════╗
║                    MATRIZ DE ACCESO A RUTAS                   ║
╚════════════════════════════════════════════════════════════════╝

RUTA            │ SIN REG     │ CON REG      │ PROTECCIÓN │ BEHAVIOR
────────────────┼─────────────┼─────────────┼────────────┼──────────────────
/               │  /landing   │  auto→/surv │  Redirect  │ Redirige a /landing
                │             │             │            │
/landing        │  ✅ VER     │  ❌ AUTO    │    NO      │ Si ya registrado
                │  Landing   │  →/survey   │            │ redirige a /survey
                │             │             │            │
/register       │  ✅ VER     │  ✅ VER     │    NO      │ Cualquiera la ve
                │  Form      │  Form       │            │ (podría bloquear)
                │             │             │            │
/survey         │  ❌ BLOQUED │  ✅ VER     │    SÍ      │ ProtectedRoute
                │  →/register │  Encuesta  │            │ valida sesión
                │             │             │            │
/results        │  ❌ BLOQUED │  ✅ VER     │    SÍ      │ ProtectedRoute
                │  →/register │  Resultado │            │ valida sesión
                │             │             │            │
/logout         │  N/A        │  ✅ EXEC    │    NO      │ Limpia session +
                │             │  Logout    │            │ → /landing
```

---

## 5️⃣ VALIDACIÓN DE SESIÓN (Paso a Paso)

```
┌─ USUARIO INTENTA ACCEDER A /survey
│
├─ PASO 1: ProtectedRoute montado
│  └─ const { isAuthenticated } = useContext(AuthContext)
│
├─ PASO 2: Evaluar condición
│  ├─ IF isAuthenticated === false:
│  │  └─ <Navigate to="/register" />
│  │     (Redirige a formulario)
│  │
│  └─ IF isAuthenticated === true:
│     └─ RETORNA children (Survey.jsx)
│        (Renderiza encuesta)
│
├─ PASO 3: ¿De dónde viene el valor de isAuthenticated?
│  ├─ AuthContext.jsx useEffect inicial:
│  │  ├─ const saved = localStorage.getItem('session')
│  │  ├─ IF saved:
│  │  │  ├─ const parsed = JSON.parse(saved)
│  │  │  ├─ setSession(parsed)
│  │  │  └─ setIsAuthenticated(true)  ← AQUÍ
│  │  │
│  │  └─ ELSE:
│  │     └─ setIsAuthenticated(false)
│  │
│  └─ O también de AuthContext.login() cuando se registra:
│     ├─ localStorage.setItem('session', JSON.stringify(data))
│     ├─ setSession(data)
│     └─ setIsAuthenticated(true)  ← AQUÍ
│
├─ PASO 4: Si localStorage está vacío
│  ├─ localStorage.getItem('session') → null
│  ├─ IF null:
│  │  └─ setIsAuthenticated(false)
│  ├─ ProtectedRoute ve: isAuthenticated = false
│  └─ Redirige a /register
│
├─ PASO 5: Si localStorage tiene sesión
│  ├─ localStorage.getItem('session') → "{...}"
│  ├─ parsed = JSON.parse(...)
│  ├─ setIsAuthenticated(true)
│  ├─ ProtectedRoute ve: isAuthenticated = true
│  └─ Renderiza Survey.jsx
│
└─ PASO 6: Acceso a datos
   └─ En Survey.jsx:
      ├─ const { session } = useContext(AuthContext)
      ├─ session.companyId
      ├─ session.userId
      └─ Usar datos para API calls
```

---

## 6️⃣ FLUJO DE DATOS (Data Flow)

```
┌──────────────────────────────────────────────────────────────┐
│              CÓMO FLUYEN LOS DATOS EN LA APP                 │
└──────────────────────────────────────────────────────────────┘

1. REGISTRO
   ┌─ User input
   │  ├─ Register.jsx formData = { nombreEmpresa, email, ... }
   │  └─ handleChange → setFormData({ ...formData, [name]: value })
   │
   ├─ Validación local
   │  ├─ validateForm(formData) → { isValid, errors }
   │  └─ IF errors → mostrar en UI
   │
   ├─ Envío a backend
   │  ├─ POST /api/companies/register
   │  └─ Body: { company: {...}, user: {...} }
   │
   ├─ Respuesta
   │  ├─ Response: { company: { id, name, ... }, user: { id, ... } }
   │  └─ Frontend recibe ✓
   │
   └─ Guardar sesión
      ├─ sessionData = { companyId, userId, ... }
      ├─ localStorage.setItem('session', JSON.stringify(sessionData))
      ├─ AuthContext.login(sessionData)
      ├─ setSession(sessionData)
      └─ setIsAuthenticated(true)

2. ENCUESTA
   ┌─ Inicialización
   │  ├─ Survey.jsx monta
   │  ├─ const { session } = useContext(AuthContext)
   │  └─ session = { companyId: "xxx", userId: "yyy", ... }
   │
   ├─ Mostrar preguntas
   │  ├─ surveyService.getSurveyQuestions()
   │  ├─ GET /api/survey/questions
   │  └─ Response: { questions: [...] }
   │
   ├─ Usuario responde
   │  ├─ handleAnswer(questionId, answerValue)
   │  └─ POST /api/survey/responses
   │
   └─ Guardar respuesta
      ├─ Body: {
      │    company_id: session.companyId,
      │    user_id: session.userId,
      │    question_id: questionId,
      │    answer: answerValue
      │  }
      └─ Backend guarda en BD

3. PERSISTENCIA
   ┌─ Recargar página (F5)
   │  ├─ App monta
   │  ├─ SessionProvider monta
   │  ├─ useEffect: localStorage.getItem('session')
   │  ├─ setSession(parsed)
   │  ├─ setIsAuthenticated(true)
   │  └─ Usuario sigue autenticado ✓
   │
   └─ Cerrar navegador y reabre
      ├─ localStorage persiste en disco
      ├─ Usuario abre app
      ├─ Same as "Recargar página"
      └─ Sesión recuperada ✓
```

---

## 7️⃣ ANTI-PATRONES A EVITAR

```
❌ MALO - NO HACER:
└─ Guardar usuario en global variable
   ├─ Se pierde al recargar
   └─ No persiste

❌ MALO - NO HACER:
└─ Validar only en frontend
   ├─ Usuario podría falsificar datos
   └─ Backend no valida company_id

❌ MALO - NO HACER:
└─ Dejar /register sin protección
   ├─ Usuario podría registrarse múltiples veces
   └─ Datos duplicados

❌ MALO - NO HACER:
└─ Enviar password en localStorage
   ├─ Seguridad comprometida
   └─ Vulnerable a XSS

❌ MALO - NO HACER:
└─ No limpiar localStorage en logout
   ├─ Sesión sigue visible
   └─ Otro usuario accederá

✅ BUENO - SÍ HACER:
└─ Usar Context + localStorage
   ├─ Persiste en navegador
   └─ Accesible en toda la app

✅ BUENO - SÍ HACER:
└─ Validar en frontend Y backend
   ├─ UX mejor
   └─ Seguridad mejorada

✅ BUENO - SÍ HACER:
└─ Usar ProtectedRoute para rutas sensibles
   ├─ Control centralizado
   └─ Fácil mantener

✅ BUENO - SÍ HACER:
└─ Implementar refresh de sesión (opcional)
   ├─ Actualizar token
   └─ Mantener seguridad
```

---

## 8️⃣ TIEMPO DE EJECUCIÓN (Performance)

```
Acción                    │ Tiempo esperado │ Cuello de botella
──────────────────────────┼─────────────────┼──────────────────
Cargar /landing           │ 50-100ms        │ Render React
Llenar formulario         │ 0-50ms          │ JS local
Validar form              │ 5-20ms          │ Regexes
POST /register            │ 500ms-3s        │ Network + BD
Crear empresa en BD       │ 100-200ms       │ Query SQL
Redirigir a /survey       │ 50-100ms        │ React route
Cargar preguntas survey   │ 500ms-1s        │ Network + BD
Responder pregunta        │ 20-50ms         │ JS local
POST respuesta            │ 500ms-1s        │ Network + BD
Recargar (F5)             │ 100-300ms       │ localStorage read
```

---

**Esta guía visual te ayuda a entender cada parte del flujo. Úsala como referencia mientras codeas. 🎯**

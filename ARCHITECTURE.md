# Architecture Document — [FREEADER]

## Stack
- Frontend: Next.js 14 (App Router)
- Backend: Supabase (PostgreSQL + Auth + Storage)
- Deployment: Vercel
- Async jobs: Inngest
- Embeddings: OpenAI text-embedding-3-small
- Vector search: pgvector (extensión de PostgreSQL vía Supabase)

## Principios de arquitectura
- Sin métricas públicas en ninguna capa del sistema
- Distribución por afinidad semántica, nunca por popularidad
- Las señales de comportamiento de lectura son anónimas y agregadas
- El usuario controla su perfil de intereses explícitamente

## Estructura de carpetas
/app → rutas y páginas (Next.js App Router)
/components → componentes React reutilizables
/lib → lógica de negocio, clientes de Supabase y OpenAI
/inngest → funciones de procesamiento asíncrono
/types → tipos TypeScript compartidos
/docs → documentación adicional

## Variables de entorno necesarias
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY
OPENAI_API_KEY
INNGEST_EVENT_KEY
INNGEST_SIGNING_KEY

## Estado del proyecto
- [x] Scaffolding inicial
- [x] Autenticación (registro, login, logout)
- [x] Onboarding de intereses
- [x] Dashboard base
- [ ] Publicación de contenido
- [ ] Motor de embeddings
- [ ] Feed semántico
- [ ] Sesión de descubrimiento (usuario free)
- [ ] Suscripción

## Decisiones técnicas relevantes
- middleware.ts simplificado temporalmente: la protección de rutas 
  la maneja cada página individualmente hasta resolver conflicto 
  con Supabase SSR. Reconstruir antes del deploy a producción.
- window.location.href en lugar de router.push para navegación 
  post-login, necesario para sincronizar sesión con el servidor.

## Sistema de Pen Pals — Reglas de negocio

### Vocabulario
- Carta: mensaje individual, mínimo 200 palabras
- Postal: crédito de conexión. Usuario free: 3 postales. Usuario suscrito: 8 postales

### Flujo de conexión
1. El sistema presenta un pool de sugerencias basado en similitud semántica
   de embeddings de perfil. No se puede contactar perfiles fuera del pool.
2. El usuario remitente elige un perfil y redacta una Carta (mín. 200 palabras)
3. La Carta llega al receptor en estado "cerrada". El receptor ve el perfil 
   del remitente y una vista previa de las primeras 50 palabras. 
4. El receptor decide abrir o ignorar la Carta:
   - Si la abre: se descuenta 1 postal a cada usuario. La conversación inicia.
   - Si la ignora: no hay penalización para ninguno. La postal del remitente 
     se libera y se añade un nuevo perfil al pool de sugerencias.
5. Una vez abierta, el receptor tiene 7 días para responder.
   - Si responde: la conversación continúa con las reglas de cadencia.
   - Si no responde en 7 días: conversación cerrada. El remitente recupera 
     su postal. El receptor no recupera la suya (penalización blanda).
6. Al cerrarse la conversación, el remitente conserva su Carta. 
   El receptor pierde acceso a ella.
7. Los mismos dos usuarios pueden iniciar una nueva conversación 
   en cualquier momento sin restricciones.

### Reglas de cadencia (conversación activa)
- 1 carta por usuario cada 24-48 horas (no hay dobles mensajes)
- Longitud mínima: 200 palabras por carta
- Sin confirmación de lectura
- Sin indicador de escritura
- Sin registro de última actividad
- Máximo de conversaciones simultáneas: 3 (free) / 8 (suscrito)

### Reposición de postales
- Una postal reembolsada vuelve al balance disponible del usuario
- Cuando se libera una postal por rechazo o timeout, el sistema 
  agrega automáticamente un nuevo perfil al pool de sugerencias

### Dependencia técnica
- El sistema de sugerencias requiere embeddings de perfil enriquecidos
- Implementar en Fase 2, después del motor de embeddings y feed semántico

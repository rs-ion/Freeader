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
- [ ] Scaffolding inicial
- [ ] Autenticación
- [ ] Perfil de intereses
- [ ] Publicación de contenido
- [ ] Motor de embeddings
- [ ] Feed semántico
- [ ] Sesión de descubrimiento (usuario free)
- [ ] Suscripción

## Decisiones técnicas relevantes
[Se irá completando a medida que avance el desarrollo]

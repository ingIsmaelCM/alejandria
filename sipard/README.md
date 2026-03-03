# SISTEMA INTEGRADO DE PASANTÍAS (SIPARD) 

## PRÓPOSITO 
El Sistema Integrado de Pasantías (SIPARD) nace con el propósito de transformar y digitalizar el ecosistema de prácticas profesionales en el país. Su objetivo primordial es centralizar, estandarizar y supervisar la relación entre las instituciones académicas, el sector empresarial y los estudiantes en formación, garantizando un proceso transparente, eficiente y orientado a la inserción laboral efectiva. 

### Objetivos Estratégicos 
Para cumplir con este propósito, la plataforma se fundamenta en cuatro pilares: 

1. Estandarización Académica: Proveer a las instituciones de educación superior un marco tecnológico que permita validar la elegibilidad de los estudiantes y homologar los procesos de convenio mediante flujos de trabajo digitales. 
2. Transparencia en la Supervisión: Facilitar el seguimiento continuo del desempeño del pasante a través de bitácoras digitales y evaluaciones 360 grados, eliminando la opacidad en el cumplimiento de horas y objetivos de aprendizaje. 
3. Impulso al Empleo Joven: Actuar como un motor de reclutamiento especializado que identifique el talento temprano y reduzca la brecha entre la culminación de la pasantía y la contratación definitiva, mediante el uso de métricas de éxito y visibilidad corporativa. 
4. Seguridad y Cumplimiento: Garantizar un entorno profesional mediante el uso de tecnología para la moderación de contenido y la gestión administrativa rigurosa de convenios legales. 

## AUDIENCIA 
La plataforma SIPARD está diseñada como un entorno multitenant que atiende a cuatro segmentos de usuarios principales: 

### Estudiantes y Candidatos 
Individuos cursando programas de educación superior o técnica en la República Dominicana que buscan su primera experiencia laboral formal. 
* Uso principal: Gestión de perfil profesional, postulación a vacantes, registro de bitácoras de actividades y consulta de evaluaciones de desempeño. 

### Instituciones Académicas (Universidades e Institutos) 
Coordinadores de pasantías y directores de carrera encargados de supervisar el cumplimiento de los requisitos de graduación de sus alumnos. 
* Uso principal: Validación de elegibilidad estudiantil, monitoreo en tiempo real del progreso del pasante, gestión de convenios interinstitucionales y emisión de certificaciones digitales. 

### Sector Empresarial (Empresas y Supervisores) 
Departamentos de Gestión Humana y supervisores técnicos que buscan captar talento joven y dinamizar sus equipos de trabajo. 
* Uso principal: Publicación de plazas, filtrado de candidatos, gestión de entrevistas, evaluación de pasantes y administración de planes de suscripción. 

### Entidades Gubernamentales 
Instituciones reguladoras de la República Dominicana, como el Ministerio de Trabajo, el MESCYT y el MICM, quienes acceden a la plataforma con un rol de consulta y fiscalización institucional. 
* Uso principal: Monitoreo del cumplimiento de la normativa laboral juvenil, validación de la calidad de los programas de pasantías académicos y extracción de indicadores macroeconómicos. 

### Administradores del Sistema (Super Admins) 
Personal técnico y operativo de SIPARD responsable de la integridad y mantenimiento de la red. 
* Uso principal: Auditoría de acciones (Audit Logs), verificación de empresas, configuración de parámetros globales, gestión de catálogos nacionales y soporte técnico. 

## VALOR AGREGADO DEL SIPARD 
El valor fundamental de SIPARD reside en su capacidad para transformar un proceso tradicionalmente burocrático y fragmentado en un ecosistema digital ágil y transparente. La plataforma elimina las fricciones que retrasan la inserción laboral, garantizando la integridad del proceso mediante inteligencia artificial para la moderación de contenido.

Introduce trazabilidad en el aprendizaje práctico a través de bitácoras digitales, permitiendo a las universidades supervisar en tiempo real las competencias desarrolladas. A nivel empresarial y fiscal, aporta rigor administrativo, incluyendo un modelo de registro interno para facturación (preparado para futura integración con Azul) con soporte para Comprobantes Fiscales (NCF) e ITBIS, y mide el éxito de la contratación post-pasantía con datos reales de empleabilidad. 

## REVISIÓN DE LA ARQUITECTURA DE SOFTWARE 
La arquitectura de SIPARD seguirá un patrón hexagonal: dominio, servicios (adapters & ports), resource controller. Se implementará una arquitectura orientada a microservicios iterativos (ej. ms_auth, ms_catalog).

### Stack Tecnológico

#### Backend y Arquitectura
* Motor de Base de Datos: PostgreSQL.
* Core y Orquestación: NestJS actuando como API Gateway y base de los microservicios.
* ORM: TypeORM para gestión de schemas y consultas.
* Autenticación: PassportJS (JWT, OAuth, Google, etc.).
* Comunicación y Caché: Redis para comunicación entre microservicios y caché.
* Colas y Procesos: BullMQ, implementado bajo principios CLEAN, SOLID y KISS.
* Moderación: NSFWjs (auditor de imágenes con contenido adulto).
* Almacenamiento: Cloudflare R2 para gestión integral de archivos y multimedia.
* Secretos: Infisical.

#### Frontend
* Core: Nuxt (Server-Side Rendering con Vue).
* UI/UX: PrimeVue.
* Estilos: TailwindCSS 4.
* Estado Global: Pinia.

#### Infraestructura, Despliegue y CI/CD
* Alojamiento: Servidor VPS propio.
* Contenedores: Docker.
* Registros Propios: DockerHub privado y NPM registry privado mediante Verdaccio alojados en el VPS.
* CI/CD: Jenkins, GitHub Actions y Husky.
* Observabilidad: Sistema de logs propio (fase inicial).

## CONVENCIONES DE CÓDIGO Y CARPETAS 

### Backend (Patrón Hexagonal y Clean Architecture)
La organización de las carpetas y el código debe seguir estrictamente este esquema, asegurando la separación de las herramientas externas (como BullMQ) de la lógica de negocio:

    ModuleName 
    ├── Domain 
    │   ├── Models (definiciones crudas) 
    │   └── Ports 
    ├── Application 
    │   ├── DTOs 
    │   ├── UseCases / Services (Lógica de negocio pura) 
    │   └── Mappers (domain <> output) (opcional) 
    ├── Infraestructure 
    │   ├── Adapters 
    │   │   ├── Controllers 
    │   │   └── Queue 
    │   │       ├── Processors (Clases @Processor que extraen payload y llaman al UseCase) 
    │   │       ├── Producers (Inyectan el cliente BullMQ y encolan) 
    │   │       └── Workers (Configuración de hilos) 
    │   ├── Mappers (domain <> persistence) 
    │   └── Entities (DB Entities) 
    └── name.module.ts 

    Commons 
    ├── Helpers 
    ├── Utils 
    ├── Config 
    ├── Guards 
    └── Middlewares / Interceptors 

### Frontend
La organización de las carpetas y el código debe seguir el ecosistema de Nuxt:

    Composables 
    Components 
    Pages 
    Plugins 
    Assets 
    Middleware 
    Layouts 
    Public 
    Server 
    Api 

## TIPOS Y PAQUETES COMPARTIDOS 
Las interfaces y contratos deben ser gestionadas como paquetes compartidos distribuidos a través de Verdaccio alojado en el VPS. 

Si el servicio a utilizar espera como payload el mismo tipo de DTO que se va a procesar de forma interna, se crea una entidad compartida. Las interfaces y DTOs se publicarán para mantener sincronía consistente entre el frontend y el backend.

Se debe respetar el principio de responsabilidad única en la creación de paquetes. Ejemplos de distribución:

* @sipard/auth: Interfaces del sistema de autenticación (JWT, OAuth, SMS, OTP).
* @sipard/contracts: Interfaces y tipos comunes entre backend y frontend (DTOs generales).
* @sipard/helpers: Funciones auxiliares que se rigen por el mismo criterio tanto en frontend como en backend.
* @sipard/config: Constantes del ecosistema que no varían por contexto de ejecución (enums, reglas, parámetros globales).

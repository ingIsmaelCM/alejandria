# CASOS DE USO MVP - SIPARD

Esta es la idea inicial que tengo sobre los casos de usos del MVP, para que sea funcional y atractivo, tanto para usuarios, como para posibles inversores.

---

## MS_AUTH
**Entidades:** `User`, `Role`, `Permission`, `UserRole`, `RolePermission`, `UserPermission`.

#### `AUTENTICACIÓN`
* [ ] **[CU-01-01] Registrar usuario**
    * [ ] Validar unicidad de email.
    * [ ] Validar unicidad de username.
    * [ ] Hashear contraseña.
    * [ ] Crear registro en `User`.
    * [ ] Asignar rol por defecto.
    * [ ] Emitir evento `USER_REGISTERED`.

* [ ] **[CU-01-02] Verificar usuario**
    * [ ] Recibir token de verificación.
    * [ ] Cambiar `isEmailVerified` a `true`.

* [ ] **[CU-01-03] Iniciar Sesión**
    * [ ] Validar credenciales.
    * [ ] Generar JWT.
    * [ ] Actualizar `lastLoginAt`.

* [ ] **[CU-01-04] Cerrar Sesión**
    * [ ] Invalidar token en almacenamiento persistente (`Redis`).

* [ ] **[CU-01-05] Recuperar contraseña**
    * [ ] Generar token temporal de reset.
    * [ ] Enviar correo vía `ms_notifications`.

* [ ] **[CU-01-06] Cambiar contraseña**
    * [ ] Validar token temporal de reset.
    * [ ] Hashear nueva contraseña.
    * [ ] Actualizar contraseña.
    * [ ] Invalidar token temporal de reset.
    * [ ] Limpiar sesiones activas.
    * [ ] Emitir evento `PASSWORD_CHANGED`.

#### `RBAC (CONTROL DE ACCESO BASADO EN ROLES)`
* [ ] **[CU-02-01] Asignar rol a usuario**
    * [ ] Crear registro en `UserRole`.

* [ ] **[CU-02-02] Asignar permisos a rol**
    * [ ] Crear registro en `RolePermission`.

* [ ] **[CU-02-03] Asignar permisos a usuario**
    * [ ] Crear registro en `UserPermission`.

* [ ] **[CU-02-04] Remover permisos a usuario**
    * [ ] Eliminar registro en `UserPermission`.

* [ ] **[CU-02-05] Remover permisos a rol**
    * [ ] Eliminar registro en `RolePermission`.
    
* [ ] **[CU-02-06] Remover rol a usuario**
    * [ ] Eliminar registro en `UserRole`.

---

## MS_CATALOG
**Entidades:** `Province`, `Municipality`, `Sector`, `AcademicArea`, `Career`, `Skill`, `SystemConfig`.

#### `GESTIÓN DE CATÁLOGOS TÉCNICOS`

* [ ] **[CU-03-01] Crear catálogo (factory seed)**
    * [ ] Insertar provincias (migration)
    * [ ] Insertar municipios (migration)
    * [ ] Insertar áreas académicas (migration)
    * [ ] Insertar carreras (migration)
    * [ ] Insertar habilidades (migration)
    * [ ] Insertar sectores empresariales (migration)

* [ ] **[CU-03-02] Listar Provincias**
    * [ ] Consultar catálogo de provincias nacionales.
    
* [ ] **[CU-03-03] Listar Municipios**
    * [ ] Consultar municipios pertenecientes a una provincia (`provinceId`).
    
* [ ] **[CU-03-04] Listar Áreas Académicas**
    * [ ] Consultar facultades o áreas de estudio globales.
    
* [ ] **[CU-03-05] Listar Carreras Académicas**
    * [ ] Consultar carreras pertenecientes a un área académica (`academicAreaId`).
    
* [ ] **[CU-03-06] Listar Sectores Empresariales**
    * [ ] Consultar categorías de la industria para perfiles corporativos.
    
* [ ] **[CU-03-07] Listar Habilidades (Skills)**
    * [ ] Consultar catálogo de competencias técnicas y blandas.

---

## MS_PROFILES
**Entidades:** `Student`, `Institution`, `Company`, `CompanyBranding`, `StudentAcademicRecord`.

#### `PERFILES DE ACTORES`
* [ ] **[CU-04-01] Configurar Datos Personales de Estudiante**
    * [ ] Actualizar información de contacto y ubicación (`Municipality`).
    
* [ ] **[CU-04-02] Configurar Récord Académico de Estudiante**
    * [ ] Vincular `Institution`, `Career` y actualizar datos académicos.
    
* [ ] **[CU-04-03] Validar Elegibilidad Estudiantil**
    * [ ] El coordinador marca `isEligibleForInternship` tras validación interna.
    
* [ ] **[CU-04-04] Registrar Perfil Corporativo**
    * [ ] Ingresar RNC, nombre legal y sector de la empresa.
    
* [ ] **[CU-04-05] Gestionar Branding Corporativo**
    * [ ] Actualizar `CompanyBranding` (Logo, Portada y Descripción).

---

## MS_CORE
**Entidades:** `InternshipOffer`, `Application`, `Internship`, `LogbookEntry`, `Evaluation`, `InternshipCertificate`.

#### `PLAZAS Y POSTULACIONES`
* [ ] **[CU-05-01] Publicar Oferta de Pasantía**
    * [ ] Validar verificación de empresa.
    * [ ] Crear vacante vinculando requisitos de carrera y habilidades.
    
* [ ] **[CU-05-02] Postularse a Oferta**
    * [ ] Validar elegibilidad automática del estudiante.
    * [ ] Registrar `Application` en estado inicial.
    
* [ ] **[CU-05-03] Actualizar Estatus de Aplicación**
    * [ ] La empresa modifica el estado (Entrevista, Aceptado, Rechazado).

#### `EJECUCIÓN DE PASANTÍA`
* [ ] **[CU-05-04] Iniciar Pasantía Formal**
    * [ ] Crear `Internship` tras aceptación de oferta.
    
* [ ] **[CU-05-05] Registrar Entrada de Bitácora**
    * [ ] El estudiante reporta descripción de tareas y horas trabajadas.
    
* [ ] **[CU-05-06] Adjuntar Evidencias a Bitácora**
    * [ ] Subir archivos a almacenamiento Cloudflare R2 y vincular a la entrada.
    
* [ ] **[CU-05-07] Revisar Bitácora (Supervisor)**
    * [ ] Aprobar o Rechazar entrada con feedback.
    
* [ ] **[CU-05-08] Sincronizar Horas Totales**
    * [ ] Al aprobar bitácora, actualizar acumulado de horas en la pasantía.
    
* [ ] **[CU-05-09] Emitir Certificación Digital**
    * [ ] Generar `InternshipCertificate` al cumplir la meta de horas.

---

## MS_NOTIFICATIONS
**Entidades:** `NotificationTemplate`, `InAppNotification`.

#### `SISTEMA DE ALERTAS`
* [ ] **[CU-06-01] Enviar Notificación en Tiempo Real (SSE)**
    * [ ] Notificar al usuario mediante socket o evento de servidor.
    
* [ ] **[CU-06-02] Despachar Correo Electrónico**
    * [ ] Generar email basado en `NotificationTemplate`.
    * [ ] Insertar el correo en la cola de envío (queue).
    * [ ] Despachar el correo de la cola mediante un worker/processor.
    * [ ] Registrar el envío en la tabla `InAppNotification`.
    
* [ ] **[CU-06-03] Consultar Historial de Notificaciones**
    * [ ] Listar alertas recibidas en la bandeja de entrada del sistema.

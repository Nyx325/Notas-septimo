## FN1. Inicio de sesión.

### Original

**Descripción:** El sistema web tendrá tres tipos de
inicio de sesión para cada uno de los usuarios:
administrativo, profesional y cliente.

**Importancia:** Esencial.

**Validez:**

● Medible: Cuando un usuario registrado ingrese su
nombre de usuario y contraseña en el sistema web, el
sistema deberá redireccionarlo a la página
correspondiente según su rol, ya sea administrativo,
profesional o cliente.

● Alcanzable: Se desarrollará un formulario de inicio
de sesión para los diferentes tipos de usuarios. Al
ingresar sus credenciales (nombre de usuario y
contraseña) dentro del formulario, éstas serán
validadas con los registros almacenados en la base de
datos.

● Relevante: Es de suma importancia contar con
diversos métodos para el ingreso dentro del sitio web,
asegurar la autenticación y autorización pertinentes
de cada usuario, garantizando el acceso únicamente a
las funciones y datos relacionados a su rol.

### Cambios

#### Medible

```text
El usuario al querer acceder al sistema se le solicitarán credenciales como usuario y contraseña para poder determinar si es una persona autorizada y qué permisos tiene sobre el sistema (administrativo, profesional o cliente).
```

**Razón del cambio:** Quitar énfasis en el dashboard
porque si bien es parte del flujo de la página, describir
cómo se ve el dashboard o qué tendrá ya no depende del
requisito de inicio de sesión, podria llegarse a cambiar
por mostrar otro apartado que no sea el dashboard por
cualquier cosa y entonces ya no se cumpliria el requisito

#### Alcanzable

**Opcional:** Lo que ya se tiene, pero indicar además
que la busqueda a la BD también determinará el tipo
de usuario que tiene el usuario

## FN2. Gestión de Profesionales.

**Descripción:** El usuario Profesional podrá
registrar su información en el sistema tal como:
Oficios que realiza (se le presentará el conjunto de
oficios que permita el sistema), datos personales y
evidencias que acrediten su expertise en el o los
oficios que puede realizar (se le facilitará un
espacio para suba un archivo con sus evidencias),
credenciales de acceso.

El Administrativo validará a los profesionales que
solicitan el registro y autorizará el acceso al
sistema.

Una vez que un Profesional fue autorizado, éste
podrá visualizar su perfil y editarlo. La eliminación
de un profesional estará a cargo del Administrativo.

**Importancia:** Esencial.

**Validez:**

● Medible: Se podrá verificar que un profesional
registrado pueda acceder a su perfil y editarlo después
de ser autorizado por el Administrativo.

● Alcanzable: Se implementará un formulario que permitirá
a los profesionales ingresar sus datos y evidencias,
validando su información en la base de datos.

● Relevante: La gestión adecuada de los registros de
profesionales es crucial para asegurar la calidad de los
servicios ofrecidos y la confianza en el sistema.

### Correcciones

#### Descripción

```text
Un usuario cliente puede convertirse en profesional siempre que brinde la información relacionada para una posterior revisión por un administrador
```

#### Medible

```text
Un cliente podrá convertirse en profesional tras brindar una serie de datos como: el oficio que realiza y facilitando una serie de identificaciones, (todo lo demás específico que pide) para una posterior validación por un administrador dentro del sistema, si la información es correcta, el administrador podrá autorizar el alta de este usuario como un profesional
```

**Razón del cambio:** Considero que se hace más énfasis en
cómo un administrador puede alterar el perfil del profesional
y más bien la descripción del funcionamiento del requisito
se vacía en la descripción, además ¿Por qué un administrador
debería editar el perfil de un profesional? Ta raro que un
admin edite la info personal de otro jajaj pero sí que pueda
dar de alta o baja a los profesionales, pero creo eso deberia
ser en un requisito aparte

#### Alcanzable

Cambiar `validando su información en la base de datos.` por:

```text
Para permitir mostrar la información a validar a un usuario administrador
```

## FN3. Gestión de clientes.

Descripción: Ambos usuarios Clientes y Administrativo realizarán la consulta. La edición solo la realizará el cliente.
Los usuarios Administrativos y clientes pueden eliminar el registro.
El usuario cliente podrá registrar su información tal como: nombre, apellido paterno, apellido materno, dirección, teléfono y correo electrónico.
Importancia: Esencial.
Validez:
● Medible: Se verificará que un cliente pueda registrar, editar y eliminar su información de forma efectiva en el sistema.
● Alcanzable: Se desarrollará un formulario para que los clientes ingresen su información, validando la entrada de datos.

● Relevante: Facilitar la gestión de la información del cliente es fundamental para ofrecer un servicio personalizado y eficiente.

# Aplicación web y API sin servidor

Creará una aplicación web sin servidor alojando contenido de sitio web estático en GitHub Pages e implementando el programa de fondo de la aplicación mediante IBM Cloud™ Functions.

En lugar de suministrar una máquina virtual, un contenedor o un entorno de ejecución de Cloud Foundry para desplegar el programa de fondo, puede implementar la API de programa de fondo con una plataforma sin servidor. Esta puede ser una buena solución para evitar pagar por el tiempo de desocupación y dejar la plataforma se escale cuando sea necesario.


## IBM Developer Advocates Team

Módulo: Serverless	
Agenda
* [Prework](#Prework)
* [Crear y configurar Cloudant DB](#Crear-y-configurar-Cloudant-DB)
* [Configuración de Functions](#Configuración-de-Functions)
* [Configuración de API](#Configurar-el-API)
* [Despliegue de Aplicación](#Despliegue)





## Prework:
* Cuenta de IBM Cloud (https://cloud.ibm.com/registration)
* Instalar cli de IBM Cloud https://cloud.ibm.com/docs/cli/reference/ibmcloud?topic=cloud-cli-install-ibmcloud-cli 
* Cuenta en github https://github.com/join
* Utilizar safari, chrome, firefox, edge

### Cupones para Estudiantes y profesores
 

	1.	Acceder al HUB para Software para uso académico. Y navegar hasta la parte de abajo de la pagina https://onthehub.com/ibm/?utm_sourc=ibm-ai-productpage&utm_medium=onthehubproductpage&utm_campaign=IBM 
	2.	Buscar el WebStore del instituto/escuela al que perteneces. 
	3.	En caso de no contar con WebStore, acceder al portal de IBM Academic Initiative y seleccionar la opción de <Students> 
	4.	Seleccionar <Add to Cart> para IBM Bluemix – 6 Month Trial. 
	5.	Realizar el registro correspondiente utilizando la cuenta de correo académica 
 
### Cargar créditos en IBM Cloud
	1.	Ingresamos a nuestro panel de control de IBM Cloud (console.bluemix.net)
	2.	Una vez que tengamos el código nos vamos a Gestionar>Facturación y Uso>Facturación
	3.	Buscamos “Códigos de características (Promocionales)”


## Crear y configurar Cloudant DB:
	1.De nuestro catálogo en console.bluemix.net seleccionamos Cloudant
	2.Buscamos Cloudant
 
	3.Lo nombramos guestbook-db, seleccionamos Legacy Credentials y IAM, posteriormente creamos una instancia del servicio
	4.Una vez que lo creamos dentro de nuestro servicio, le damos click en “Launch cloudant dashboard”
	5.Nos vamos a la 3er tab de lado izquierdo, damos click en Create Database y la nombramos guestbook
 
	6.Regresamos al servicio y en el tab de “Service Credentials” generamos una nueva credencial
 

## Configuración de Functions
En esta sección configuraremos nuestro servicio de Functions.
1. Secuencia de acciones para escribir a la base de datos
	1. Vamos al catálogo y buscamos Cloud Functions
 	2. Una vez dentro seleccionamos Actions
	3. Damos click en Create
	5. Damos click en Create action
	6. Ponemos el nombre prepare-entry-for-save y seleccionamos Node.js 6 como el Runtime, damos click en Create
	7. Cambiamos el código por el siguiente:
		``` js
		function main(params) {
		 if (!params.nombre || !params.comentario) {
		  return Promise.reject({ error: 'no name or comment'});
		  }
		 return {
		  doc: {
		   createdAt: new Date(),
		   name: params.nombre,
		   email: params.correo,
		   comment: params.comentario
		  }
		 };
	 	}
		```
	8. Lo salvamos
	9. Para añadir nuestra acción a una secuencia primero nos vamos al tab “Enclosing Secuences” y damos click en “Add to Sequence”
 	10.	Para el nombre de la secuencia ponemos save-guestbook-entry-sequence y posteriormente damos click en Create and Add
	11.	Una vez que esta creada nuestra secuencia le damos click y damos click en Add posteriormente
 	12.	Damos click en Use Public y seleccionamos Cloudant
 	13.	Seleccionamos la acción create-document, damos click en New Binding, ponemos de nombre de nuestro paquete binding-for-guestbook y en Cloudant Instance seleccionamos Input Your Own Credentials
 	14.	 Para llenar todos los datos posteriores copiamos lo que teníamos en el servicio de Cloudant como credenciales y damos click en Add:
 	15.	Para probar que esté funcionando, damos click en change input e ingresamos nuestro siguiente JSON y damos click en Apply y luego en Invoke
	 ```json
		{
		"nombre": "John Smith",
		"correo": "john@smith.com",
		"comentario": "this is my comment"
		}
	```
	Una vez hecho esto podremos verlo escrito en nuestra base de datos de Cloudant en la sección Documents
 
2. Secuencia de acciones para obtener las entradas de la base de datos
Esta secuencia la usaremos para tomar las entradas de cada usuario y sus respectivos comentarios
	1.	En nuestra tab de functions creamos una acción Node.js y le ponemos el nombre set-read-input, siguiendo el mismo proceso que en la acción anterior
	2.	Reemplazamos el código que viene, esta acción pasa los parámetros apropiados a nuestra siguiente acción
		```js
		function main(params) {
		 return {
		  params: {
		   include_docs: true
		   }
		 };
		}
		```
	3. Damos click en Save 
	4. Damos click en Enclosing Sequences, Add to Sequence y Create New con el nombre read-guestbook-entries-sequence damos click en Create and Add
	5. Damos click en Actions y  damos click en read-guestbook-entries-sequence
 	6. Damos click en Add para crear una segunda acción en la secuencia
	7. Seleccionamos Public y Cloudant
 	8.	Seleccionamos list-documents en actions y seleccionamos el binding binding-for-guestbook y posteriormente damos click en Add
 	9.	Damos click en Add para añadir una acción más a la secuencia, esta es la que va a dar el formato de los documentos cuando regresen de Cloudant
	10.	La nombraremos format-entries y posteriormente damos click en Create and add 
	11.	Damos click en format-entries y reemplazamos el código con:
		```JS
		const md5 = require('spark-md5');
			
		function main(params) {
		 return {
		  entries: params.rows.map((row) => { return {
		   name: row.doc.nombre,
		   email: row.doc.correo,
		   comment: row.doc.comentario,
		   createdAt: row.doc.createdAt,
		   icon: (row.doc.correo ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
		  }})
		 };
		}
		```
	12.	Salvamos y damos click en invoke
 
## Configurar el API
1.	Dentro de nuestras acciones seleccionamos nuestras secuencias y en la tab de Endpoints damos click en Enable Web Action y damos click en Save
 
2.	Nos vamos a Functions y damos click en APIs
 
3.	Damos click en Create Managed API
4.	En el API name ponemos guestbook y en el path ponemos /guestbook y damos click en create operation
 
5.	Creamos un path que sea /entries ponemos el verbo a GET y seleccionamos la secuencia read-guestbook-entries-sequence y damos click en Create
 
6.	Realizamos la misma acción pero ahora con un POST y la secuencia save-guestbook-entries-sequence y damos click en Create
7.	Salvamos y exponemos la API
 
## Despliegue
1.	Clonamos el siguiente repositorio en alguna carpeta:
https://github.com/IBM-Cloud/serverless-guestbook
2.	Modificamos el docs/guestbook.js y reemplazamos el valor de apiUrl con la ruta dada por API Gateway, que obtenemos al dar click en APIs
 
3.	Hacemos push de esto a un nuevo repositorio
4.	En el área de Settings>Github Pages, seleccionamos master branch /docs folder
 
5.	Podemos entrar a nuestra página en el link que aparece

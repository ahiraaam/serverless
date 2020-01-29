# Aplicación web y API sin servidor
[![IBM Cloud powered][img-ibmcloud-powered]][url-ibmcloud]

Creará una aplicación web sin servidor alojando contenido de sitio web estático en GitHub Pages e implementando el programa de fondo de la aplicación mediante IBM Cloud™ Functions.

En lugar de suministrar una máquina virtual, un contenedor o un entorno de ejecución de Cloud Foundry para desplegar el programa de fondo, puede implementar la API de programa de fondo con una plataforma sin servidor. Esta puede ser una buena solución para evitar pagar por el tiempo de desocupación y dejar la plataforma se escale cuando sea necesario.

# Functions de IBM Cloud
Plataforma de IBM cloud de programacion poliglota FaaS (Functions-as-a-Service) para desarrollo de codigo ligero que escala dependiendo de la demanda.

Para mayor informacion: [![Functions][img-cloud-functions]][url-ibmcloud-Functions]

## IBM Developer Advocates Team

Módulo: Serverless	
Agenda
* [Prework](#Prework)
* [Crear y configurar Cloudant DB](#Crear-y-configurar-Cloudant-DB)
* [Configuración de Functions](#Configuración-de-Functions)
* [Configuración de API](#Configurar-el-API)
* [Despliegue de Aplicación](#Despliegue)





## Prework:
* Cuenta de [IBM Cloud][url-IBMCLOUD]
* Instalar [CLI de IBM Cloud][url-CLI-IBMCLOUD] 
* Cuenta en [GitHub][url-github-join]
* Instalar [CLI de GitHub][url-github-cli] o instalar [GitHub Desktop][url-githubdesktop]
* Utilizar safari, chrome, firefox, edge

### Cupones para Estudiantes y profesores
 

	1.	Acceder al HUB para Software para uso académico. Y navegar hasta la parte de abajo de la pagina https://onthehub.com/ibm/?utm_sourc=ibm-ai-productpage&utm_medium=onthehubproductpage&utm_campaign=IBM 
	2.	Buscar el WebStore del instituto/escuela al que perteneces. 
	3.	En caso de no contar con WebStore, acceder al portal de IBM Academic Initiative y seleccionar la opción de <Students> 
	4.	Seleccionar <Add to Cart> para IBM Bluemix – 6 Month Trial. 
	5.	Realizar el registro correspondiente utilizando la cuenta de correo académica 
 
### Cargar créditos en IBM Cloud
	1.	Ingresamos a nuestro panel de control de IBM Cloud (cloud.ibm.com)
	2.	Una vez que tengamos el código nos vamos a Gestionar>Facturación y Uso>Facturación
	3.	Buscamos “Códigos de características (Promocionales)”


## Crear y configurar Cloudant DB:
1.De nuestro catálogo en cloud.ibm.com buscamos Cloudant.<br>
![](img/imc1.png)
<br>
2.Seleccionamos Cloudant.<br>
![](img/imc2.png)
<br>
3.Lo nombramos guestbook-db, seleccionamos Legacy Credentials y IAM, posteriormente creamos una instancia del servicio.<br>
![](img/imc3.png)
<br>
4.Ya que el servicio este desplegado y listo para usar, en el tab de “Service Credentials” buscarmo si ya tenemos alguna creada, si no es asi, generamos una nueva credencial, que usaremos más adelante.<br>
![](img/imc5.png)
<br>
![](img/imc6.png)<br>
5.Volvemos a la tab de "Manage" y le damos click en “Launch cloudant dashboard”.<br>
![](img/imc4.png)<br>
6.Nos vamos a la tab de lado izquierdo, damos click en "Create Database" y la nombramos "guestbook".<br>
 ![](img/imc7.png)<br>

## Configuración de Functions
En esta sección configuraremos nuestro servicio de Functions.
1. Secuencia de acciones para escribir a la base de datos
	1. Vamos al catálogo y buscamos **Cloud Functions**.<br>
	![](img/im1.png)<br>
 	2. Una vez dentro seleccionamos "Actions".<br>
	![](img/im2.png) <br>
	3. Damos click en "Create".<br>
	![](img/im3.png)<br>
	5. Ponemos el nombre "prepare-entry-for-save" y seleccionamos "Node.js 10" como el Runtime, damos click en "Create".<br>
	![](img/im4.png)<br>
	6. Cambiamos el código por el siguiente:
		``` js
		function main(params) {
		 if (!params.nombre || !params.comentario) {
		  return Promise.reject({ error: 'no nombre or comentario'});
		  }
		 return {
		  doc: {
		   createdAt: new Date(),
		   nombre: params.nombre,
		   correo: params.correo,
		   comentario: params.comentario
		  }
		 };
	 	}
		```
	7. Lo salvamos.<br>
	![](img/im5.png)<br>
	8. Para añadir nuestra acción a una secuencia primero nos vamos al tab “Enclosing Secuences” y damos click en “Add to Sequence”.
	<br>
	![](img/im6.png)<br>
 	9.	Para el nombre de la secuencia ponemos "save-guestbook-entry-sequence" y posteriormente damos click en "Create and Add".<br>
	![](img/im7.png)<br>
	10.	Una vez que esta creada nuestra secuencia le damos click al nombre de la secuencia "save-guestbook-entry-sequence" y posteriormente damos click en "Add".<br>
	![](img/im8.png)<br>
 	11.	Damos click en "Use Public" y seleccionamos "Cloudant".<br>
	![](img/im9.png)<br>
 	12.	Seleccionamos la acción "create-document", damos click en "New Binding", ponemos de nombre de nuestro paquete "binding-for-guestbook" y en "Instance" seleccionamos "Input Your Own Credentials".<br>
	![](img/im10.png)<br>
 	13.	 Nos desplegara una lista. Para llenar estos datos copiamos las credenciales que tenemos en nuestro servicio de "Cloudant" y damos click en "Add", lo llenamos de la siguiente manera:<br>
	![](img/im11.png)<br>
 	14.	Para probar que esté funcionando, damos click en "save" y luego en "change input" e ingresamos nuestro siguiente JSON y damos click en Apply y luego en Invoke
	 ``` json
		{
		"nombre": "Isaac Carrada",
		"correo": "isaac@carrada.com",
		"comentario": "HOLA MUNDO"
		}
	```
	Una vez hecho esto y nos de una id de activacion correcta, podremos verlo escrito en nuestra base de datos de "Cloudant", en el "Dashboard", en la sección "Documents"<br>
	![](img/im12.png)<br>
	![](img/im13.png)<br>
	![](img/im14.png)<br>
2. Secuencia de acciones para obtener las entradas de la base de datos
Esta secuencia la usaremos para tomar las entradas de cada usuario y sus respectivos comentarios, regresemos a "Functions/Actions".
<br>
![](img/im15.png)<br>
	1.	En nuestra tab de functions creamos una nueva acción Node.js y le ponemos el nombre "set-read-input", siguiendo el mismo proceso que en la acción anterior.<br>
	![](img/im16.png)<br>
	![](img/im17.png)<br>
	![](img/im18.png)<br>
	2.	Reemplazamos el código que viene, esta acción pasa los parámetros apropiados a nuestra siguiente acción:
		``` js
		function main(params) {
		 return {
		  params: {
		   include_docs: true
		   }
		 };
		}
		```
	3. Damos click en "Save" y click en "Enclosing Sequences".<br>
	![](img/im19.png)<br>
	4. Damos "Add to Sequence" y "Create New" con el nombre "read-guestbook-entries-sequence", y damos click en "Create and Add".
	<br>
	![](img/im20.png)<br>
	5. Damos click en el nombre de la secuencia "read-guestbook-entries-sequence".
 	6. Damos click en "Add" para crear una segunda acción en la secuencia.
	7. Seleccionamos "Public" y "Cloudant".
 	8.	Seleccionamos "list-documents" en actions y seleccionamos el binding "binding-for-guestbook" y posteriormente damos click en "Add".<br>
	![](img/im21.png)<br>
 	9.	Damos click en "Save" y luego en "Add" para añadir una acción más a la secuencia, esta es la que va a dar el formato de los documentos cuando regresen de Cloudant.
	10.	La nombraremos "format-entries" y posteriormente damos click en "Create and add".<br>
	![](img/im22.png) <br>
	11.	Damos click en "Save" y luego en nuestra accion "format-entries" y reemplazamos el código con:
		``` js
		const md5 = require('spark-md5');
			
		function main(params) {
		 return {
		  entries: params.rows.map((row) => { return {
		   nombre: row.doc.nombre,
		   correo: row.doc.correo,
		   comentario: row.doc.comentario,
		   createdAt: row.doc.createdAt,
		   icon: (row.doc.correo ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.correo.trim().toLowerCase())}?s=64` : null)
		  }})
		 };
		}
		```
	12.	Salvamos y regresamos a nuestra secuencia para correrla con "invoke" (Esto lo podemos hacer dando click en acciones, y luego en nuestra secuencia, o en "Enclosing Sequence" y luego en nuestra secuencia).<br>
	![](img/im23.png)<br>
	![](img/im24.png)<br>
 
## Configurar el API
1.	Dentro de nuestras acciones seleccionamos ambas secuencias y en la tab de "Endpoints" damos click en "Enable Web Action" y damos click en "Save". **Es importante que se haga para cada secuencia**.<br>
![](img/im25.png)<br>
![](img/im26.png)<br>
Y nos quede de la siguiente manera:<br>
![](img/im27.png)<br>
2.	Nos vamos a el tag "APIs" que esta de lado derecho.
3.	Damos click en "Create API".<br>
![](img/im28.png)<br>
4.	En el "API name *" ponemos "guestbook" y en el "Base path for API *" ponemos "/guestbook" y damos click en "create operation".
<br>
![](img/im29.png) <br>
5.	Creamos un "Path *" que sea "/entries" ponemos el verbo a "GET" y seleccionamos la secuencia "read-guestbook-entries-sequence" y damos click en "Create".<br>
![](img/im30.png)<br>
6.	Realizamos la misma acción pero ahora con un "POST" y la secuencia "save-guestbook-entries-sequence" y damos click en "Create".
<br>
![](img/im31.png)<br>
7.	Nos dirigimos hasta abajo y damos click en "Create" para exponer la API.<br>
![](img/im32.png)<br>
 
## Despliegue
1.	Forkeamos y despues clonamos este repositorio en alguna carpeta de facil acceso.<br>
![](img/im33.png)<br>
``` bash
git clone <url de tu nuevo repositorio>
```
2.	Modificamos el "docs/guestbook.js" y reemplazamos el valor de "apiUrl" en la linea 6 con la ruta dada por el "API Gateway", que obtenemos al dar click en nuesta API.<br>
![](img/im34.png)<br>
![](img/im35.png)<br>
3.	Guardamos el archivo modificado y hacemos "push" de esto al repositorio que habiamos clonado.
``` bash
git add .
git commit -m "nuevo commit"
git push
```
4.	En el área de Settings>Github Pages, seleccionamos "master branch/docs folder.<br>
![](img/im36.png)<br>
![](img/im37.png)<br>
5.	Y podemos entrar a nuestra página en el link que aparece ya que nos diga que el sitio esta publicado.<br>
![](img/im38.png)<br>
![](img/im39.png)

[url-ibmcloud]: https://www.ibm.com/cloud/
[img-cloud-functions]: https://img.shields.io/badge/IBM%20cloud-Functions-red.svg
[url-ibmcloud-Functions]: https://www.ibm.com/cloud/functions
[url-IBMCLOUD]: https://cloud.ibm.com/registration
[url-CLI-IBMCLOUD]: https://cloud.ibm.com/docs/cli/reference/ibmcloud?topic=cloud-cli-install-ibmcloud-cli
[url-github-join]: https://github.com/join
[url-github-cli]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
[url-githubdesktop]: https://desktop.github.com/
[img-ibmcloud-powered]: https://img.shields.io/badge/IBM%20cloud-powered-blue.svg
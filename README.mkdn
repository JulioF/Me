Magento-IE
===================


Magento-IE provee un conjunto de librerias(libraries) que **resuelven la conexión a las APIs nativas de Magento2** para facilitar su integración con otros sistemas.
Estas librarías, manejan: la comunicación rest, autenticación, swagger.ui, loggeo de mensajes, mappeo por default dozzer, manejo de errores entre otros. También proveen cada método de la API de magento2 como un método Java, modelos de consultas y respuesta y herramientas que facilitan el trabajo con estos.
 

----------


Componentes
-------------

Magento-IE provee una libreria por cada modulo de magento y una clase cliente por cada servicio de Magento
mas un core que gestiona todas las funcionalidades y modelos comunes.

Cada cliente realiza la comunicación a todos los métodos del servicio de magento2 al que referencia, ésta comunicación se realiza por medio de un RestTemplate provisto por el modulo core.
Luego, el core conecta contra el servidor de magento2 y actualiza el token de ser necesario

Secuencialmente:
```sequence
Title: Client method consume
Note left of Connector: handle models
Connector->Magento ie module: consume a client
Magento ie module->Magento core: magento\nrest template
Magento core->>Magento2 Srv: send rest api\nrequest
Note right of Magento2 Srv: verify authorizacion\nand process
Magento2 Srv->>Connector: rest api\nresponse
Note left of Connector: next integration\nfunction
```


```sequence
Title: Verify authorization
Note left of Magento core: expired token
Magento core->Magento2 Srv: send rest api\nrequest
Magento2 Srv->Magento core: response 401\n(unauthorized)
Magento core->Magento2 Srv: request token
Note right of Magento2 Srv: generate a new token
Magento2 Srv->Magento core: token
Magento core->Magento core: update token\nin rest template
Note left of Magento core: start here\nwhen token is ok
Magento core->Magento2 Srv: send rest api\nrequest
Magento2 Srv->>Magento core: response 200\n(ok)
```

Flujo de conexión con Magento2:

```flow

st=>start: start
e=>end
op1=>operation: Send API 
			request
op2=>subroutine: update token
cond1=>condition: authorized?
cond2=>condition: first time?

st->op1->cond1->cond2->op2
cond1(no)->cond2
cond1(yes)->e
cond2(no,bottom)->e
cond2(yes)->op2
op2(right)->op1
```


----------


Configuración
-------------------

Por cada servidor magento que deseemos integrar se deberán configurar:
<pre>
	magento2:
	  servers:
	    {alias}: ## reemplazar por el alias del servidor
			url: ##host del server
	        user-type: ##tipo de integration token
	        credential: 
		        username: ## usuario para obtención de token
		        password: ## password del usuario
	      	timeout:
		        read:  ## tiempo limite de lectura de una respuesta
		        connect: ## tiempo limite para establecer una conexión
</pre>
si la integración es para con un unico servidor magento 2 puede usar como alias "master" y tendrá disponible todos los clientes de las APIs como Spring-Beans sin necesidad de ninguna configuración adicional.

si se deseara usar mas de un servidor Magento2 debe inicializar el cliente suministrando el alias del servidor
por ejemplo:

		@Autowired
		Map<String, RestConnectionBus> restBuses;
		
		public final void functionality() {
			ProductClient productClientSrv1 = new ProductClientImpl(restBuses.get("servidor_1"));
			ProductClient productClientSrv2 = new ProductClientImpl(restBuses.get("servidor_2"));
			...
		}

IMPORTANTE: Todos los clientes deben ser STATELESS y SINGLETON. Para lo último, puede optar por configurar un bean por cada cliente a usar.

		@Bean
		public ProductClient productClientSrv1() {
			return new ProductClientImpl(restBuses.get("servidor_1"));
		}

----------


Uso
-------------

Para hacer uso de las herramientas que provee Magento-IE 

1. Importar el Core del IntegrationEngine

		<dependency>
			<groupId>com.summa.ie</groupId>
			<artifactId>magento-ie-catalog</artifactId>
			/*<version>{my-version}</version>*/
		</dependency>

2. Importar las librerias de los módulos de Magento que desea integrar. 
Por ejemplo Catalog

		<dependency>
			<groupId>com.summa.ie</groupId>
			<artifactId>magento-ie-catalog</artifactId>
			/*<version>{my-version}</version>*/
		</dependency>
	

3. Configurar en el application.properties o YML los datos del/los servidor/es de magento2
Por ejemplo:
<pre>
	magento2:
	  servers:
	    master:
	      url: http://dev.canaldeautopartes.summasolutions.net/index.php/rest/default
	      user-type: admin
	      credential: 
	        username: admin
	        password: Summa2017
	      timeout:
	        read: 20000
	        connect: 20000
</pre>

----------


> **Note:** 

> - **Verificar** que se encuentre correctamente configurado el archivo settings.xml de maven
> - **Verificar** jdk >1.8






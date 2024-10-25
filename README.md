Tarea: Despliegue un aplicativo END to END en Kubernetes con Istio y Observabilidad.
Nombre: Rosa Graciela Guerrero Idrovo

Docker Hub: rgguerrero/tarea4
Git Hub: 
Esta tarea esta compuesta por 5 pasos:
1.	Configuración inicial del entorno.
2.	Creación y publicación de la imagen Docker.
3.	Despliegue en Kubernetes
4.	Exponer el servicio con Istio
5.	Observabilidad en Grafana, Kiali y Jaeger
Desarrollo:
1.	Configuración Inicial del Entorno
a)	Abrir Visual Studio Code y navega a la carpeta:
Abrimos el terminal en Visual Studio Code y navegamos al directorio donde trabajaremos, en este caso: cd C:\Users\graci\Desktop\docker

b)	Crear un archivo Dockerfile con el siguiente contenido:
En Visual Studio Code, en el explorador de archivos, hacemos clic derecho en la carpeta y seleccionamos "Nuevo archivo". Lo nombramos Dockerfile y agregamos el siguiente texto: 
FROM nginx:latest
RUN echo '<h1>Hola Mundo, este es la tarea 4 de Rosa Graciela Guerrero</h1>' > /usr/share/nginx/html/index.html

3.	Crear la Imagen de Docker y Subirla a Docker Hub

a)	Construimos la Imagen de Docker
En el terminal en Visual Studio Code, escribimos
docker build -t rgguerrero/tarea4 .

b)	Iniciar Sesión en Docker Hub
Iniciamos sesión en Docker Hub para poder subir la imagen:
docker login

c)	Subir la Imagen a Docker Hub
Una vez que hayamos iniciado sesión, subimos la imagen creada:
docker push rgguerrero/tarea4

4.	Crear Archivos de Despliegue en Kubernetes

a)	Crear el archivo de despliegue deployment.yaml:
En Visual Studio Code, creamos un archivo llamado deployment.yaml en el directorio y agregamos las siguientes líneas de código:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tarea4
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tarea4
  template:
    metadata:
      labels:
        app: tarea4
    spec:
      containers:
      - name: tarea4
        image: rgguerrero/tarea4
        ports:
        - containerPort: 80

b)	Aplicar el Despliegue en Kubernetes
Aplicamos el despliegue con kubectl:
kubectl apply -f deployment.yaml
Verificamos el Estado del Despliegue
kubectl get deployments
 
5.	Exponer el Servicio con Istio

a)	Habilitar la Inyección Automática de Istio (si aún no está activa)
Activamos la inyección en el namespace por defecto:
kubectl label namespace default istio-injection=enabled
 
b) Crear el archivo istio-service.yaml para definir el Servicio y Gateway:
En Visual Studio Code, creamos un archivo llamado istio-service.yaml y añadimos:

apiVersion: v1
kind: Service
metadata:
  name: tarea4-service
spec:
  selector:
    app: tarea4
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
---
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: tarea4-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: tarea4
spec:
  hosts:
    - "*"
  gateways:
    - tarea4-gateway
  http:
    - route:
        - destination:
            host: tarea4-service
            port:
              number: 80
c)	Aplicar la Configuración del Servicio e Istio Gateway
Ejecutamos el siguiente comando:
kubectl apply -f istio-service.yaml
 

5.	Habilitar Observabilidad en Istio
Utilizamos los siguientes comandos para abrir los dashboards de observabilidad:

a)	Para monitorear datos en Grafana:
istioctl dashboard grafana

Si se tuviese algún problema con istio, se debe corroborar lo siguiente:
•	Descargarse el instalador istioctl.exe
Verificamos el directorio donde se extrajo istio (e.j., C:\istio\bin).
•	Añadir istioctl to PATH
Ir a Settings > System > About > Advanced System Settings > Environment Variables.
Seleccionamos System Variables, seleccionamos Path y hacemos doble clic para Editar.
Añadimos Nuevo a la carpeta  que contiene istioctl.exe (ej.., C:\istio\bin) hacemos clic en Aceptar para guardar los cambios.
•	Reiniciar el terminal
Si estamos usando powershell debemos abrir y cerrar para ejecutar con los cambios realizados.
•	Verificar la disponibilidad de istioctl 
Ejecutar el commando para verificar que existe istioctl:
	istioctl versión
Una vez que istio es accesible, comprobamos en powershell el comando para abrir el dashboard de grafana
	istioctl dashboard grafana
 
 
b)	Para ver el tráfico de servicio en Kiali
Para ver el tráfico del servicio, podemos usar Kiali, con el siguiente comando en powershell:
istioctl dashboard kiali
Si tuviese algún problema, puede ser porque no se instaló Kiali, siguiendo los siguientes pasos se solventa:
•	Instalar Kiali:
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/kiali.yaml
 
•	Confirmar el despliegue de Kiali Pod 
Después de aplicar la configuración Kiali, compruebamos si el pod Kiali se está ejecutando:
kubectl get pods -n istio-system
 
Buscamos un pod con un nombre similar a kiali-xxxxxxxx. Verificamos de que su estado es Running.
Finalmente, si el pod está ejecutándose podemos desplegar el dashboard
istioctl dashboard kiali
 

c)	Para rastrear solicitudes en Jaeger
Ejecutando el comando: istioctl dashboard Jaeger
Si por algún motivo no funciona se puede realizar la siguiente inspección: 
•	Se puede desplegar Jaeger aplicando su configuración. Utilice el siguiente comando
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/jaeger.yaml
 
•	Confirmamos el despliegue del pod en jaeger
Tras aplicar la configuración de Jaeger, compruebe si el pod de Jaeger está en funcionamiento:
kubectl get pods -n istio-system
Buscamos un pod con un nombre similar a jaeger-xxxxxxxx. Verificando de que su estado es Running. 
Si el pod de Jaeger se está ejecutando, se puede acceder al panel de control de Jaeger mediante powershell
istioctl dashboard jaeger
 

Conclusiones:
•	El uso de Docker, Kubernetes e Istio permite crear un entorno robusto y escalable. Docker facilita la creación de imágenes ligeras, mientras que Kubernetes se encarga de la orquestación y gestión de los contenedores en un clúster. Istio, por su parte, añade una capa de gestión de tráfico y observabilidad.
•	La creación de un Dockerfile y su posterior implementación en Kubernetes demuestra la eficiencia del contenedor en el ciclo de vida del desarrollo. La simplificación del proceso de despliegue mediante archivos de configuración YAML hace que el manejo de aplicaciones en la nube sea más accesible.
•	La implementación de herramientas de observabilidad como Grafana, Kiali y Jaeger permite monitorear el rendimiento de la aplicación, gestionar el tráfico de microservicios y realizar trazas de las solicitudes. Esto es esencial para identificar y solucionar problemas en entornos de producción.


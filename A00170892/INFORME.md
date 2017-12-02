# PROYECTO SISTEMAS DISTRIBUIDOS #

***Presentado por: Carlos Llano (A00170892) y Emmanuel Bolaños (A00309828)***

___

***1.Consigne los comandos de linux necesarios para el aprovisionamiento de los servicios empleados. 
En este punto no debe incluir archivos tipo Dockerfile solo se requiere que usted identifique los comandos o 
acciones que debe automatizar***

Aclaracion:
* Se utiliza el playground http://labs.play-with-k8s.com/ para ejecutar kubernetes
* El servicio web se emplea utilizando una imagen propia: ***dakar9499/python-service*** y consiste en un script 
de python que da el nombre del contenedor que lo ejecuta.

Comandos necesarios y que deben automatizarse:

***Despliegue del pod con las replicas y etiqueta exigidas***

```bash
kubectl run web-deployment --image=dakar9499/python-service --replicas=3 --port=5000 --labels=app=web
```

![img1](https://user-images.githubusercontent.com/17281733/33517993-a90866a8-d75b-11e7-9ecc-c6a4a33f9b5b.png)


***Creacion del servicio***
```bash
kubectl expose deployment web-deployment --name=python-service --port=8000 --target-port=5000
```

![img2](https://user-images.githubusercontent.com/17281733/33518032-7d6b61d4-d75c-11e7-9235-0e55d26e85d6.png)

Verificacion de funcionamiento:

En el playground utilizado se deben ejecutar los siguientes comandos:

```bash
export SERVICE_IP=$(kubectl get service python-service -o go-template='{{.spec.clusterIP}}')
export SERVICE_PORT=$(kubectl get service python-service -o go-template='{{(index .spec.ports 
kubectl run busybox  --generator=run-pod/v1 --image=busybox --restart=Never --tty -i --env "SERVICE_IP=$SERVICE_IP" --env "SERVICE_PORT=$SERVICE_PORT"
wget -qO- http://$SERVICE_IP:$SERVICE_PORT
```

![img3](https://user-images.githubusercontent.com/17281733/33518138-fed1fe30-d75d-11e7-9598-eea872dd3af1.png)


___

***2.Escriba los archivos Dockerfile para los servicios empleados junto con los archivos fuente necesarios.***

***Dockerfile para el serivicio web***
```dockerfile
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

***archivos fuente (app.py)***
```python
from flask import Flask
import os
import socket

app = Flask(__name__)
host = socket.gethostname()

@app.route('/')
def hello():
    return 'Hello World! My Host name is %s\n\n' % (host)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

Para que pueda usarse el servicio con kubernetes, se tuvo que subir a docker-cloud la imagen obtenida usando los siguientes comandos

```bash
docker build -t python-service .
docker login
docker tag python-service $DOCKER_ID_USER/python-service
docker push $DOCKER_ID_USER/python-service
```


___

***3.Escriba los archivos de configuración necesarios deployment.yml, service.yml para el despliegue de la infraestructura.  
Incluya un diagrama general de los componentes empleados***

***deployment.yml***

```yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  template: 
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: dakar9499/python-service
        ports:
        - containerPort: 5000
```

***service.yml***

```yml
apiVersion: v1
kind: Service
metadata:
  name: python-service
spec:
  ports:
  - port: 8000
    targetPort: 5000
    protocol: TCP
  selector:
    app: web
```

***diagrama general de los componentes empleados***
![captura](https://user-images.githubusercontent.com/17281733/33518870-778e4cf0-d76a-11e7-90bf-a83603cd8d29.PNG)


___

***4.Incluya evidencias que muestran el funcionamiento de lo solicitado.***

***Comandos para montar el cluster***
```bash
kubectl create -f deployment.yml
kubectl create -f service.yml
```

***Inicio con dos nodos***


![evidencia1](https://user-images.githubusercontent.com/17281733/33518958-8156e3d6-d76b-11e7-8710-493ff78c56c4.png)


***Escalamiento de pods***

![evidencia2](https://user-images.githubusercontent.com/17281733/33518979-c3a5281a-d76b-11e7-85f1-842b2c599819.png)

***Caída de uno de los nodos que forman parte del cluster***

Los Pods son reasignados automáticamente a un nodo saludable. En este caso, se elimino el nodo2 y los pods fueron asignados
al nodo 3

![evidencia5](https://user-images.githubusercontent.com/17281733/33519005-1d170a12-d76c-11e7-8097-e15e9d3bd52f.png)


___

***5.Documente algunos de los problemas encontrados y las acciones efectuadas para su solución al aprovisionar 
la infraestructura y aplicaciones***


***Problema: Referenciar un conjunto de pods***

Para que con el servicio se produzca el comportamiento de balanceador de carga, hay que identificar cuales son los pods
a los cuales la peticion sera redirigida. 

Inicialmente se penso que haciendo referencia al nombre del deployment funcionaria, sin embargo, kubernetes aporta una
directiva (selector) para hacer referencia a un conjunto de objetos (pods) si previamente se los ha etiquetado.

De esta forma:

***etiqueta de los pods***

![1](https://user-images.githubusercontent.com/17281733/33519108-f9cfbb56-d76d-11e7-92ec-06a8b3b0c211.png)

***referencia a los pods desde el servicio***

![2](https://user-images.githubusercontent.com/17281733/33519109-faddabe8-d76d-11e7-8666-55a98a108dbe.png)







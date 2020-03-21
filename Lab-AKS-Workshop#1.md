**PRACTICA AKS KUBERNETES PARA TECHDAYS DE MICROSOFT**
-------------------------------------------------------

Tutorial para eventos y meetups sobre AKS donde veremos:
- Paso1: Creando un Service Principal para AKS
- Paso2: Creando un cluster de AKS
- Paso3: Configurar Kubectl y Credenciales acceso AKS
- Paso4: Configurar Helm y repositorio estable Charts 
- Como configurar el monitoring
- Como escalar una aplicación en AKS

Chuleta: https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf

Requerimientos Tutorial:
- Azure y Kubernetes fundamentos basicos
- Conocimientos Azure CLI, shell.azure.com y Kubectl
- Suscripcion de Azure con permisos Admin para Azure Active Directory

Todo se realizará directamente desde la Shell de Azure mediante comandos Azure CLI.


### Configurar Suscripcion Azure para Workshop

Abrir una shell de Azure para consultar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***
```

### Paso1: Creando un Service Principal para AKS

Contexto: Cuando creamos un AKS, Azure automaticamente genera un Service Principal de AAD, pero esto no siempre ocurre (arrggg!!) y entonces tenemos que generarlo nosotros previamente para indicarlo en el comando: az aks create... 

El Service Principal es necesario para interactuar con otros recursos de Azure, como un Load Balancer, registry ACR, Virtual Networks,etc y si AKS no dispone de uno en el momento de la creación fallará y nos dara un error: "AADSTS700016: Application with identifier xxxxx-xxxxx-xxxxx-xxxxxx was not found in the directory xxxxx-xxxxx-xxxxx-xxxxxx.

Explicacion en Microsoft Docs y Proyecto GitHub Service principals for AKSm que podemos consultar en este enlace: 
https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal

Creamos el Service Principal y guardamos en notepad la respuesta JSON con AppID y Password, este SP se guarda en Azure Active Directory en la opcion de Aplicaciones Registradas:

```
$ az ad sp create-for-rbac --skip-assignment --name AKSClusterSantiPruebasServicePrincipal
...respuesta json...
```

### Paso2: Creando un cluster de AKS

Creamos el grupo de recursos y AKS con la opcion de autoescalado, la creacion del cluster puede tardar entre 10 y 15 minutos.
```
$ az group create --name rg-santi-pruebas-cluster-aks --location westeurope

$ az aks create --resource-group rg-santi-pruebas-cluster-aks \
    --name aks-santi-cluster-pruebas \
    --location westeurope \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --service-principal <appId> \
    --client-secret <password>
```

### Paso3: Configurar Kubectl y Credenciales acceso AKS

Necesitamos configurar la herramienta de Kubectl para trabajar con AKS
```
$ az aks install-cli (para instalar kubectl si no lo tenemos en local)

$ kubectl version

$ az aks get-credentials --resource-group <nombre-rg> --name <nombre-aks-cluster> --admin
  Merged "aks-santi-cluster-pruebas-admin" as current context in /home/santimacnet/.kube/config

$ kubectl config current-context (para ver contexto correcto AKS)

$ kubectl cluster-info

$ kubectl get nodes
```

### Paso4: Configurar Helm y repositorio estable Charts 


Helm, the package manager for Kubernetes. Como explican en su web oficial https://helm.sh y actualmente se encuentra en la version Helm 3.0: https://helm.sh/blog/helm-3-released

Necesitamos configurar el repositorio de Charts de Helm y asegurarlo que esta actualizado para trabajar desde nuestro entorno: 
```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com
"stable" has been added to your repositories

$ helm repo update
Update Complete. ⎈ Happy Helming!⎈

$ helm search repo stable
```

Mas detalles: https://v3.helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository

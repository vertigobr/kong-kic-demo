# Demo de BasicAuth no Kong Ingress Controller

## Subindo um cluster local com o Kong

Para subir um cluster local:

```sh
vkpr infra up
```

Nota: o [VKPR](https://docs.vkpr.net/) é um acelerador para adoção de Kubernetes e o comando acima é apenas um wrapper para subir um cluster k3d local sem ingress controller mas com load balancer na porta 8000.

Para instalar o Kong em modo ingress controller (mudamos a ingressClass para nginx no `values.yaml`):

```sh
helm repo add kong https://charts.konghq.com
helm repo update
helm upgrade -i -f values.yaml kong kong/kong
```

Teste a instalação do Kong:

```sh
curl localhost:8000
{"message":"no Route matched with those values"}
```

## Instale aplicação de exemplo com um Ingress

Instale uma aplicação de exemplo:

```sh
vkpr whoami install --default
```

Teste esta aplicação (se "whoami.localhost" não resolve para 127.0.0.1 dê sua marretada em "/etc/hosts"):

```sh
curl whoami.localhost:8000
Hostname: whoami-7df8d764d4-5bcn2
...
X-Real-Ip: 10.42.0.0
```

## Restringir acesso com BasicAuthentication

Para proteger esta aplicação usando um plugin de BasicAuthentication iremos usar um CRD do Kong e anotar o Ingress dela:

```sh
kubectl apply -f basic-auth.yml
kubectl annotate ingress whoami konghq.com/plugins=basic-auth-whoami -n vkpr
```

Testar a aplicação novamente dará erro 403:

```sh
curl whoami.localhost:8000
{
  "message":"Unauthorized"
}%
```

## Definir usuário com acesso

Para criar um usuário é necessário criar um KongConsumer e uma secret definindo sua senha:

```sh
kubectl create secret generic consumer-pwd -n vkpr  \
  --from-literal=kongCredType=basic-auth  \
  --from-literal=password=campeao \
  --from-literal=username=mengao
kubectl apply -f consumer.yml
```

Finalmente, para testar o acesso com credenciais basta o comando abaixo:

```sh
curl -u mengao:campeao whoami.localhost:8000
```

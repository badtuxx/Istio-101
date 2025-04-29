### Day 1: Introdução ao Istio

Seja muito bem-vindo(a) ao **Istio 101**! Neste primeiro artigo, vamos explorar:

- O que é o Istio e por que ele é tão importante em arquiteturas de microsserviços.
- Como o Istio se integra ao Kubernetes e traz observabilidade, segurança e controle de tráfego.
- As diferenças entre **sidecar mode** e **ambient mode** no *data plane*.
- Como funciona o suporte e o ciclo de vida do Istio, de acordo com a documentação oficial.

A ideia é oferecer um **passo a passo** amigável e didático, repleto de exemplos e demonstrações. Vamos explicar cada detalhe dessa história. Bora lá!

---

### O que é o Istio?

O **Istio** é um *Service Mesh*: uma camada de rede que fica entre seus microsserviços, fornecendo:

- **Observabilidade** (telemetria, logs, métricas, rastreamento distribuído).
- **Segurança** (criptografia de tráfego, autenticação e autorização).
- **Tráfego inteligente** (balanceamento de carga, roteamento avançado, *circuit breaker*, etc.).
- **Políticas de acesso** unificadas.

Em outras palavras, o Istio “alivia” o código dos seus microsserviços de várias responsabilidades complexas relacionadas à rede e segurança. Com isso, cada serviço pode focar na sua própria **regra de negócio**.

#### Por que preciso de um Service Mesh?

Em sistemas distribuídos (microsserviços), a comunicação entre componentes cresce exponencialmente. Se cada microsserviço precisar gerenciar logs, segurança, roteamento e afins, o código e a arquitetura ficam caóticos. O Istio simplifica tudo ao **centralizar** essas responsabilidades em um proxy e em um *control plane*. Assim, você ganha controle e visibilidade sobre o tráfego sem alterar as aplicações.

- **Principais benefícios**:
  - Observabilidade **completa** (telemetria, logs, rastreamento e métricas).
  - Controle de tráfego avançado (canary releases, *circuit breaker*, roteamento inteligente etc.).
  - Segurança de ponta a ponta (TLS, autenticação, autorização, mTLS).
  - Políticas de acesso unificadas e fáceis de gerenciar.

---

### Como o Istio funciona por dentro?

De acordo com a [documentação oficial do Istio](https://istio.io/latest/docs/):

> O Istio utiliza um proxy para **interceptar** todo o tráfego de rede, permitindo um amplo conjunto de recursos orientados à aplicação, baseados na configuração que você definir.

O **control plane** (gerenciado pelo processo **istiod**) recebe as configurações desejadas (políticas, roteamento, regras de segurança etc.) e atualiza os proxies dinamicamente sempre que houver mudanças.

O **data plane** é onde o tráfego entre os serviços de fato acontece. Sem um *service mesh*, a rede não entende o tráfego em nível de aplicação (HTTP, gRPC etc.) a ponto de tomar decisões ou aplicar políticas. O Istio faz essa “intermediação” de forma transparente.

#### Dois modos de Data Plane: Sidecar e Ambient

1. **Sidecar Mode**:
   - O Istio implanta um *Envoy proxy* junto de **cada Pod** (um container extra rodando ao lado do container principal).
   - É o modo “clássico” e mais difundido.

2. **Ambient Mode**:
   - Modelo mais recente.
   - Usa um **proxy por nó** (per-node, camada 4) e, opcionalmente, um **proxy por namespace** para recursos de camada 7.
   - Promete simplificar a implantação, reduzindo a sobrecarga de gerenciar um sidecar por Pod.

#### Data Plane e Control Plane

1. **Data Plane (Plano de Dados)**
   - Onde o tráfego real entre os serviços é **interceptado** e **analisado**.
   - No **sidecar mode**, cada Pod tem um **Envoy** como sidecar para controlar esse tráfego.
   - No **ambient mode**, temos um **proxy por nó** (camada 4) e, se necessário, um **proxy por namespace** (camada 7).

2. **Control Plane (Plano de Controle)**
   - Gerenciado pelo **istiod**, é onde estão as configurações, políticas e inteligência do Mesh.
   - Ele “programa” os proxies (Envoy ou Ztunnel, dependendo do modo) para aplicar roteamento, segurança e telemetria.

---

### Integrando Istio ao Kubernetes

O Kubernetes orquestra os containers; o Istio **orquestra o tráfego** entre eles. Em um cluster Kubernetes:

1. **Cada serviço** pode ser gerenciado pelo Istio (seja via *sidecar mode* ou *ambient mode*).
2. No *sidecar mode*, o Istio injeta automaticamente o *sidecar* Envoy no Pod.
3. No *ambient mode*, não é necessário container extra no Pod: o proxy fica por nó ou por namespace.
4. O **istiod** (control plane) envia as configurações para todos os proxies presentes no cluster.

**Principais objetos de rede do Istio**:
- **Gateway**: Gerencia tráfego de entrada (Ingress) e saída (Egress) do cluster.
- **VirtualService**: Define regras de roteamento avançado (por ex., canary releases, A/B testing etc.).
- **DestinationRule**: Especifica comportamentos de conexão (retries, timeouts, TLS etc.).

---

### Requisitos para seguir este livro

Para acompanhar as demonstrações e laboratórios, você precisa de:

- Um **cluster Kubernetes** funcional (pode ser **Minikube**, **Kind**, **MicroK8s**, **k3s** etc.).
- Ferramentas como `kubectl` e `curl` instaladas na sua máquina.
- O cliente do Istio (`istioctl`), que usaremos para gerenciar a instalação e configurações.

Não se preocupe, já vamos mostrar como instalar tudo, passo a passo. :)

---

### Perfis de Instalação (Profiles)

Na hora de instalar o Istio, podemos escolher entre diferentes “perfis”, cada qual ativa ou desativa determinados componentes. Alguns dos mais comuns:

- **default**: Instala istio-ingressgateway e istiod.
- **demo**: Voltado para estudos e testes; inclui istio-egressgateway.
- **minimal**: Instala só o istiod, sem gateways.
- **remote**: Cenários de **multicluster** (um cluster usa recursos de outro).
- **empty**: Não traz **nenhum** componente (perfeito para instalação totalmente customizada).
- **preview**: Inclui recursos em desenvolvimento ou pré-lançamento.
- **ambient**: Ativa o **ambient mode** (instala istiod, CNI, Ztunnel etc.).

#### Tabela de Comparação dos Principais Perfis

| **Componente**          | **default** | **demo** | **minimal** | **remote** | **empty** | **preview** | **ambient** |
|-------------------------|:----------:|:-------:|:----------:|:----------:|:--------:|:----------:|:----------:|
| **istio-egressgateway** |            |    ✔    |            |            |          |            |            |
| **istio-ingressgateway**|     ✔      |    ✔    |            |            |          |     ✔      |            |
| **istiod**              |     ✔      |    ✔    |     ✔      |            |          |     ✔      |     ✔      |
| **CNI**                 |            |         |            |            |          |            |     ✔      |
| **Ztunnel**             |            |         |            |            |          |            |     ✔      |

> **Dica**: Se você está começando, o perfil **demo** costuma ser o mais indicado. Já para testar o **ambient mode**, use o perfil **ambient**.

---

### Instalando o Istio (Exemplo com Sidecar Mode)

Apesar de o **ambient mode** ser recente, o **sidecar mode** ainda é o mais usado. Vamos ver a instalação via **istioctl**, a CLI oficial:

1. **Baixar a versão mais recente do Istio**
   Acesse a [página de releases do Istio no GitHub](https://github.com/istio/istio/releases) e escolha uma versão (exemplo: 1.24.x).
   ```bash
   curl -L https://istio.io/downloadIstio | sh -
   cd istio-1.24.x
   sudo mv bin/istioctl /usr/bin/
   ```
   > **Dica**: Ajuste `1.24.x` para a versão atual, conforme a [documentação oficial](https://istio.io/latest/docs/setup/getting-started/).

2. **Verificar o istioctl**
   ```bash
   istioctl version
   ```
   Se o comando mostrar a versão instalada, significa que seu binário está OK.

3. **Instalar o Istio no cluster**
   Com um cluster Kubernetes ativo (Kind, Minikube etc.), use, por exemplo, o perfil **demo**:
   ```bash
   istioctl install --set profile=demo -y
   ```

   Você deve ver algo como:

   ```bash
   This will install the Istio 1.24.2 profile "demo" into the cluster. Proceed? (y/N) y
   ✔ Istio core installed ⛵️                                                  
   ✔ Istiod installed 🧠                                                      
   ✔ Egress gateways installed 🛫                                             
   ✔ Ingress gateways installed 🛬                                            
   ✔ Installation complete
   ```

4. **Habilitar a injeção automática de sidecars**
   Para que os Pods recebam o sidecar Envoy automaticamente, ative a injeção no *namespace* desejado:

   ```bash
   kubectl create namespace giropops
   kubectl label namespace giropops istio-injection=enabled
   ```

5. **Verificar se o Istio foi instalado corretamente**
   ```bash
   kubectl get pods -n istio-system
   ```
   Você deve ver algo como:
   ```bash
   NAME                                    READY   STATUS    RESTARTS   AGE
   istio-egressgateway-84479c75b8-6nb6g    1/1     Running   0          118s
   istio-ingressgateway-5886f95f4d-46bdq   1/1     Running   0          118s
   istiod-645bc8f9f4-c9qtg                 1/1     Running   0          2m4s
   ```
   - **istiod** = Control Plane
   - **istio-ingressgateway** e **istio-egressgateway** = Gateways para tráfego de entrada e saída.

---

### Principais Componentes e Objetos do Istio

Para organizar as informações:

| Componente/Objeto       | Função Principal                                                            |
|-------------------------|----------------------------------------------------------------------------|
| **istiod**              | Control plane: distribui configurações para os proxies Envoy.              |
| **Envoy Proxy**         | Intercepta e controla o tráfego (no sidecar ou modo ambient).              |
| **IngressGateway**      | Recebe tráfego de entrada (Ingress) no cluster.                            |
| **EgressGateway**       | Controla tráfego de saída (Egress) do cluster.                             |
| **Ambient Mode**        | Proxy por nó (camada 4) e, opcionalmente, por namespace (camada 7).        |
| **Sidecar Mode**        | Proxy Envoy em cada Pod (sidecar).                                         |
| **Gateway**             | Controla entrada/saída (Ingress/Egress) de tráfego HTTP/TCP no cluster.    |
| **VirtualService**      | Define regras de roteamento (ex.: canary, A/B testing).                    |
| **DestinationRule**     | Especifica políticas de conexão (retries, circuit breaker, TLS).           |
| **ServiceEntry**        | Permite incluir serviços externos ao Mesh.                                 |
| **Sidecar**             | Container extra em cada Pod, com o proxy Envoy.                            |
| **AuthorizationPolicy** | Define políticas de autorização (quem pode acessar o quê).                 |

---

### Uma Nova Forma de Usar o Istio com o Ambient Mode

O **ambient mode** elimina a necessidade de um *sidecar* por Pod. Em vez disso:

- Utiliza um **proxy por nó** (camada 4).
- Pode ter um **proxy por namespace** para camada 7.
- Geralmente, é instalado com o **perfil** `ambient` no `istioctl`.

Isso reduz o overhead operacional e pode facilitar a adoção do Istio em cenários específicos. Contudo, como o *ambient mode* ainda está em **evolução**, nem todos os recursos avançados do *sidecar mode* podem estar disponíveis de forma idêntica.

> Nós iremos explorar o *ambient mode* em detalhes em um dos próximos *days*.

---

### Políticas de Suporte e Lançamentos do Istio

Segundo a [documentação oficial](https://istio.io/latest/about/supported-releases/), o Istio segue um ciclo de vida com:

- **Development Build**: Versões de teste, sem suporte oficial.
- **Minor Release**: Recebe suporte até 6 semanas após a liberação da próxima versão (N+2).
- **Patch Release**: Correções pontuais para versões *minor*.
- **Security Patch**: Como um *patch*, mas focado em falhas de segurança.

#### Esquema de Nomeação

- `<major>.<minor>.<patch>` (ex.: `1.24.1`).
- `<minor>` cresce a cada grande lançamento.
- `<patch>` conta as correções lançadas para aquele `<minor>`.

#### Control Plane/Data Plane Skew

Recomenda-se que o **Control Plane** (istiod) nunca esteja mais de uma versão à frente do **Data Plane** (proxies). Ter proxies em versão superior ao *control plane* não é recomendado.

#### Status de Suporte das Versões

Exemplo de tabela de suporte (simplificada):

| Versão  | Em Suporte? | Data Lanç.   | EOL Estimado | K8s Testado/Suportado  | K8s Testado (não suportado) |
|---------|------------|--------------|--------------|------------------------|-----------------------------|
| **1.24** | Sim        | Nov 7, 2024  | ~Ago 2025     | 1.28,1.29,1.30,1.31    | 1.23,1.24,1.25,1.26,1.27    |
| **1.23** | Sim        | Aug 14, 2024 | ~Mai 2025     | 1.27,1.28,1.29,1.30    | 1.23,1.24,1.25,1.26         |
| **1.22** | Sim        | May 13, 2024 | ~Jan 2025     | 1.27,1.28,1.29,1.30    | 1.23,1.24,1.25,1.26         |
| **1.21** | Sim        | Mar 13, 2024 | Sep 27, 2024  | 1.26,1.27,1.28,1.29    | 1.23,1.24,1.25              |
| 1.20    | Não        | Nov 14, 2023 | Jun 25, 2024  | 1.25,1.26,1.27,1.28    | 1.23,1.24                   |

> Consulte a [página oficial de releases](https://istio.io/latest/about/supported-releases/) para informações atualizadas.

---

### Criando o Nosso Primeiro Ingress Gateway

Para expor serviços do cluster externamente, usamos o **Ingress Gateway** do Istio. Ele recebe o tráfego na borda do cluster e pode aplicar políticas de segurança, autenticação etc.

Existem **dois tipos** de Gateway:
- **Ingress Gateway**: Tráfego **entrando** no cluster.
- **Egress Gateway**: Tráfego **saindo** do cluster.

Vamos criar um Gateway simples no namespace `giropops`. Crie um arquivo YAML (`ingress-gateway.yaml`):

```yaml
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: giropops
  namespace: giropops
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
```

- O `selector` indica que este Gateway será servido pelo **istio-ingressgateway**.
- Ele escuta na porta **80** para qualquer host.

Para aplicar essa configuração:

```bash
kubectl apply -f ingress-gateway.yaml
```

Agora temos uma “porta de entrada” (porta 80) no namespace `giropops`.

---

### Meu Primeiro VirtualService

O **VirtualService** dita **como** o tráfego que chega ao Gateway é roteado para os serviços internos.

Vamos criar dois *Deployments*:

```bash
kubectl create deployment nginx-giropops --image linuxtips/nginx_giropops:v1 --replicas 1 -n giropops
kubectl expose deployment nginx-giropops --port 80 -n giropops

kubectl create deployment nginx-strigus --image linuxtips/nginx_strigus:v1 --replicas 1 -n giropops
kubectl expose deployment nginx-strigus --port 80 -n giropops
```

Cada um serve uma página HTML simples com mensagens diferentes. Para rotear o tráfego, crie um arquivo `virtualservice.yaml`:

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: nginx-giropops
  namespace: giropops
spec:
  hosts:
    - "*"
  gateways:
    - giropops
  http:
    - name: nginx-giropops
      match:
        - uri:
            prefix: /giropops
      rewrite:
        uri: "/"
      route:
        - destination:
            host: nginx-giropops
            port:
              number: 80
    - name: nginx-strigus
      match:
        - uri:
            prefix: /strigus
      rewrite:
        uri: "/"
      route:
        - destination:
            host: nginx-strigus
            port:
              number: 80
```

- **hosts**: Aplica para todos os hosts (`"*"`).  
- **gateways**: Aponta para o Gateway que criamos (`giropops`).  
- **http**: Agrupa as rotas.  
  - `match`: Define como identificar a rota (ex.: `prefix: /giropops`).  
  - `rewrite`: Ajusta o caminho antes de repassar ao serviço (remove `/giropops`, trocando para `/`).  
  - `route`: Especifica **para onde** enviar o tráfego (no caso, `nginx-giropops` ou `nginx-strigus`).

Aplica a configuração:

```bash
kubectl apply -f virtualservice.yaml
```

Pronto! Se tudo deu certo, quando você **acessar** o Ingress Gateway e usar o caminho `/giropops`, o Istio envia a requisição ao **nginx-giropops**. Já `/strigus` vai para o **nginx-strigus**.

#### Descobrindo o IP do Ingress Gateway

Depende do seu cluster:

- **Kind**: EXTERNAL-IP costuma ser `<pending>`. Você pode fazer *port-forward* para acessar localmente.  
- **Minikube**: Use `minikube ip` para descobrir o IP.  
- **EKS, GKE, AKS**: Normalmente, um LoadBalancer atribui um IP externo.  
- **Port-forward** (solução universal para teste local):
  ```bash
  kubectl port-forward svc/istio-ingressgateway 8888:80 -n istio-system
  curl localhost:8888/giropops
  curl localhost:8888/strigus
  ```
  - `/giropops` deve mostrar “Bem-vindo ao Giropops Nginx V1!”.  
  - `/strigus` deve mostrar “Bem-vindo ao Giropops Nginx V2!”.

Se estiver funcionando, parabéns! Você criou seu primeiro Ingress Gateway e VirtualService com Istio.



### DestinationRule na Prática

O **DestinationRule** é o objeto do Istio responsável por definir **políticas de conexão** para um destino específico no Service Mesh. Isso permite controlar comportamentos como:

- **Resiliência** e **tolerância a falhas** (retries, *circuit breaker*, *outlier detection*).  
- **Time-out** para requisições.  
- **TLS** (ex.: modo mTLS entre serviços).  
- **Políticas de balanceamento** (round robin, least connection etc.).

Em resumo, se o **VirtualService** diz **“para onde”** enviar uma requisição (roteamento), o **DestinationRule** define **“como”** essa requisição será tratada a nível de conexão.

Calma que vou te explicar já já como criar um *DestinationRule* passo a passo.

### DestinationRule para o nginx

Vamos criar o nosso `DestinationRule` para o serviço **nginx-giropops** que já criamos no namespace `giropops`. Esse objeto vai definir políticas de resiliência e de balanceamento de carga para o nosso microserviço.

Exemplo de `destinationrule.yaml`:

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: giropops-destinationrule
  namespace: giropops
spec:
  host: "*"
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    connectionPool:
      http:
        http1MaxPendingRequests: 100
        maxRequestsPerConnection: 10
      tcp:
        maxConnections: 100
    outlierDetection:
      consecutive5xxErrors: 2
      interval: 10s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
    tls:
      mode: DISABLE
```

#### Explicando Cada Campo

- **host**: Aponta para o serviço `*` (todos). Em cenários mais comuns, você usa o FQDN do serviço, por exemplo `nginx-giropops.giropops.svc.cluster.local`.  
- **loadBalancer**: Define a forma de balanceamento de carga, aqui usando `ROUND_ROBIN`, que distribui requisições igualmente entre as réplicas do `nginx-giropops`.  
- **connectionPool**:
  - **http** e **tcp**: Controlam o número de conexões simultâneas e requisições pendentes.  
  - `http1MaxPendingRequests`: Limite de requisições HTTP 1.x pendentes.  
  - `maxRequestsPerConnection`: Máximo de requisições antes de encerrar uma conexão.  
  - `maxConnections`: Limite global de conexões TCP.  
- **outlierDetection**: Observa quantos erros 5xx consecutivos acontecem para “expulsar” temporariamente uma réplica problemática do pool de rota.  
  - `consecutive5xxErrors: 2`: Ejecta o endpoint após 2 erros 5xx seguidos.  
  - `interval: 10s`: Intervalo de verificação das estatísticas.  
  - `baseEjectionTime: 30s`: Tempo mínimo que o endpoint fica “banido” do pool.  
  - `maxEjectionPercent: 50`: Máximo de endpoints que podem ser ejetados ao mesmo tempo.  
- **tls**: Neste exemplo está `DISABLE`, mas você poderia configurar `ISTIO_MUTUAL` para habilitar mTLS, por exemplo.

Esses parâmetros trazem **resiliência** ao serviço, pois evitam que erros momentâneos ou réplicas desatualizadas derrubem a sua aplicação por completo. É o famoso “morrer, mas passando bem” do mundo DevOps. :)

#### Aplicando o DestinationRule

Basta rodar o comando:

```bash
kubectl apply -f destinationrule.yaml
```

Verifique se a criação foi bem-sucedida:

```bash
kubectl get destinationrule -n giropops
```

Você deve ver o objeto `nginx-destinationrule` na lista de DestinationRules. Agora, quando o seu **VirtualService** roteia requisições para o `nginx-giropops`, o Istio aplicará automaticamente essas regras de **outlier detection** e **balanceamento**.



### Resumo de Como o DestinationRule Entra no Fluxo

1. **Ingress Gateway** recebe o tráfego externo na porta 80 ou 443.  
2. **VirtualService** decide para qual host (serviço) encaminhar a requisição (ex.: `nginx-giropops`).  
3. **DestinationRule** informa **como** esse host vai tratar a conexão (retries, TLS, outlier detection, etc.).  

Assim, unindo *Gateway*, *VirtualService* e *DestinationRule*, você tem uma **malha de serviço** completa e bem controlada.  

#### Tabela de Itens Envolvidos

| **Objeto**          | **Responsabilidade**                                          |
|---------------------|--------------------------------------------------------------|
| **Gateway**         | Expõe tráfego de entrada (Ingress) ou saída (Egress).       |
| **VirtualService**  | Roteia o tráfego para cada serviço (ex.: /giropops → nginx-giropops).|
| **DestinationRule** | Define políticas de conexão (balanceamento, retries, TLS).   |

> Para mais detalhes sobre cada campo, consulte a [Documentação Oficial do Istio sobre Destination Rules](https://istio.io/latest/docs/reference/config/networking/destination-rule/).



### Vamos Ver Tudo Isso Acontecendo no Kiali

O **Kiali** é uma ferramenta com interface gráfica que permite visualizar a malha de serviços do Istio. Ele mostra gráficos, métricas, topologia de serviços e muito mais.

#### Observando o Mesh e o DestinationRule pelo Kiali

Depois de instalar o **Kiali** (seja pela [instalação rápida do Istio](https://raw.githubusercontent.com/istio/istio/release-1.24/samples/addons/kiali.yaml) ou pelas [instruções oficiais do projeto Kiali](https://kiali.io/documentation/)), vamos utilizá-lo para verificar **como** nossos serviços, políticas de roteamento e configurações (incluindo **DestinationRules**) estão se comportando no **Service Mesh**.  



#### Antes, o Prometheus

Para que o Kiali funcione corretamente, é necessário que o **Prometheus** esteja instalado e em execução no seu cluster. O Prometheus é responsável por coletar métricas e estatísticas dos serviços e do Service Mesh.

Vamos instalar uma configuração básica do Prometheus no cluster por enquanto:

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.24/samples/addons/prometheus.yaml
```

Isso instala o Prometheus no namespace `istio-system`. Aguarde alguns segundos até que todos os Pods estejam no estado `Running`.

Não vamos entrar em detalhes sobre o Prometheus neste *day*, mas saiba que ele é essencial para o Kiali funcionar corretamente — e nós o exploraremos em mais profundidade em um dos próximos *days*.

#### Instalar o Kiali no Cluster

Instalação rápida (demo) no Istio:

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.24/samples/addons/kiali.yaml
```

- Cria os recursos básicos do Kiali (Deployment, Service, ConfigMap) no namespace `istio-system`.  
- Ideal para laboratórios e testes. Não é recomendado para produção sem ajustes de segurança.



#### Acessar o Console do Kiali

A maneira mais simples de acessar o Kiali é usando o comando `istioctl`:

```bash
istioctl dashboard kiali
```

Isso abrirá o console do Kiali no seu navegador padrão, **automagicamente**. Se preferir, você pode acessar manualmente:

- **Kind/Minikube**: Use `kubectl port-forward` para acessar o serviço localmente.  
- **EKS, GKE, AKS**: Use o IP do LoadBalancer ou o DNS do serviço.

Mas vai por mim, com o `istioctl` é mais fácil. :)

Com o Kiali, você ganha uma **visão global** do que está acontecendo no seu Service Mesh, podendo diagnosticar problemas de forma rápida e intuitiva, além de validar se suas configurações de Istio (DestinationRules, VirtualServices etc.) estão surtindo o efeito desejado.  

Aproveite para brincar com diferentes cenários, testar novas **DestinationRules** e observar tudo em tempo real no Kiali. Boa viagem! hahah :)



### Vamos Colocar Fogo no Parquinho com o Hey

O **Hey** é uma ferramenta de benchmarking e teste de carga para aplicações web. Com ele, você pode simular milhares de requisições HTTP para testar a resiliência e o desempenho dos seus serviços. É super simples de usar e pode ser um grande aliado na hora de validar as suas configurações de Istio.

#### Instalando o Hey

No Linux, você pode instalar o **Hey** com a ferramenta de instalação de pacotes da sua distro, como `apt` ou `yum`, ou ainda através de um simples `go get`:

```bash
sudo apt install hey
```
ou
```bash
go get -u github.com/rakyll/hey
```

#### Testando o nginx-giropops com o Hey

Vamos fazer um teste simples de carga no nosso serviço **nginx-giropops** usando o **Hey**. Para isso, basta rodar o comando:

```bash
hey -n 1000 -z 1m -c 10 http://<IP_DO_INGRESS_GATEWAY>/giropops
```

- `-n 1000`: Realiza 1000 requisições.  
- `-z 1m`: Limita a duração do teste a 1 minuto.  
- `-c 10`: 10 conexões simultâneas.  
- `http://<IP_DO_INGRESS_GATEWAY>/giropops`: Endereço do Ingress Gateway.  
- `/giropops`: Caminho para o serviço **nginx-giropops**.

Lembre-se de substituir `<IP_DO_INGRESS_GATEWAY>` pelo IP correto do seu cluster. O **Hey** vai disparar 1000 requisições para o serviço `nginx-giropops` em 1 minuto, com 10 conexões simultâneas.

Assim, conseguimos testar a resiliência do nosso serviço e ver como o Istio lida com as políticas de **DestinationRule** que configuramos anteriormente.



#### Visualizando os Resultados no Kiali

Depois de rodar o teste com o **Hey**, você pode voltar ao **Kiali** para ver como o Istio lidou com as requisições. No Kiali, você pode:

- **Verificar** se as requisições foram distribuídas corretamente entre as réplicas do `nginx-giropops`.  
- **Observar** se as políticas de resiliência foram aplicadas corretamente.  
- **Validar** se o balanceamento de carga (round robin) está funcionando como esperado.  
- **Identificar** possíveis gargalos ou falhas no serviço.  
- **Ajustar** as configurações de **DestinationRule** conforme necessário.

Você consegue observar em tempo real as requisições sendo distribuídas entre as réplicas do `nginx-giropops`. Brinque com as opções do Kiali, pois você literalmente verá o tráfego passando por dentro do seu cluster.

Vamos aumentar as réplicas do `nginx-giropops` para 3 e rodar o teste novamente com o **Hey**:

```bash
kubectl scale deployment nginx-giropops --replicas=3 -n giropops
```

Depois, rode o **Hey** novamente para ver como o Istio lida com mais réplicas do serviço:

```bash
hey -n 1000 -z 1m -c 10 http://<IP_DO_INGRESS_GATEWAY>/giropops
```

Habilite algumas opções na tela *Traffic Graph* do Kiali para ver mais detalhes sobre o tráfego. Você pode ver o número de requisições, o tempo de resposta, o volume de tráfego e muito mais.

### Agora é com você!

Agora que você já conhece muita coisa do Istio, chegou a hora de praticar! Crie novos **VirtualServices**, **DestinationRules** e **Gateways**, teste diferentes cenários com o **Hey** e observe tudo em tempo real no **Kiali**.  
Use outras aplicações, crie novos serviços e explore tudo o que o Istio tem a oferecer. É uma ferramenta poderosa e flexível, então aproveite para aprender na prática.

Para que você possa reproduzir tudo o que estamos fazendo, pode executar tudo localmente usando o **Kind**. Abaixo, um arquivo de configuração de exemplo para criar o cluster:

```bash
vim kind.yaml
```
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 8888
    protocol: TCP
  - containerPort: 443
    hostPort: 4443
    protocol: TCP
```

Para criar o cluster:

```bash
kind create cluster --config kind.yaml
```

Com isso, você terá um cluster **Kind** com port-forwarding nas portas 8888 e 4443 para os serviços HTTP e HTTPS, respectivamente, facilitando o acesso aos serviços.

Depois é só instalar o Istio, Prometheus e o Kiali e **começar a brincadeira**!



## Resumo do Day-1

1. **Istio** é um Service Mesh que traz observabilidade, segurança e controle de tráfego para microsserviços.  
2. Ele é dividido em **Control Plane** (*istiod*) e **Data Plane** (que pode ser *sidecar mode* ou *ambient mode*).  
3. Integra-se ao **Kubernetes** injetando (ou não) proxies em cada Pod (sidecar) ou por nó (ambient).  
4. Há diversos **perfis** de instalação (como *default*, *demo*, *minimal*, *ambient*), cada um ativando componentes distintos.  
5. O **istioctl** é a CLI recomendada para gerenciar a instalação e configuração.  
6. **Gateways**, **VirtualServices** e **DestinationRules** são os principais objetos de rede do Istio.  
   - **DestinationRule** especifica **como** o tráfego será tratado (retries, timeouts, TLS etc.).  
7. O projeto segue uma política de suporte clara, com releases *minor*, *patch* e *security patch*.  
8. Combinando **VirtualService** + **DestinationRule**, você obtém um fluxo de tráfego **personalizado** e **resiliente**.



**Links Úteis**:

- [Documentação Oficial do Istio](https://istio.io/latest/docs/)  
- [Repositório Oficial no GitHub](https://github.com/istio/istio)  
- [Exemplos e Tutoriais Oficiais](https://istio.io/latest/docs/examples/)  
- [Perfis de Instalação](https://istio.io/latest/docs/setup/additional-setup/config-profiles/)  
- [Políticas de Suporte e Versões](https://istio.io/latest/about/supported-releases/)  
- [Destination Rules em Detalhes](https://istio.io/latest/docs/reference/config/networking/destination-rule/)  
- [Addons de Istio (Kiali, Grafana, Jaeger)](https://istio.io/latest/docs/addons/)  
- [Documentação Oficial do Kiali](https://kiali.io/documentation/)  

Bons estudos e até o próximo *day*! #VAIIII  

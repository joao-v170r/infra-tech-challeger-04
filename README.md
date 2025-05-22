# Contexto

O projeto pode ser visualizado em: https://www.youtube.com/watch?v=DLOVqnFtUb0

### Serviços
* Pedido Receiver: https://github.com/eduardoesr/micro-receiver-clean-arch
* Produto: https://github.com/eduardoesr/micro-produto-clean-arch
* Pagamento: https://github.com/joao-v170r/micro-pagamento-clean-arch
* Cliente: https://github.com/joao-v170r/micro-cliente-clean-arch
* Pedido: https://github.com/joao-v170r/micro-pedido-clean-arch
* Estoque: https://github.com/eduardoesr/micro-estoque-clean-arch

## Documentação do Projeto de Microsserviços

Este projeto é composto por uma série de microsserviços interconectados, utilizando Docker e Docker Compose para orquestração e gerenciamento. Cada serviço tem sua própria base de dados MongoDB e, em alguns casos, interage com outros serviços ou com uma fila de mensagens RabbitMQ.

### 1. Visão Geral da Arquitetura

O sistema é modularizado em vários microsserviços, cada um com uma responsabilidade específica e um banco de dados MongoDB dedicado. A comunicação entre os serviços é realizada via HTTP, e há um serviço de fila de mensagens (RabbitMQ) para comunicação assíncrona entre o `pedido-app` e o `receiver-app`.

### 2. Serviços

Abaixo estão os detalhes de cada serviço configurado no `docker-compose.yml`:

#### 2.1. `cliente-app`
* **Descrição**: Microsserviço responsável pela gestão de clientes.
* **Porta Exposta**: 8080 (mapeada para 8080 no host).
* **Banco de Dados**: `mongo-cliente` (MongoDB) na porta 27017.
* **Variáveis de Ambiente**:
    * `SPRING_DATA_MONGODB_URI`: `mongodb://admin:admin123@mongo-cliente:27017/db_cliente?authSource=admin`.
    * `SPRING_PROFILES_ACTIVE`: `prod`.
* **Dependências**: `mongo-cliente` (aguarda o serviço estar saudável).
* **Healthcheck**: Verifica a URL `http://localhost:8080/actuator/health`.

#### 2.2. `mongo-cliente`
* **Descrição**: Instância MongoDB para o serviço de cliente.
* **Imagem**: `mongo:7.0`.
* **Porta Exposta**: 27017 (mapeada para 27017 no host).
* **Volume**: `mongodb_cliente_data` para persistência de dados.
* **Variáveis de Ambiente**: Credenciais de root e banco de dados inicial.
* **Healthcheck**: `mongosh --eval "db.adminCommand('ping')"`.

#### 2.3. `pedido-app`
* **Descrição**: Microsserviço responsável pela gestão de pedidos.
* **Porta Exposta**: 8081 (mapeada para 8081 no host).
* **Banco de Dados**: `mongodb-pedido` (MongoDB) na porta 27017.
* **Variáveis de Ambiente**:
    * `SPRING_DATA_MONGODB_URI`: `mongodb://admin:admin123@mongodb-pedido:27017/db_pedido?authSource=admin`.
    * `SPRING_PROFILES_ACTIVE`: `prod`.
    * `CLIENTE_SERVICE_URL`: `http://cliente-app:8080`.
    * `PAGAMENTO_SERVICE_URL`: `http://pagamento-app:8082`.
    * `PAGAMENTO_PRODUTO_URL`: `http://produto-app:8083`.
    * `PAGAMENTO_ESTOQUE_URL`: `http://estoque-app:8084`.
* **Dependências**: `mongodb-pedido` e `rabbitmq` (ambos aguardam estar saudáveis).
* **Healthcheck**: Verifica a URL `http://localhost:8081/actuator/health`.

#### 2.4. `mongodb-pedido`
* **Descrição**: Instância MongoDB para o serviço de pedido.
* **Imagem**: `mongo:7.0`.
* **Porta Exposta**: 27018 (mapeada para 27017 no container).
* **Volume**: `mongodb_pedido_data` para persistência de dados.
* **Variáveis de Ambiente**: Credenciais de root e banco de dados inicial.
* **Healthcheck**: `mongosh --eval "db.adminCommand('ping')"`.

#### 2.5. `pagamento-app`
* **Descrição**: Microsserviço responsável pela gestão de pagamentos.
* **Porta Exposta**: 8082 (mapeada para 8082 no host).
* **Banco de Dados**: `mongodb-pagamento` (MongoDB) na porta 27017.
* **Variáveis de Ambiente**:
    * `SPRING_DATA_MONGODB_URI`: `mongodb://admin:admin123@mongodb-pagamento:27017/db_pagamento?authSource=admin`.
    * `SPRING_PROFILES_ACTIVE`: `prod`.
* **Dependências**: `mongodb-pagamento` (aguarda o serviço estar saudável).
* **Healthcheck**: Verifica a URL `http://localhost:8082/actuator/health`.

#### 2.6. `mongodb-pagamento`
* **Descrição**: Instância MongoDB para o serviço de pagamento.
* **Imagem**: `mongo:7.0`.
* **Porta Exposta**: 27019 (mapeada para 27017 no container).
* **Volume**: `mongodb_pagamento_data` para persistência de dados.
* **Variáveis de Ambiente**: Credenciais de root e banco de dados inicial.
* **Healthcheck**: `mongosh --eval "db.adminCommand('ping')"`.

#### 2.7. `estoque-app`
* **Descrição**: Microsserviço responsável pela gestão de estoque.
* **Porta Exposta**: 8084 (mapeada para 8084 no host).
* **Banco de Dados**: `mongodb-estoque` (MongoDB) na porta 27017.
* **Variáveis de Ambiente**:
    * `SPRING_DATA_MONGODB_URI`: `mongodb://admin:admin123@mongodb-estoque:27017/db_estoque?authSource=admin`.
    * `SPRING_PROFILES_ACTIVE`: `prod`.
* **Dependências**: `mongodb-estoque` (aguarda o serviço estar saudável).
* **Healthcheck**: Verifica a URL `http://localhost:8084/actuator/health`.

#### 2.8. `mongodb-estoque`
* **Descrição**: Instância MongoDB para o serviço de estoque.
* **Imagem**: `mongo:7.0`.
* **Porta Exposta**: 27020 (mapeada para 27017 no container).
* **Volume**: `mongodb_estoque_data` para persistência de dados.
* **Variáveis de Ambiente**: Credenciais de root e banco de dados inicial.
* **Healthcheck**: `mongosh --eval "db.adminCommand('ping')"`.

#### 2.9. `produto-app`
* **Descrição**: Microsserviço responsável pela gestão de produtos.
* **Porta Exposta**: 8083 (mapeada para 8083 no host).
* **Banco de Dados**: `mongodb-produto` (MongoDB) na porta 27017.
* **Variáveis de Ambiente**:
    * `SPRING_DATA_MONGODB_URI`: `mongodb://admin:admin123@mongodb-produto:27017/db_produto?authSource=admin`.
    * `SPRING_PROFILES_ACTIVE`: `prod`.
* **Dependências**: `mongodb-produto` (aguarda o serviço estar saudável).
* **Healthcheck**: Verifica a URL `http://localhost:8083/actuator/health`.

#### 2.10. `mongodb-produto`
* **Descrição**: Instância MongoDB para o serviço de produto.
* **Imagem**: `mongo:7.0`.
* **Porta Exposta**: 27016 (mapeada para 27017 no container).
* **Volume**: `mongodb_produto_data` para persistência de dados.
* **Variáveis de Ambiente**: Credenciais de root e banco de dados inicial.
* **Healthcheck**: `mongosh --eval "db.adminCommand('ping')"`.

#### 2.11. `receiver-app`
* **Descrição**: Microsserviço responsável por receber mensagens de fila, possivelmente processando-as.
* **Porta Exposta**: 8085 (mapeada para 8085 no host).
* **Variáveis de Ambiente**:
    * `SPRING_PROFILES_ACTIVE`: `prod`.
    * `CLIENTE_SERVICE_URL`: `http://localhost:8085` (note que esta URL aponta para o próprio serviço, o que pode ser um erro de configuração dependendo da intenção).
    * `SERVER_PORT`: `8085`.
* **Dependências**: `rabbitmq` (aguarda o serviço estar saudável).
* **Healthcheck**: Verifica a URL `http://localhost:8085/actuator/health`.

#### 2.12. `rabbitmq`
* **Descrição**: Servidor de mensagens RabbitMQ.
* **Imagem**: `rabbitmq:4.1.0-management`.
* **Portas Expostas**:
    * 5672 (para comunicação AMQP).
    * 15672 (para a interface de gerenciamento).
* **Volumes**:
    * `./rabbitmq/rabbitmq.conf` para configuração do RabbitMQ.
    * `./rabbitmq/definitions.json` para definir usuários, vhosts, permissões e filas.
    * `rabbitmq_data` para persistência de dados do RabbitMQ.
* **Configurações iniciais**: O arquivo `definitions.json` configura um usuário `guest` com senha `guest` (hash `Kv3Um6zhG4CekdNNjHzKX+rVspFozSi2LfPZcYoBJgjbVvtY` usando `sha256`) com privilégios de administrador, no vhost padrão `/`. Também define uma fila durável chamada `PedidoQueue`.
* **Healthcheck**: `rabbitmq-diagnostics check_port_connectivity`.

### 3. Redes

* **`microservices-net`**: Uma rede do tipo `bridge` que conecta todos os microsserviços, permitindo que eles se comuniquem entre si usando seus nomes de serviço (ex: `cliente-app`, `mongo-cliente`).

### 4. Volumes

Os volumes são usados para persistir os dados dos bancos de dados MongoDB e do RabbitMQ, garantindo que os dados não sejam perdidos quando os contêineres são removidos ou recriados.

* `rabbitmq_data`
* `mongodb_cliente_data`
* `mongodb_pedido_data`
* `mongodb_pagamento_data`
* `mongodb_estoque_data`
* `mongodb_produto_data`

### 5. Como Iniciar o Projeto

1.  **Pré-requisitos**:
    * Docker instalado
    * Docker Compose instalado
2.  **Clonar o Repositório**: Certifique-se de que a estrutura de diretórios dos seus microsserviços (ex: `micro-cliente-clean-arch`, `micro-pedido-clean-arch`) esteja no mesmo nível do arquivo `docker-compose.yml`.
3.  **Construir e Iniciar os Serviços**: Navegue até o diretório onde o arquivo `docker-compose.yml` está localizado e execute o seguinte comando:
    ```bash
    docker-compose up --build -d
    ```
    * `--build`: Constrói as imagens dos microsserviços (caso não existam ou tenham sido atualizadas).
    * `-d`: Inicia os contêineres em modo "detached" (segundo plano).
4.  **Verificar o Status dos Serviços**: Para verificar se todos os serviços estão em execução e saudáveis, execute:
    ```bash
    docker-compose ps
    ```
    E para visualizar os logs de todos os serviços:
    ```bash
    docker-compose logs -f
    ```

### 6. Acessando os Serviços

* **`cliente-app`**: `http://localhost:8080`
* **`pedido-app`**: `http://localhost:8081`
* **`pagamento-app`**: `http://localhost:8082`
* **`produto-app`**: `http://localhost:8083`
* **`estoque-app`**: `http://localhost:8084`
* **`receiver-app`**: `http://localhost:8085`
* **RabbitMQ Management UI**: `http://localhost:15672` (Credenciais padrão: `guest`/`guest`)

### 7. Observações e Considerações Futuras

* **Configuração do `receiver-app`**: A variável de ambiente `CLIENTE_SERVICE_URL` no `receiver-app` aponta para `http://localhost:8085`. Se o `receiver-app` precisar se comunicar com o `cliente-app`, essa URL deve ser corrigida para `http://cliente-app:8080` para garantir a comunicação dentro da rede Docker.
* **Segurança do RabbitMQ**: As credenciais padrão (`guest`/`guest`) são usadas para o RabbitMQ. Em um ambiente de produção, é altamente recomendável criar usuários dedicados com permissões mais restritas e senhas fortes.
* **Escalabilidade**: Para ambientes de produção, considere soluções de orquestração mais robustas como Kubernetes para gerenciar e escalar os microsserviços.
* **Monitoramento e Logs**: Implemente soluções de monitoramento e agregação de logs (ex: Prometheus, Grafana, ELK Stack) para ter uma visão completa do estado e desempenho dos microsserviços.
* **Configuração Externa**: Para maior flexibilidade, as variáveis de ambiente e outras configurações sensíveis poderiam ser gerenciadas por meio de um serviço de configuração centralizado (ex: Spring Cloud Config Server, HashiCorp Vault) em vez de serem hardcoded no `docker-compose.yml`.
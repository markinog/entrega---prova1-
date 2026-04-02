# Sistema de E-commerce com Microsserviços

Projeto acadêmico de arquitetura de microsserviços aplicado em um sistema de e-commerce.

##Repositórios dos serviços

1.  `https://github.com/markinog/product-prova1/tree/main` - usuário
2. `https://github.com/markinog/user-prova1/tree/main ` - produto
3. `https://github.com/markinog/inventory-prova1/blob/main/README.md` - estoque
4. `https://github.com/markinog/payment-prova1/blob/main/README.md` - pagamento
5. `https://github.com/markinog/order-prova1/blob/main/README.md` - pedido 


## Arquitetura

O sistema é composto por 5 microsserviços independentes:

1. **Product Service** (porta 8081) - Gerenciamento de produtos
2. **User Service** (porta 8082) - Gerenciamento de usuários
3. **Inventory Service** (porta 8083) - Controle de estoque
4. **Payment Service** (porta 8084) - Processamento de pagamentos
5. **Order Service** (porta 8085) - Orquestração de pedidos

## Tecnologias Utilizadas

- Java 21
- Spring Boot 3.2.0
- MongoDB
- Lombok
- SpringDoc OpenAPI

## Pré-requisitos

- JDK 21
- Maven 3.8+
- MongoDB 6.0+ (rodando na porta padrão 27017)
- MongoDB Compass (opcional, para visualização)

## Configuração do MongoDB

### MongoDB Compass - Criando Conexões

Para cada microsserviço, crie uma conexão no MongoDB Compass:

1. **Product Service Database**
   - URI: `mongodb://localhost:27017/product_db`
   - Database: `product_db`

2. **User Service Database**
   - URI: `mongodb://localhost:27017/user_db`
   - Database: `user_db`

3. **Inventory Service Database**
   - URI: `mongodb://localhost:27017/inventory_db`
   - Database: `inventory_db`

4. **Payment Service Database**
   - URI: `mongodb://localhost:27017/payment_db`
   - Database: `payment_db`

5. **Order Service Database**
   - URI: `mongodb://localhost:27017/order_db`
   - Database: `order_db`

## Como Executar

### 1. Inicie o MongoDB
```bash
# Certifique-se que o MongoDB está rodando na porta 27017
mongod
```

### 2. Execute cada microsserviço em terminais separados

**Terminal 1 - Product Service:**
```bash
cd product-service
mvn spring-boot:run
```

**Terminal 2 - User Service:**
```bash
cd user-service
mvn spring-boot:run
```

**Terminal 3 - Inventory Service:**
```bash
cd inventory-service
mvn spring-boot:run
```

**Terminal 4 - Payment Service:**
```bash
cd payment-service
mvn spring-boot:run
```

**Terminal 5 - Order Service:**
```bash
cd order-service
mvn spring-boot:run
```

## URLs dos Serviços

- Product Service: http://localhost:8081
  - Swagger UI: http://localhost:8081/swagger-ui.html
- User Service: http://localhost:8082
  - Swagger UI: http://localhost:8082/swagger-ui.html
- Inventory Service: http://localhost:8083
  - Swagger UI: http://localhost:8083/swagger-ui.html
- Payment Service: http://localhost:8084
  - Swagger UI: http://localhost:8084/swagger-ui.html
- Order Service: http://localhost:8085
  - Swagger UI: http://localhost:8085/swagger-ui.html

## Fluxo de Criação de Pedido

1. Cliente envia `POST /orders` ao Order Service
2. Order Service consulta `GET /products/{id}` no Product Service
3. Order Service consulta `GET /inventory/{productId}` no Inventory Service
4. Se estoque OK, Order Service cria pedido com status CRIADO
5. Order Service chama `POST /payments` no Payment Service
6. Order Service atualiza status do pedido para PAGO ou CANCELADO
7. Se aprovado, Order Service chama `POST /inventory/{productId}/decrease` no Inventory Service para dar baixa no estoque

## Testando o Sistema

Use a coleção Postman/Insomnia incluída no arquivo `postman_collection.json` na raiz do projeto.

### Exemplo de Fluxo Completo (via cURL):

```bash
# 1. Criar produto
curl -X POST http://localhost:8081/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Notebook Dell",
    "description": "Notebook i7 16GB RAM",
    "price": 3500.00
  }'

# 2. Criar usuário
curl -X POST http://localhost:8082/user \
  -H "Content-Type: application/json" \
  -d '{
    "name": "João Silva",
    "email": "joao@email.com",
    "cpf": "12345678900"
  }'

# 3. Adicionar estoque (use o productId retornado)
curl -X PUT http://localhost:8083/inventory/{productId} \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 10
  }'

# 4. Criar pedido (use userId e productId)
curl -X POST http://localhost:8085/order \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "userId aqui",
    "items": [
      {
        "productId": "productId aqui",
        "quantity": 2
      }
    ]
  }'
```

## Estrutura de Cada Microsserviço

```
service-name/
├── src/
│   ├── main/
│   │   ├── java/com/ecommerce/service/
│   │   │   ├── controller/
│   │   │   ├── model/
│   │   │   ├── repository/
│   │   │   ├── service/
│   │   │   └── ServiceApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
├── pom.xml
└── README.md
```

## Princípios Arquiteturais Implementados

✅ **Baixo Acoplamento**: Serviços se comunicam apenas via APIs REST  
✅ **Alta Coesão**: Cada serviço tem responsabilidade bem definida  
✅ **Isolamento de Dados**: Cada serviço tem seu próprio banco MongoDB  
✅ **Independência**: Cada serviço pode ser executado separadamente  

## Troubleshooting

**Erro de conexão com MongoDB:**
- Verifique se o MongoDB está rodando: `mongod --version`
- Verifique se a porta 27017 está livre

**Porta já em uso:**
- Verifique se outro serviço já está usando a porta
- Altere a porta no `application.properties`

**Serviços não se comunicam:**
- Verifique se todos os serviços estão rodando
- Verifique os logs para erros de conexão

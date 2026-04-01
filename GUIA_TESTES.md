# Guia de Testes - Sistema E-commerce

## Pré-requisitos

1. MongoDB rodando na porta 27017
2. Todos os 5 microsserviços executando simultaneamente
3. Postman ou Insomnia instalado

## Cenário 1: Fluxo Completo de Sucesso

### Passo 1: Criar Produtos

```bash
# Criar Produto 1 - Notebook
curl -X POST http://localhost:8081/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Notebook Dell Inspiron",
    "description": "Notebook i7 16GB RAM 512GB SSD",
    "price": 3500.00
  }'

# Salve o "id" retornado como PRODUCT_ID_1
```

```bash
# Criar Produto 2 - Mouse
curl -X POST http://localhost:8081/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Mouse Logitech MX Master 3",
    "description": "Mouse sem fio ergonômico",
    "price": 450.00
  }'

# Salve o "id" retornado como PRODUCT_ID_2
```

### Passo 2: Criar Usuário

```bash
curl -X POST http://localhost:8082/user \
  -H "Content-Type: application/json" \
  -d '{
    "name": "João Silva",
    "email": "joao.silva@email.com",
    "cpf": "12345678900"
  }'

# Salve o "id" retornado como USER_ID
```

### Passo 3: Adicionar Estoque aos Produtos

```bash
# Adicionar estoque do Notebook (substitua PRODUCT_ID_1)
curl -X PUT http://localhost:8083/inventory/PRODUCT_ID_1 \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 50
  }'
```

```bash
# Adicionar estoque do Mouse (substitua PRODUCT_ID_2)
curl -X PUT http://localhost:8083/inventory/PRODUCT_ID_2 \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 100
  }'
```

### Passo 4: Criar Pedido (Orquestração Completa)

```bash
# Substitua USER_ID, PRODUCT_ID_1 e PRODUCT_ID_2 pelos IDs reais
curl -X POST http://localhost:8085/order \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "USER_ID",
    "items": [
      {
        "productId": "PRODUCT_ID_1",
        "quantity": 2
      },
      {
        "productId": "PRODUCT_ID_2",
        "quantity": 1
      }
    ]
  }'
```

**O que acontece internamente:**

1. Order Service consulta produtos no Product Service ✓
2. Order Service verifica estoque no Inventory Service ✓
3. Order Service cria pedido com status CRIADO ✓
4. Order Service solicita pagamento ao Payment Service ✓
5. Payment Service retorna APROVADO ou RECUSADO (80% aprovação) ✓
6. Order Service atualiza status do pedido ✓
7. Se aprovado, Order Service chama POST /inventory/{productId}/decrease para dar baixa no estoque ✓

### Passo 5: Consultar Pedido Criado

```bash
# Substitua ORDER_ID pelo ID retornado no passo anterior
curl -X GET http://localhost:8085/order/ORDER_ID
```

**Resposta esperada:**

```json
{
  "id": "...",
  "userId": "...",
  "items": [
    {
      "productId": "...",
      "productName": "Notebook Dell Inspiron",
      "quantity": 2,
      "price": 3500.0,
      "subtotal": 7000.0
    },
    {
      "productId": "...",
      "productName": "Mouse Logitech MX Master 3",
      "quantity": 1,
      "price": 450.0,
      "subtotal": 450.0
    }
  ],
  "totalAmount": 7450.0,
  "status": "PAGO", // ou "CANCELADO" se pagamento recusado
  "createdAt": "...",
  "paymentId": "..."
}
```

### Passo 6: Verificar Estoque Atualizado

```bash
curl -X GET http://localhost:8083/inventory/PRODUCT_ID_1
```

**Se pedido foi PAGO, estoque deve estar reduzido:**

- Notebook: 50 - 2 = 48 unidades
- Mouse: 100 - 1 = 99 unidades

---

## Cenário 2: Estoque Insuficiente

```bash
curl -X POST http://localhost:8085/order \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "USER_ID",
    "items": [
      {
        "productId": "PRODUCT_ID_1",
        "quantity": 1000
      }
    ]
  }'
```

**Resultado esperado:** HTTP 400 Bad Request
**Mensagem:** "Estoque insuficiente para produto: Notebook Dell Inspiron"

---

## Verificando nos Logs

Cada microsserviço exibe logs informativos. Exemplo do Order Service:

```
Iniciando criação de pedido para usuário: 675abc123...
Produto encontrado: Notebook Dell Inspiron - R$ 3500.00
Estoque disponível: 50 unidades
Pedido criado com ID: 675def456... - Total: R$ 7450.00
Pagamento processado: 675ghi789... - Status: APROVADO
Pagamento aprovado! Atualizando estoque...
Estoque atualizado para produto 675abc123...: 48 unidades
```

---

## Verificando no MongoDB Compass

Conecte-se aos bancos de dados:

1. **product_db** - Produtos cadastrados
2. **user_db** - Usuários cadastrados
3. **inventory_db** - Controle de estoque
4. **payment_db** - Histórico de pagamentos
5. **order_db** - Pedidos com status

---

## Swagger UI

Acesse a documentação interativa de cada serviço:

- Product Service: http://localhost:8081/swagger-ui.html
- User Service: http://localhost:8082/swagger-ui.html
- Inventory Service: http://localhost:8083/swagger-ui.html
- Payment Service: http://localhost:8084/swagger-ui.html
- Order Service: http://localhost:8085/swagger-ui.html

---

## Troubleshooting

**Erro: Connection refused**

- Verifique se todos os serviços estão rodando
- Confirme as portas: 8081, 8082, 8083, 8084, 8085

**Erro: MongoDB connection**

- Verifique se o MongoDB está rodando: `mongod --version`
- Porta padrão: 27017

**Pagamento sempre recusado**

- É aleatório (80% aprovação, 20% recusa)
- Tente criar outro pedido

**Estoque não atualiza**

- Verifique se o pagamento foi aprovado
- Somente pedidos PAGOS atualizam o estoque

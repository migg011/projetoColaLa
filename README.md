# Sem Viagem Perdida

Resolve a "viagem perdida": ir até um comércio de bairro e descobrir, só ao chegar, que está fechado ou sem o item que você precisa.

**Diferencial:** ferramentas como Google Maps já mostram se o estabelecimento existe e seu horário declarado — mas nenhuma confirma se tem um produto específico disponível agora, para comércio pequeno. É esse gap que o projeto ocupa.

**Como funciona:** usuários confirmam por voto se a loja estava aberta e tinha o item; o lojista pode opcionalmente reforçar isso com um check-in de baixíssima fricção; o sistema calcula, com base nesses sinais recentes, se a informação da loja é confiável (**Ativa**) ou arriscada (**Desatualizada**).

---

## Stack
- **Frontend:** Angular
- **Backend:** Python (FastAPI)
- **Banco:** PostgreSQL

---

## Escopo Core (precisa funcionar, sem exceção)

### 1. Dados básicos
Tabela de lojas e produtos, cadastrados via seed (sem tela de admin).

### 2. Fluxo do usuário
- Busca de item → mostra quais lojas costumam ter
- Voto simples: "estava aberta?" / "tinha o item?" (sim ou não)

### 3. Fluxo do lojista (opcional, baixa fricção)
Um botão único: "confirmar que estou aberto hoje". Esse check-in conta **exatamente como um voto positivo comum** — sem peso especial, sem hierarquia de confiança. Serve pra loja não ficar sem informação nenhuma antes do primeiro voto de usuário.

### 4. Motor de confiança

```
1. Buscar todos os votos + check-ins da loja com timestamp <= 12h atrás
2. Se total de registros == 0:
     status = "Desatualizada"
3. Senão:
     positivos = contagem de votos "sim" + contagem de check-ins
     negativos = contagem de votos "não"

     Se positivos > negativos:
         status = "Ativa"
     Senão (negativos >= positivos, incluindo empate):
         status = "Desatualizada"
```

Contagem simples de maioria dentro de uma janela de 12 horas — sem inferência, sem IA, sem ponderação diferenciada entre fontes. Simplicidade é intencional: garante resultado auditável e explicável.

**Fora do escopo deliberadamente:** ponderação por peso de fonte, decaimento exponencial por tempo, threshold de confiança contínua (%).

**"Tempo real" = polling simples, não WebSocket.** Recarregar a busca a cada X segundos (ou F5) resolve 100% da demo.

### 5. Demonstração
Script de seed com lojas, produtos e histórico de votos/check-ins simulados, pra demo funcionar sem depender de uso real. Mínimo: pelo menos uma loja "Ativa" e uma "Desatualizada" no seed.

---

Design do Frontend
- Design visual aceitavel
- Autenticação real (fica mockada — usuário fixo fake)
- Responsividade mobile
- Telas/fluxos extras além do essencial
- Tratamento de erro refinado
- Filtros avançados, busca por categoria

---

## O que o projeto explicitamente não é
Sem comparação de preço, delivery, checkout/carrinho, controle de estoque detalhado, catálogo ilimitado.

---

## Decisões técnicas fechadas
- **Janela de recência:** 12 horas
- **Zero dado (sem voto, sem check-in):** status padrão é Desatualizada
- **Empate de votos:** vai pra Desatualizada
- **Timezone:** tudo em UTC
- **Relação loja-produto:** booleano simples (vende ou não vende), sem quantidade/estoque
- **Falha durante voto/check-in:** sem tratamento especial nesta versão (risco assumido)
- **Voto em massa/manipulação:** sem limite de frequência nesta versão (aceitável pra demo controlada)
- **Critério do Top 10:** itens mais citados nas conversas informais de validação
- **Piso mínimo de voto:** 1 voto/check-in já conta, sem mínimo artificial
- **Como a loja sabe que tem o produto:** associação cadastrada manualmente no banco, sem inferência automática nem IA

## Limitações conhecidas (assumidas conscientemente)
- Cadastro de loja é manual/curado — não escala, mas é intencional pro controle de qualidade do MVP
- Validação da dor foi feita de forma informal — suficiente pra MVP acadêmico
- Sistema de votos pode ter viés de amostragem (quem desiste de ir por causa do status "desatualizado" não gera voto novo, criando possível espiral negativa)

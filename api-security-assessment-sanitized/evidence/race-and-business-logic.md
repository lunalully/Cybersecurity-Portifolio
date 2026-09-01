# Evidência — Race Conditions & Lógica de Negócio

## Idempotência de favorito paralelo (PASS)

```bash
# 10 × POST /store/{id}/favorite em paralelo → um único favorito, sem duplicatas
# GET /favorites → 1 entrada
# 10 × DELETE em paralelo → 1 sucesso, 9 × 404 favorite not found (sem corrupção)
```

## Idempotência de criação de pedido paralela (PASS com nota)

```bash
# 10 × POST /store/{id}/order {menu_id} em paralelo
# → 10 × 200 com o MESMO id de pedido (uniq -c → 10)
```

> Um único carrinho `CREATED` ativo por loja/usuário evita duplicatas — mas esse invariante é
> implícito e não documentado (achado F-09).

## Duplicação de CNPJ — race prova ausência de lock (FALHA — achado F-02)

```bash
# 10 × POST /store com o mesmo CNPJ em paralelo
# → 10 × 200, ids de loja distintos
```

## Volume de criação de loja (FALHA — achado F-06)

```bash
# 32 criações no total em < 60s (2 iniciais + 10 race + 20 stress)
# → zero respostas 429
```

## Lógica de cupom — valor mínimo de pedido aplicado (PASS)

```bash
# 5 × POST /order/{id}/apply-coupon {"code":"WELCOME10"} em pedido CREATED vazio
# → 5 × 400 order does not meet minimum order value of 3000
```

## Máquina de estados do pedido (majoritariamente bloqueada, precisa staging para walk completo)

```bash
PATCH /order/{id}/place            sem address_id → 400 missing address_id
PATCH /order/{id}/cancel           customer em CREATED → 400 invalid_argument
PATCH /order/{id}/accept           store em pedido vazio → 400 (erro de parse)
PATCH /order/{id}/deliver          customer → 403 (bloqueado)
```

## Sanidade de carga após testes agressivos

```bash
GET /health  (5×) → 200 em 0.598 / 0.613 / 0.602 / 0.602 / 0.609s
```

> O serviço permaneceu disponível através de 30×5k payloads, 20× image bombs e 32 criações de
> loja — sem degradação prolongada, sem crash (sinal positivo de resiliência).

## Não testado neste ciclo (requer staging autorizado + reset)

- Race de cupom (`apply-coupon` × 100 com `WELCOME10`) em pedido `PLACED` com itens
- Falsificação de webhook de pagamento com sandbox Asaas
- Races de seleção de `variant`/`addon`
- Rotação/replay de refresh token (requer `device_id` real)
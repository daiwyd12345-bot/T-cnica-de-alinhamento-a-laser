# Método RRA — Relógio Reverso Assistido

**Alinhamento de eixos com "inteligência de laser" usando relógio comparador**

Versão 1.0 — documento de fechamento da ideia, base para o prompt do App.

---

## 1. Conceito

A precisão de um sistema de alinhamento a laser não vem do feixe de luz em si. Ela vem de quatro coisas:

1. **Geometria de medição reversa**: cada cabeça mede o outro eixo, em dois planos separados.
2. **Rotação conjunta dos dois eixos**: elimina o erro de batimento (runout) dos eixos e do acoplamento.
3. **Ângulo conhecido em cada leitura**: inclinômetro, o que permite medir em arco parcial.
4. **Cálculo automático e acompanhamento ao vivo**: correção nos pés, "live move", tolerância e relatório.

O **Método RRA** reproduz esses quatro elementos com **dois relógios comparadores** (o sensor) e um **smartphone** (inclinômetro, cálculo, guia passo a passo e relatório). O resultado é um fluxo de trabalho equivalente ao de um sistema a laser, com custo de relógio comparador.

O laser entra como ferramenta opcional de **pré-alinhamento grosseiro**: um nível laser de linha cruzada ou uma caneta laser em base magnética. Ele leva o conjunto para dentro da faixa útil dos relógios (≤ 0,5 mm), e a partir daí os relógios fazem a medição fina.

> Método base: **relógio reverso** (reverse dial indicator). Ele é o equivalente mecânico direto do laser de dois sensores, é o menos sensível a folga axial e funciona em acoplamentos curtos e longos.

---

## 2. Equipamento

| Item | Especificação mínima | Observação |
|---|---|---|
| 2 relógios comparadores | Resolução 0,01 mm (ou 0,001") e curso ≥ 5 mm | Digitais são preferíveis (v2: leitura Bluetooth) |
| 2 suportes (brackets) com corrente ou base magnética | Rígidos e leves | A flecha (sag) deles será medida |
| Tubo ou barra rígida para medir a flecha | Diâmetro semelhante ao do eixo | |
| Smartphone com o App | Acelerômetro (todos têm) | Preso por ímã ou suporte no bracket, servindo de inclinômetro |
| Trena ou escala | 1 mm | Para as distâncias |
| Calços calibrados (shims) | Aço inox, espessuras variadas | |
| Relógio com base magnética | — | Para medir pé manco e tensão de tubulação |
| *(Opcional)* Nível laser de linha ou caneta laser | — | Pré-alinhamento |

---

## 3. Convenções (obrigatórias para o App e para o técnico)

- **Máquina FIXA (S)**: a que não será movida (normalmente a bomba). **Máquina MÓVEL (M)**: a que recebe calços e é deslocada (normalmente o motor).
- **Posição do observador**: atrás da máquina móvel, olhando para a máquina fixa. Nessa vista, **12h = topo, 3h = direita, 6h = baixo, 9h = esquerda**.
- **Eixo de referência**: origem (0) no **centro do acoplamento**, positivo no sentido da máquina móvel. Todas as distâncias são medidas a partir daí.
- **Relógio "M"**: montado no eixo **fixo**, apalpando o eixo **móvel**, no plano `xM` (> 0).
- **Relógio "S"**: montado no eixo **móvel**, apalpando o eixo **fixo**, no plano `xS` (< 0).
- **Sinal da leitura**: ponta (haste) entrando = **positivo**.
- Os dois relógios são montados **na mesma posição angular**: quando um está às 12h, o outro também está.
- **Resultado vertical**: positivo = móvel **acima** da fixa. **Resultado horizontal**: positivo = móvel deslocada **para 3h**.
- **Correção**: calço positivo = **adicionar**, negativo = **remover**. Movimento horizontal positivo = empurrar **para 3h**.

Distâncias que o técnico informa:

```
   MÁQUINA FIXA (S)                 MÁQUINA MÓVEL (M)
  ─────────────────[ S ]──╫──[ M ]─────────[F1]──────────[F2]
                    xS    0    xM           F1            F2
                    │◄─────D────►│
  xS  = posição do plano do relógio S (negativo)
  xM  = posição do plano do relógio M (positivo)
  F1  = pé dianteiro da móvel   F2 = pé traseiro da móvel
  D   = xM − xS (distância entre os planos dos relógios)
  Dc  = diâmetro do acoplamento (para expressar a angularidade como "gap")
```

---

## 4. Procedimento em fases

O App deve conduzir o técnico fase a fase. Nenhuma fase é pulada sem uma justificativa registrada.

### Fase 0 — Segurança e dados

- Bloqueio e etiquetagem (LOTO) da máquina, com registro de quem bloqueou.
- Cadastro do ativo (TAG, RPM, tipo de acoplamento, fabricante) e da tolerância a usar (Seção 6).
- Dados de crescimento térmico, se houver (Fase 7).

### Fase 1 — Inspeção prévia

- Fundação, chumbadores, calços velhos (máximo de 4 calços por pé; trocar pacotes enferrujados ou amassados).
- Folga axial (end float) dos eixos: anotar o valor.
- **Tensão de tubulação**: relógio na móvel junto ao acoplamento, soltar os chumbadores. Um movimento > 0,05 mm indica tensão de tubulação, que precisa ser corrigida antes de alinhar.

### Fase 2 — Batimento (runout)

- Relógio no eixo e no cubo do acoplamento, girando 360°.
- Limite: **≤ 0,05 mm TIR**. Se estiver acima, corrigir (eixo empenado, cubo excêntrico) antes de prosseguir.

### Fase 3 — Pé manco (soft foot)

- Relógio com base magnética sobre cada pé. Soltar um chumbador por vez e ler.
- **> 0,05 mm = pé manco.** Classificar o tipo (paralelo, angular, mola ou induzido) e corrigir com calço ou retrabalho.
- Repetir até todos os pés ficarem ≤ 0,05 mm. *O App bloqueia a Fase 5 se algum pé estiver reprovado sem justificativa.*

### Fase 4 — Flecha do suporte (bracket sag)

- Montar o bracket **exatamente** com o comprimento usado na máquina, sobre um tubo rígido.
- Zerar o relógio às 12h, girar até 6h e ler. Esse valor é a **flecha (Sag)**, sempre **negativa** (ex.: −0,04 mm).
- Medir a flecha de cada relógio, se as montagens forem diferentes.

### Fase 5 — Pré-alinhamento (laser opcional)

- Com régua ou nível laser de linha nas geratrizes dos cubos, levar o conjunto a ≤ 0,5 mm de desalinhamento.
- Objetivo: manter os relógios na faixa linear e reduzir o número de iterações.

### Fase 6 — Medição fina (relógio reverso)

1. Montar os dois brackets com os relógios na mesma posição angular e o smartphone preso a um deles.
2. Pré-carregar os relógios no meio do curso.
3. **Girar os dois eixos juntos**, sempre no sentido de rotação de operação. Nunca girar um eixo só.
4. Dois modos de medição:
   - **Modo 4 posições**: 12h → 3h → 6h → 9h. Zerar às 12h e registrar as 4 leituras de cada relógio. O App confirma pelo inclinômetro que a posição está correta (±3°).
   - **Modo multiponto (arco restrito)**: quando não é possível girar 360° (ex.: interferência com proteção ou base). Registrar **≥ 3 pontos** cobrindo um arco de **≥ 90°** (recomendado; 60° é o mínimo absoluto). O ângulo de cada ponto é lido automaticamente pelo inclinômetro.
5. **Verificação de validade (modo 4 posições)**: `R12 + R6 = R3 + R9` (com zero às 12h: `R6 = R3 + R9`). Diferença tolerada: ≤ 0,02 mm ou ≤ 10% do TIR, o que for maior. Se falhar, a causa provável é folga no bracket, relógio travando, leitura errada ou eixo que não acompanhou a rotação. Nesse caso, refazer.
6. **Repetibilidade**: medir **duas vezes**. As duas medições devem concordar em ≤ 0,02 mm, senão investigar.
7. O App calcula o desalinhamento no acoplamento e a correção em cada pé (Seção 5).

### Fase 7 — Metas (alvos frios)

- Sem crescimento térmico: alvo = 0.
- Com crescimento térmico: `ΔL = α · L · ΔT`, com α(aço) = 11,5 × 10⁻⁶ /°C, L = altura da base até a linha de centro do eixo e ΔT = temperatura de operação menos a temperatura ambiente da carcaça.
- Alvo frio vertical da móvel = −(ΔL_móvel − ΔL_fixa). Se a móvel cresce mais, ela é deixada mais baixa a frio.
- Também é possível informar alvos dados pelo fabricante (offset e angularidade a frio).

### Fase 8 — Correção

Ordem obrigatória:

1. **Vertical primeiro (calços)**. Soltar os chumbadores, aplicar os calços calculados em F1 e F2 e reapertar em sequência cruzada e com o torque especificado.
2. **Remedir.** Se a vertical estiver em tolerância, seguir.
3. **Horizontal com "live move"**:
   - Girar os eixos até os relógios ficarem às **3h**. Zerar os dois relógios nessa posição.
   - O App mostra o **alvo de cada relógio**: `alvo_M = −H_M/2` e `alvo_S = −H_S/2` (meia leitura horizontal, com sinal invertido).
   - Deslocar a móvel com parafusos de empurrar (jack bolts) até os dois relógios mostrarem seus alvos ao mesmo tempo. Na v2, os relógios digitais via Bluetooth mostram isso ao vivo na tela.
   - Reapertar e conferir se a leitura não fugiu durante o aperto.
4. **Medição final completa**, que precisa passar em validade, repetibilidade e tolerância.

### Fase 9 — Relatório

- Valores "como encontrado" (as found) e "como deixado" (as left): offset e angularidade, vertical e horizontal, em relação à tolerância.
- Pé manco, batimento, flecha, calços aplicados, alvos térmicos, técnico, data, fotos e assinatura.
- Exportação em PDF.

---

## 5. Matemática (especificação de cálculo para o App)

### 5.1 Modelo de leitura (válido para qualquer ângulo)

Para um relógio apalpando um eixo deslocado de `(z, y)` em relação ao eixo onde o relógio está montado, com θ medido a partir das 12h no sentido horário (vista do observador):

```
r(θ) = k + (y − Sag/2)·cos θ + z·sin θ
```

- `k`: constante (zero arbitrário do relógio).
- `y`: deslocamento vertical do eixo apalpado (+ = para cima).
- `z`: deslocamento horizontal do eixo apalpado (+ = para 3h).
- O termo `Sag/2` modela a flecha do bracket (Sag < 0, medida na Fase 4).

**Modo multiponto**: com N ≥ 3 pares (θᵢ, rᵢ), resolver por **mínimos quadrados** `r = k + C·cos θ + S·sin θ`, e então `y = C + Sag/2` e `z = S`. O resíduo do ajuste serve como indicador de qualidade da medição.

**Modo 4 posições** (forma fechada, zero às 12h):

```
V = R6 − Sag          (vertical corrigida pela flecha)
H = R3 − R9           (horizontal; a flecha se cancela)
y = −V/2              z = +H/2
```

### 5.2 Posição da linha de centro da móvel

- Relógio **M** (no fixo, lendo o móvel em `xM`): mede o próprio móvel, logo `pos_M(xM) = (y_M, z_M)`.
- Relógio **S** (no móvel, lendo o fixo em `xS`): mede o fixo visto pelo móvel, logo `pos_M(xS) = (−y_S, −z_S)`.

Para cada plano (vertical e horizontal), com `pM = posição em xM` e `pS = posição em xS`:

```
b = (pM − pS) / D                 inclinação (angularidade), em mm/mm
a = pS − b · xS                   offset no centro do acoplamento, em mm
p(x) = a + b · x                  posição da móvel em qualquer x
```

Explícito no modo 4 posições:

```
Vertical:   pM = −V_M/2     pS = +V_S/2
Horizontal: pM = +H_M/2     pS = −H_S/2
```

### 5.3 Correção nos pés (com metas)

```
Correção(F) = (a_alvo + b_alvo · F) − (a + b · F)
```

Sem metas, `Correção(F) = −p(F)`. Na vertical, positivo = adicionar calço. Na horizontal, positivo = mover para 3h.

### 5.4 Formas de exibir

- **Offset** no centro do acoplamento: `a` (mm).
- **Angularidade**: `b × 100` (mm/100 mm) e também como gap no acoplamento: `b × Dc` (mm).
- Gráfico das duas linhas de centro (fixa reta e móvel inclinada), com os pés marcados. É o "visual de laser".

### 5.5 Live move (horizontal, relógios às 3h e zerados)

```
alvo_M = −H_M/2        alvo_S = −H_S/2
```

Com metas térmicas ou do fabricante, somar a contribuição do alvo em cada plano.

---

## 6. Tolerâncias de referência (acoplamentos flexíveis curtos)

Tabela típica da indústria. **Deve ser configurável no App**, e a recomendação do fabricante do acoplamento ou da máquina sempre prevalece.

| RPM | Offset excelente (mm) | Offset aceitável (mm) | Angularidade excelente (mm/100 mm) | Angularidade aceitável (mm/100 mm) |
|---|---|---|---|---|
| 0 – 1000 | 0,07 | 0,13 | 0,06 | 0,10 |
| 1000 – 2000 | 0,05 | 0,10 | 0,05 | 0,08 |
| 2000 – 3000 | 0,03 | 0,07 | 0,04 | 0,07 |
| 3000 – 4000 | 0,02 | 0,05 | 0,03 | 0,06 |
| 4000 – 6000 | 0,01 | 0,03 | 0,02 | 0,05 |

Status por plano: **Excelente** (verde), **Aceitável** (amarelo), **Fora** (vermelho).

---

## 7. Exemplo resolvido (caso de teste do App)

Dados: `xS = −100`, `xM = +100`, `D = 200`, `F1 = 300`, `F2 = 800` mm. `Sag = −0,04` mm.

Leituras brutas (zero às 12h):

| Relógio | 12h | 3h | 6h | 9h | Validade (R3 + R9 − R6) |
|---|---|---|---|---|---|
| M | 0,00 | −0,09 | −0,28 | −0,19 | 0,00 ✔ |
| S | 0,00 | +0,09 | +0,12 | +0,03 | 0,00 ✔ |

**Vertical**:
- `V_M = −0,28 − (−0,04) = −0,24`, então `pM = +0,12`.
- `V_S = 0,12 + 0,04 = +0,16`, então `pS = +0,08`.
- `b = (0,12 − 0,08)/200 = 0,0002`, ou seja, **0,02 mm/100 mm**.
- `a = 0,08 + 0,0002·100` = **+0,10 mm** (móvel alta).
- Correção: **F1 = −0,16 mm** e **F2 = −0,26 mm** (remover calços).

**Horizontal**:
- `H_M = −0,09 − (−0,19) = +0,10`, então `pM = +0,05`.
- `H_S = 0,09 − 0,03 = +0,06`, então `pS = −0,03`.
- `b = 0,08/200 = 0,0004`, ou seja, **0,04 mm/100 mm**.
- `a = −0,03 + 0,04` = **+0,01 mm**.
- Correção: **F1 = −0,13 mm** e **F2 = −0,33 mm** (empurrar para 9h).
- Live move a 3h: **alvo_M = −0,05** e **alvo_S = −0,03**.

Verificação: o mesmo caso medido em **arco restrito** (30°, 70°, 110° e 150°) e resolvido por mínimos quadrados devolve exatamente `a` e `b` acima. Esse caso deve virar teste automatizado no App.

---

## 8. Erros que o método bloqueia

| Erro comum | Como o RRA evita |
|---|---|
| Girar só um eixo | Instrução obrigatória; a falha de validade e de repetibilidade denuncia |
| Ignorar a flecha do bracket | Fase 4 obrigatória antes da medição |
| Sinal trocado (calçar em vez de tirar) | Convenções fixas, gráfico visual e confirmação "a máquina vai DESCER x mm em F1" |
| Posição angular imprecisa | Inclinômetro do celular valida cada ponto |
| Pé manco mascarando o resultado | Fase 3 bloqueia a medição |
| Corrigir horizontal antes da vertical | Ordem imposta pelo fluxo |
| Leitura mal anotada | Regra de validade `R12 + R6 = R3 + R9` |
| Resultado não repetível | Duas medições obrigatórias com concordância ≤ 0,02 mm |

---

## 9. Escopo proposto do App

**MVP**

- Cadastro de ativo e de tolerância.
- Wizard das Fases 0 a 9.
- Entrada manual das leituras com inclinômetro do celular (modo 4 posições e multiponto).
- Cálculo, gráfico, correção nos pés, live move "calculado" e relatório PDF.
- Unidades mm e mils.
- Funcionamento 100% offline.

**v2**

- Relógios digitais via Bluetooth (live move em tempo real).
- Pé limitante: escolher mover a fixa, ambas, ou respeitar limites de calço e de folga do chumbador.
- Conjuntos com 3 ou mais máquinas (trem de máquinas).
- Método face-borda (rim-face) como alternativa para eixos curtos sem espaço para o reverso.
- Histórico por ativo e sincronização em nuvem.

---

## 10. Decisões em aberto (fechar antes do prompt do programador)

1. Plataforma: Android apenas, Android + iOS (Flutter ou React Native) ou PWA web.
2. Unidade padrão: mm ou mils, e se o idioma é só PT-BR.
3. Relógio digital Bluetooth já no MVP? Se sim, qual modelo e protocolo.
4. Armazenamento: só local, ou com nuvem e multiusuário.
5. Identidade visual e logotipo do relatório PDF.

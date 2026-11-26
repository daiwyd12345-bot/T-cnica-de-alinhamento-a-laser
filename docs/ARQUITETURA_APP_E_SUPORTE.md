# Arquitetura do App + Cabeça Bluetooth + Suporte dos Relógios

Complemento do [Método RRA](METODO_ALINHAMENTO.md). Versão 1.0, rascunho para fechamento.

---

## 1. Decisão de hardware: relógio barato + "cabeça" Bluetooth própria

Relógios com Bluetooth embutido custam a partir de ≈ R$ 1.270 cada (Werka 4209-012W), e o protocolo deles não é público. Relógios digitais **com saída de dados** custam bem menos (ex.: Vonder RD-010, 0,01 mm, conector mini-USB, entre ≈ R$ 280 e 430 conforme a oferta).

Esses relógios baratos costumam usar um protocolo serial simples, **clock + data**:

- Sinais de ≈ 1,5 V.
- Quadro de 24 bits, enviado continuamente.
- Bits 0–19 = valor, bit 20 = sinal, bit 23 = unidade (mm ou pol).

Esse protocolo é bem documentado por quem já fez interface com Arduino.

**Proposta:** construir uma **Cabeça RRA**, uma pequena caixa com ESP32 presa no suporte. Ela:

1. lê os **dois relógios** por cabo;
2. lê o **ângulo** de rotação com um acelerômetro;
3. envia tudo junto, **sincronizado**, ao celular por **Bluetooth Low Energy (BLE)**, com um protocolo **nosso e documentado**.

Vantagens:

- **Preço:** é a opção mais barata.
- **Independência de marca:** trocar de relógio não muda o App.
- **Sincronização:** as duas leituras e o ângulo saem no mesmo pacote, com um único carimbo de tempo.
- **Ângulo:** vem do acelerômetro preso ao suporte, e não do celular.

> **Risco a eliminar primeiro (prova de conceito):** confirmar o protocolo do relógio escolhido com um analisador lógico barato ou osciloscópio, antes de comprar o segundo relógio. O conector "mini-USB" desses relógios **não é USB de verdade**: é apenas o formato do plugue. Alguns modelos só transmitem quando um pino de requisição é acionado.

---

## 2. Cabeça RRA (eletrônica)

```
 Relógio M ──cabo 4 vias──┐
                          │   ┌──────────── CABEÇA RRA ────────────┐
 Relógio S ──cabo 4 vias──┼──►│ Conversor de nível 1,5 V → 3,3 V   │
                          │   │ (comparador ou transistor por linha)│
                          │   │            │                        │
                          │   │         ESP32 (BLE) ◄── Acelerômetro│
                          │   │            │          (3 eixos, I²C)│
                          │   │  Bateria 18650 + carregador + chave │
                          │   └────────────┼────────────────────────┘
                          │                ▼  BLE
                          │           Smartphone (App)
```

| Bloco | Sugestão | Observação |
|---|---|---|
| Microcontrolador | ESP32 (WROOM ou C3) | BLE nativo e barato |
| Entrada dos relógios | 2 linhas por relógio (clock, data) com conversão de nível | Os sinais de ~1,5 V não são lidos diretamente pelo ESP32 |
| Alimentação dos relógios | Bateria própria de cada relógio (fase 1) | Mais simples e mantém os relógios isolados; na fase 2, regulador de 1,5 V |
| Ângulo | Acelerômetro 3 eixos (ADXL345, LIS3DH ou MPU-6050) | θ = atan2 da gravidade no plano perpendicular ao eixo; precisão ~0,5–1° (o método exige ±3°) |
| Energia | 18650 + módulo carregador + chave liga/desliga | |
| Caixa | Impressa em 3D (PETG) | Plástico só na caixa, nunca na estrutura do suporte |

O cabo do relógio do outro suporte atravessa o acoplamento. Isso não é problema, porque **os dois eixos giram juntos** e o cabo gira com eles. Deixe uma folga e prenda o cabo com abraçadeira.

### 2.1 Firmware

- Leitura dos quadros de 24 bits por interrupção na borda do clock, com timeout para ressincronizar.
- Conversão para **micrômetros inteiros** (se o relógio estiver em polegadas, o firmware converte).
- Filtro do ângulo (média móvel de ~200 ms) e calibração de offset do zero às 12h.
- Envio por BLE a ~10 Hz.

### 2.2 Protocolo BLE (GATT) da Cabeça RRA

| Característica | Tipo | Conteúdo |
|---|---|---|
| `Medida` | Notify, 16 bytes | `seq` u16, `t_ms` u32, `rM` i32 (µm), `rS` i32 (µm), `ang` i16 (0,1°), `flags` u8 (validM, validS, polegada, bateria baixa) |
| `Comando` | Write | `ZERAR_M`, `ZERAR_S`, `ZERAR_ANGULO`, `TAXA(hz)` |
| `Info` | Read | Versão de firmware, tensão da bateria, número de série |

UUIDs próprios, definidos no início do projeto. Este documento é a especificação que o programador e o firmware seguem.

---

## 3. Arquitetura do App

**Recomendação:** **Flutter** (Android e iOS com um só código e BLE maduro), funcionando 100% offline.

```
┌──────────────────────── UI (telas) ───────────────────────────┐
│ Ativos · Wizard Fases 0–9 · Medição · Resultado · Live Move ·  │
│ Relatório · Dispositivos                                       │
├──────────────────── Aplicação ────────────────────────────────┤
│ SessaoAlinhamento (máquina de estados das fases)               │
│ CapturaAutomatica · ValidacaoMedicao · GeradorRelatorio        │
├──────────────────── Domínio (Dart puro, sem dependências) ────┤
│ CalculadoraRRA (4 posições + mínimos quadrados) · Flecha ·     │
│ Tolerancias · MetasTermicas · CorrecaoPes · LiveMoveAlvos      │
│ Modelos: Ativo, Geometria, Leitura, Resultado, Correção        │
├──────────────────── Infraestrutura ───────────────────────────┤
│ FonteDeMedicao (interface) ─┬─ FonteCabecaRRA (BLE próprio)    │
│                             ├─ FonteManual (digitação +        │
│                             │   inclinômetro do celular)       │
│                             ├─ FonteBleFabricante (futuro:     │
│                             │   Werka etc., se houver protocolo)│
│                             └─ FonteSimulada (testes e demo)   │
│ Banco local (SQLite) · PDF · Sensores do celular · Permissões  │
└────────────────────────────────────────────────────────────────┘
```

### 3.1 Abstração-chave: `FonteDeMedicao`

Todo o App enxerga apenas um fluxo de amostras:

```
Amostra { t_ms, rM_um, rS_um, angulo_graus, validoM, validoS }
```

Isso permite:

- usar o App **hoje**, com relógio analógico e digitação (`FonteManual`);
- desenvolver e testar **sem hardware** (`FonteSimulada`, que gera leituras a partir de um desalinhamento conhecido, como no exemplo da Seção 7 do método);
- trocar ou acrescentar relógios sem mexer em cálculo nem em telas.

### 3.2 Captura automática ("modo laser")

O técnico só gira os eixos, e o App captura sozinho quando:

1. o ângulo está a ±3° da posição-alvo (modo 4 posições) ou a ≥ 20° do último ponto (modo multiponto); e
2. as leituras estão estáveis (desvio-padrão < 2 µm em 1 s); e
3. os dois relógios estão válidos.

A cada captura o App dá vibração e bipe e marca o ponto num gráfico circular. Ao final, aplica a validade (`R12 + R6 = R3 + R9`) e a repetibilidade, e mostra o resultado.

### 3.3 Telas

1. **Ativos:** lista, cadastro (TAG, RPM, acoplamento, tolerância) e histórico.
2. **Dispositivos:** procurar e parear a Cabeça RRA, bateria, zerar e teste de leitura ao vivo.
3. **Wizard:** uma tela por fase (segurança, inspeção, batimento, pé manco, flecha, geometria com desenho, medição, metas, correção, relatório), com bloqueios conforme o método.
4. **Medição:** mostrador de ângulo, as duas leituras grandes e os pontos capturados.
5. **Resultado:** offset e angularidade (V e H) com status de cor, gráfico das linhas de centro e calços e movimentos em F1 e F2 escritos por extenso ("REMOVER 0,16 mm no pé dianteiro").
6. **Live Move:** os dois relógios em tamanho grande, com o alvo de cada um, barra de aproximação e setas para 3h ou 9h.
7. **Relatório:** PDF "como encontrado / como deixado", com fotos e assinatura.

### 3.4 Pacotes Flutter sugeridos

`flutter_blue_plus` (BLE), `sensors_plus` (acelerômetro do celular), `drift` ou `sqflite` (banco), `pdf` + `printing` (relatório), `permission_handler`.

As permissões Bluetooth do Android 12+ (`BLUETOOTH_SCAN` e `BLUETOOTH_CONNECT`) e do iOS precisam ser tratadas na primeira execução.

### 3.5 Testes obrigatórios

- O domínio tem testes unitários com o exemplo resolvido do método: 4 posições e multiponto devem devolver `a` e `b` exatos.
- Decodificador do pacote BLE (bytes → `Amostra`), incluindo sinal negativo e flags.
- `FonteSimulada` com ruído, para testar a captura automática.

---

## 4. Suporte dos relógios (o seu projeto mecânico)

### 4.1 Por que são **dois** suportes, e não um só

No método reverso, cada relógio precisa estar **rigidamente preso a um eixo** e girar com ele:

- o relógio M fica preso ao eixo fixo S e lê o eixo móvel;
- o relógio S fica preso ao eixo móvel M e lê o eixo fixo.

Uma peça única presa nos dois eixos ao mesmo tempo amarraria os eixos entre si e invalidaria a medição.

O "suporte" é portanto um **kit de dois suportes iguais e modulares**. A Cabeça RRA vai em um deles.

### 4.2 Desenho de conceito (vista lateral, relógios às 12h)

**Suporte A**: preso no eixo FIXO, leva o relógio M até o eixo MÓVEL.

```
                garra em cruz        barra axial Ø10–12 (aço)
                      ┌────────────────────────────●  ← Relógio M
          ┌──────────┐│ haste radial               │    (ponta para baixo,
          │CABEÇA RRA││                            ▼     no eixo móvel, plano xM)
          └────┬─────┘│
            ┌──┴──────┴┐
            │ V + CORR.│
 ═══════════╧══════════╧══════════╦╦══════════════════════════════
      EIXO FIXO (S)          ACOPLAMENTO          EIXO MÓVEL (M)
```

**Suporte B**: preso no eixo MÓVEL, leva o relógio S até o eixo FIXO.

```
      Relógio S  →  ●────────────────────────────┐ garra em cruz
  (ponta para baixo,│      barra axial Ø10–12    │
   no eixo fixo,    ▼                            │ haste radial
   plano xS)                                  ┌──┴───────┐
                                              │ V + CORR.│
 ═════════════════════════════════╦╦══════════╧══════════╧═══════
      EIXO FIXO (S)          ACOPLAMENTO          EIXO MÓVEL (M)
```

Os dois suportes são montados ao mesmo tempo. As barras cruzam o acoplamento lado a lado, com uma pequena diferença de altura nas hastes radiais para uma não tocar a outra. Os dois relógios ficam na mesma posição angular.

### 4.3 Componentes de cada suporte

| Peça | Especificação sugerida | Por quê |
|---|---|---|
| **Bloco em V** | Alumínio 6061, V de 90°, ~40 mm de largura × 60 mm de comprimento, rasgo para a corrente | Autocentra no eixo; dois apoios de linha garantem repetibilidade |
| **Corrente + tensor** | Corrente de rolos (passo 3/8") + parafuso tensor M6 com manípulo | Abraça eixos de Ø 20 a 150 mm trocando o comprimento da corrente; aperto firme sem marcar o eixo |
| **Haste radial** | Barra de aço Ø 10–12 mm, comprimentos de 100, 150 e 200 mm, rosqueada no bloco | Altura para passar por cima do acoplamento |
| **Garra em cruz** | Aço ou alumínio, dois furos a 90° com parafuso de aperto | Une a haste radial à barra axial com ajuste de altura e de distância |
| **Barra axial** | Aço Ø 10–12 mm, o **mais curta possível** | A flecha cresce com o cubo do vão; aço maciço flete menos que tubo fino |
| **Porta-relógio** | Furo de **8 mm** (haste padrão de relógio métrico) com aperto, ou fixação pela orelha traseira | Ponta **perpendicular** ao eixo e apontada para o **centro**, para evitar erro de cosseno |
| **Posição da Cabeça RRA** | Perto do bloco em V, não na ponta da barra | Peso perto da base reduz a flecha |

### 4.4 Regras de projeto e uso

1. **Rigidez acima de leveza; repetibilidade acima de flecha zero.** A flecha será medida e compensada (Fase 4 do método). O que não pode acontecer é ela variar. Meta: flecha ≤ 0,05 mm no vão usado, e três medições da flecha com diferença ≤ 0,01 mm.
2. **Nada estrutural impresso em 3D.** Plástico flexiona e muda com a temperatura. Use 3D só na caixa da eletrônica e no suporte do celular.
3. **Mesma posição angular nos dois suportes.** Na montagem, o App mostra o ângulo da Cabeça e o técnico ajusta o outro suporte até ficar alinhado (±2°, com o próprio celular como nível). Um erro de 3° gera um erro de leitura de apenas 0,14%.
4. **Pré-carga:** montar cada relógio com a ponta no meio do curso, para não bater no fim durante a rotação.
5. **Distâncias:** marcar na barra axial uma escala, ou medir com trena, as posições `xS`, `xM`, `F1` e `F2` que o App pede, sempre a partir do centro do acoplamento.
6. **Gabarito de flecha:** fazer junto um tubo rígido (ex.: tubo de aço Ø 50 mm, parede grossa) para medir a flecha com o suporte exatamente na configuração da máquina.

---

## 5. Custo estimado do kit (ordem de grandeza)

| Item | Estimativa | Base |
|---|---|---|
| 2 relógios digitais com saída de dados (ex.: Vonder RD-010) | R$ 550 – 850 | Preços pesquisados na Loja do Mecânico (variam por oferta) |
| Eletrônica da Cabeça RRA (ESP32, acelerômetro, conversores, bateria, caixa, cabos) | R$ 100 – 250 | **Estimativa, não pesquisada** |
| Material e usinagem dos 2 suportes | R$ 150 – 500 | **Estimativa, não pesquisada**; depende de oficina própria ou terceirizada |
| Analisador lógico para a prova de conceito | R$ 40 – 80 | **Estimativa, não pesquisada** |
| **Total** | **≈ R$ 850 – 1.700** | Contra ≈ R$ 2.550 só do par de relógios Werka, ainda sem suportes |

---

## 6. Plano de execução sugerido

1. **Prova de conceito (1 relógio):** comprar 1 Vonder RD-010, ler o protocolo com analisador lógico e decodificar com ESP32 na bancada.
2. **App MVP com `FonteManual` e `FonteSimulada`:** cálculo, wizard e relatório, em paralelo com a etapa 1.
3. **Cabeça RRA v1:** 2 relógios, acelerômetro e BLE, e integração da `FonteCabecaRRA` ao App, com captura automática e live move ao vivo.
4. **Suportes v1:** fabricar, medir a flecha e testar a repetibilidade numa bancada (dois eixos em mancais com acoplamento).
5. **Validação em campo:** comparar com o alinhamento feito pelo método tradicional ou por um sistema a laser emprestado.

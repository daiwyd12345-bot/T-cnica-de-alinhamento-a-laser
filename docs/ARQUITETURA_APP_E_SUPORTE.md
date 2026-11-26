# Arquitetura do App + Relógios Bluetooth + Suporte dos Relógios

Complemento do [Método RRA](METODO_ALINHAMENTO.md). Versão 1.1: decisão por relógios com Bluetooth integrado.

---

## 1. Decisão de hardware: relógios com Bluetooth integrado

**Decisão:** usar **dois relógios comparadores com Bluetooth (BLE) embutido**, conectados **diretamente ao celular**. Não há eletrônica própria nem ESP32.

Candidatos levantados (preços de loja, sujeitos a mudança):

| Modelo | Preço aprox. | Situação |
|---|---|---|
| Werka 4209-012W (12,7 mm / 0,01 mm, IP54) | ≈ R$ 1.270 cada | Compra nacional; o protocolo Bluetooth precisa ser confirmado |
| Sylvac S_Dial Work Smart (12,5 mm / 0,001 mm) | ≈ 615 CHF a 825 € cada | Importação ou orçamento com revendedor |
| Motionics BlueDial / BlueDial-LT | ≈ US$ 300 a 424 cada | Importação; tem app próprio para Android (MultiGage) |
| Mitutoyo digital + transmissor U-WAVE | ≈ R$ 2.800 cada | Compra nacional; conjunto de três peças |

### 1.1 Condição para comprar: o protocolo

O App só consegue ler o relógio se souber **como ele envia os dados**. Antes de comprar, peça ao fornecedor:

1. **O protocolo BLE (GATT) é documentado?** Existe SDK, ou ao menos a especificação dos serviços e características?
2. **O relógio envia leituras continuamente (notify), em quantas leituras por segundo?** Ou só quando se aperta o botão?
3. **O relógio funciona como "teclado Bluetooth" (HID)?** Nesse modo ele "digita" o valor. **Esse modo não serve**, porque o App precisa distinguir os dois relógios e receber o fluxo contínuo do live move.
4. **É possível conectar dois relógios iguais ao mesmo celular ao mesmo tempo**, e cada um tem identificador (MAC ou número de série) próprio?
5. **Dá para zerar o relógio remotamente pelo Bluetooth?** É desejável, mas não obrigatório, porque o App pode fazer o zero por software.

> **Regra:** comprar **um** relógio, testar com o app genérico **nRF Connect** (gratuito), confirmar as respostas acima e só depois comprar o segundo. Se o modelo não passar, troque de marca antes de investir no par.

---

## 2. Ângulo de rotação: o próprio celular como inclinômetro

Sem a caixa eletrônica, o ângulo vem do **acelerômetro do celular**, que fica preso a um dos eixos e gira junto com ele.

- **Onde prender:** em **suporte próprio** (pequeno bloco em V com corrente ou cinta) no eixo ou cubo da máquina **fixa**, e **não nas barras dos relógios**. O peso do celular (~200 g) na barra aumentaria a flecha.
- **Plano de medição:** o celular fica com a tela paralela à face do acoplamento (perpendicular ao eixo). O ângulo θ = atan2 da gravidade nesse plano, com o zero calibrado às 12h.
- **Retorno ao técnico:** como a tela gira com o eixo, o App avisa por **som, vibração e voz** ("três horas, capturado"). Um segundo aparelho (tablet ou outro celular) pode servir de tela espelho, o que fica para a v2.

### 2.1 Sincronização

Cada relógio é uma conexão Bluetooth separada. O App:

1. dá a cada amostra o **carimbo de tempo de chegada**;
2. guarda as últimas leituras de cada relógio e do ângulo;
3. só captura um ponto quando **os dois relógios e o ângulo estão estáveis ao mesmo tempo** (ver 3.2).

Com isso, o pequeno atraso entre as duas conexões não afeta o resultado, porque a medição é feita com os eixos parados em cada posição.

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
│ FonteDeMedicao (interface) ─┬─ FonteRelogiosBle                │
│                             │   = 2× DriverRelogio + ângulo    │
│                             │     do celular                   │
│                             ├─ FonteManual (digitação +        │
│                             │   inclinômetro do celular)       │
│                             └─ FonteSimulada (testes e demo)   │
│ DriverRelogio (interface) ──┬─ DriverWerka · DriverSylvac ·    │
│                             └─ DriverMotionics · DriverUWave   │
│ Banco local (SQLite) · PDF · Sensores do celular · Permissões  │
└────────────────────────────────────────────────────────────────┘
```

### 3.1 Abstração-chave: `FonteDeMedicao`

Todo o App enxerga apenas um fluxo de amostras:

```
Amostra { t_ms, rM_um, rS_um, angulo_graus, validoM, validoS }
```

Cada marca de relógio é um `DriverRelogio` (conectar, receber leitura em µm, zerar, nível de bateria). O primeiro driver a implementar é o do modelo comprado. Os outros entram só quando necessário.

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
2. **Dispositivos:** procurar e parear os dois relógios, **designando qual é o "M" e qual é o "S"** (o App lembra pelo identificador), bateria, zerar, teste de leitura ao vivo e um **modo diagnóstico** que registra os dados brutos do Bluetooth (ajuda a implementar drivers de novos modelos).
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
- Decodificador do driver do relógio (bytes → µm), incluindo valores negativos e polegada, com capturas reais gravadas no modo diagnóstico.
- `FonteSimulada` com ruído, para testar a captura automática.

---

## 4. Suporte dos relógios (o seu projeto mecânico)

### 4.1 Por que são **dois** suportes, e não um só

No método reverso, cada relógio precisa estar **rigidamente preso a um eixo** e girar com ele:

- o relógio M fica preso ao eixo fixo S e lê o eixo móvel;
- o relógio S fica preso ao eixo móvel M e lê o eixo fixo.

Uma peça única presa nos dois eixos ao mesmo tempo amarraria os eixos entre si e invalidaria a medição.

O "suporte" é portanto um **kit de dois suportes iguais e modulares**, mais um **suporte de celular** separado.

### 4.2 Desenho de conceito (vista lateral, relógios às 12h)

**Suporte A**: preso no eixo FIXO, leva o relógio M até o eixo MÓVEL.

```
                garra em cruz        barra axial Ø10–12 (aço)
                      ┌────────────────────────────●  ← Relógio M
                      │ haste radial               │    (ponta para baixo,
                      │                            ▼     no eixo móvel, plano xM)
                      │
            ┌─────────┴┐
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
| **Suporte do celular** (peça à parte) | Pequeno bloco em V com cinta ou corrente, presa no eixo ou cubo da máquina fixa; berço com trava firme | Mantém o peso do celular fora das barras; o celular não pode escorregar durante a rotação |
| **Proteção dos relógios** | Relógios Bluetooth são mais pesados que os analógicos | Aumenta a flecha, mas ela é medida e compensada |

### 4.4 Regras de projeto e uso

1. **Rigidez acima de leveza; repetibilidade acima de flecha zero.** A flecha será medida e compensada (Fase 4 do método). O que não pode acontecer é ela variar. Meta: flecha ≤ 0,05 mm no vão usado, e três medições da flecha com diferença ≤ 0,01 mm.
2. **Nada estrutural impresso em 3D.** Plástico flexiona e muda com a temperatura. Use 3D só no berço do celular.
3. **Mesma posição angular nos dois relógios e no celular.** Na montagem, com os eixos às 12h, o App mostra o ângulo do celular e o técnico ajusta os dois suportes até os relógios ficarem a prumo (±2°). Um erro de 3° gera um erro de leitura de apenas 0,14%.
4. **Pré-carga:** montar cada relógio com a ponta no meio do curso, para não bater no fim durante a rotação.
5. **Distâncias:** marcar na barra axial uma escala, ou medir com trena, as posições `xS`, `xM`, `F1` e `F2` que o App pede, sempre a partir do centro do acoplamento.
6. **Gabarito de flecha:** fazer junto um tubo rígido (ex.: tubo de aço Ø 50 mm, parede grossa) para medir a flecha com o suporte exatamente na configuração da máquina.

---

## 5. Custo estimado do kit (ordem de grandeza)

| Item | Estimativa | Base |
|---|---|---|
| 2 relógios com Bluetooth integrado (ex.: Werka 4209-012W) | ≈ R$ 2.550 | Preço pesquisado na Tecnoferramentas |
| Material e usinagem dos 2 suportes + suporte do celular | R$ 150 – 500 | **Estimativa, não pesquisada**; depende de oficina própria ou terceirizada |
| Tubo rígido para medir a flecha | R$ 50 – 150 | **Estimativa, não pesquisada** |
| **Total** | **≈ R$ 2.750 – 3.200** | Usa o celular que o técnico já tem |

---

## 6. Plano de execução sugerido

1. **Escolha do relógio:** enviar as perguntas da Seção 1.1 aos fornecedores. Comprar **um** e testar com o nRF Connect.
2. **App MVP com `FonteManual` e `FonteSimulada`:** cálculo, wizard e relatório, em paralelo com a etapa 1.
3. **Driver do relógio escolhido:** comprar o segundo relógio e integrar a `FonteRelogiosBle` com ângulo do celular, captura automática e live move ao vivo.
4. **Suportes v1:** fabricar, medir a flecha e testar a repetibilidade numa bancada (dois eixos em mancais com acoplamento).
5. **Validação em campo:** comparar com o alinhamento feito pelo método tradicional ou por um sistema a laser emprestado.

# Prompt para o programador — App de Alinhamento de Eixos "RRA"

> Copie tudo abaixo da linha e envie ao programador (pessoa ou assistente de IA).
> Os documentos de apoio estão em `docs/METODO_ALINHAMENTO.md` e `docs/ARQUITETURA_APP_E_SUPORTE.md`.

---

## 1. Contexto

Você vai desenvolver um **aplicativo móvel de alinhamento de eixos de máquinas rotativas** (ex.: motor elétrico acoplado a bomba). O app usa **dois relógios comparadores digitais com Bluetooth** e o **acelerômetro do celular** como inclinômetro, e reproduz o fluxo de trabalho de um alinhador a laser profissional:

- medição guiada passo a passo;
- captura automática por ângulo;
- cálculo de calços e movimentos nos pés da máquina;
- ajuste horizontal acompanhado ao vivo ("live move");
- relatório em PDF.

O método de medição é o **relógio reverso** (reverse dial indicator), que chamamos de **Método RRA (Relógio Reverso Assistido)**. Toda a matemática está especificada abaixo e **deve ser seguida exatamente**.

O usuário é um **técnico de manutenção em campo**: mãos sujas ou com luvas, ambiente industrial, às vezes sem internet. Por isso, a interface precisa de botões grandes, alto contraste, poucas digitações e funcionamento 100% offline.

---

## 2. Stack e restrições

- **Flutter (Dart)**, um único código para **Android e iOS**.
- **100% offline.** Nenhuma funcionalidade depende de internet ou de servidor.
- Banco local: `drift` (SQLite) ou `sqflite`.
- Bluetooth Low Energy: `flutter_blue_plus`.
- Acelerômetro: `sensors_plus`.
- PDF: `pdf` + `printing`.
- Permissões: `permission_handler` (Android 12+: `BLUETOOTH_SCAN` e `BLUETOOTH_CONNECT`; iOS: `NSBluetoothAlwaysUsageDescription`).
- Gerência de estado à sua escolha (Riverpod ou Bloc), mas com separação clara de camadas (Seção 4).
- Idioma da interface: **português do Brasil**. Textos centralizados para tradução futura.
- Unidades: **mm (padrão)** e **mils (0,001")**, alternáveis. Internamente, tudo em **micrômetros (µm), inteiros**, ou em mm `double` na camada de cálculo.
- Código e identificadores podem ser em português ou inglês, mas de forma consistente.

---

## 3. Entregas por fases

Cada fase termina com o app compilando, os testes passando e uma demonstração funcional.

| Fase | Entrega |
|---|---|
| **A** | Camada de **domínio** (cálculo puro em Dart) com **testes unitários** usando o caso de teste da Seção 7 |
| **B** | App completo com **entrada manual** das leituras + inclinômetro do celular: cadastro, wizard das fases, resultado, live move calculado e relatório PDF |
| **C** | Infraestrutura Bluetooth: tela de dispositivos, **modo diagnóstico BLE**, `FonteSimulada` e captura automática |
| **D** | **Driver do relógio Werka 4209-012W**, a ser feito quando tivermos o protocolo, que está em consulta ao fabricante; e live move em tempo real |

**Não comece a Fase D sem o protocolo confirmado.** As Fases A a C não dependem do modelo do relógio.

---

## 4. Arquitetura

```
UI (telas)
  └── Aplicação: SessaoAlinhamento (máquina de estados das fases),
                 CapturaAutomatica, ValidacaoMedicao, GeradorRelatorio
        └── Domínio (Dart puro, sem Flutter, 100% testável):
              CalculadoraRRA, AjusteSenoidal (mínimos quadrados), Flecha,
              Tolerancias, MetasTermicas, CorrecaoPes, LiveMoveAlvos, Modelos
        └── Infraestrutura:
              FonteDeMedicao (interface)
                ├─ FonteManual        (digitação + ângulo do celular)
                ├─ FonteSimulada      (gera leituras de um desalinhamento conhecido + ruído)
                └─ FonteRelogiosBle   (2 × DriverRelogio + ângulo do celular)
              DriverRelogio (interface)
                ├─ DriverWerka        (Fase D)
                └─ (futuros: DriverSylvac, DriverMotionics, DriverUWave)
              Repositórios (SQLite), ServicoPdf, SensorAngulo, Permissoes
```

### 4.1 Interfaces obrigatórias (assinaturas indicativas)

```dart
class Amostra {
  final int tMs;              // carimbo de tempo de chegada (relógio do celular)
  final double? rM;           // leitura do relógio M em mm (null = indisponível)
  final double? rS;           // leitura do relógio S em mm
  final double? anguloGraus;  // 0 = 12h, sentido horário visto do observador
}

abstract class FonteDeMedicao {
  Stream<Amostra> get amostras;
  Future<void> iniciar();
  Future<void> parar();
}

abstract class DriverRelogio {
  String get id;                       // MAC ou número de série
  Stream<double> get leiturasMm;       // leitura contínua, em mm
  Stream<int?> get bateriaPct;
  Future<void> conectar();
  Future<void> desconectar();
  Future<void> zerar();                // remoto, se suportado; senão, zero por software
}
```

- O **zero por software** (tara) sempre existe no app, independentemente do relógio.
- A `FonteRelogiosBle` combina os dois drivers e o ângulo **pela última amostra válida de cada um**, com carimbo de tempo.

---

## 5. Convenções (crítico: erros aqui invertem calços)

- **Máquina FIXA (S)**: não se move. **Máquina MÓVEL (M)**: recebe calços e é deslocada.
- **Observador**: atrás da máquina móvel, olhando para a fixa. **12h = topo, 3h = direita, 6h = baixo, 9h = esquerda.**
- **Ângulo θ**: 0° às 12h, crescendo no **sentido horário** visto pelo observador (90° = 3h, 180° = 6h, 270° = 9h).
- **Eixo axial x**: origem no **centro do acoplamento**, positivo em direção à máquina móvel. Unidade: mm.
- **Relógio M**: preso no eixo **fixo**, apalpa o eixo **móvel** no plano `xM` (> 0).
- **Relógio S**: preso no eixo **móvel**, apalpa o eixo **fixo** no plano `xS` (< 0).
- **Leitura**: haste entrando = positivo.
- **Resultado vertical**: + = móvel **acima**. **Resultado horizontal**: + = móvel deslocada **para 3h**.
- **Correção vertical**: + = **adicionar calço**, − = **remover**. **Correção horizontal**: + = mover **para 3h**, − = mover **para 9h**.
- **Pés**: `F1` = dianteiro (perto do acoplamento) e `F2` = traseiro, ambos com x > 0.
- **Sag (flecha)**: medido com o suporte num tubo rígido, zerado às 12h e lido às 6h. Valor **≤ 0** (ex.: −0,04 mm).

Na tela, toda correção deve aparecer **por extenso**, por exemplo: "**REMOVER 0,16 mm** de calço no pé **dianteiro**" e "**EMPURRAR 0,33 mm para a ESQUERDA (9h)** o pé **traseiro**".

---

## 6. Matemática (implementar exatamente)

### 6.1 Modelo de leitura

Para um relógio cujo eixo apalpado está deslocado `(z, y)` em relação ao eixo onde o relógio está preso:

```
r(θ) = k + (y − Sag/2)·cos θ + z·sin θ
```

### 6.2 Modo multiponto (arco restrito)

Com N ≥ 3 pares `(θᵢ, rᵢ)`, ajustar por **mínimos quadrados**:

```
r = k + C·cos θ + S·sin θ      →   y = C + Sag/2,  z = S
```

- Calcular e exibir o **resíduo RMS** do ajuste como indicador de qualidade.
- Exigir arco total **≥ 60°** (bloqueio) e recomendar **≥ 90°** (aviso).

### 6.3 Modo 4 posições (forma fechada, zero às 12h)

```
V = R6 − Sag        H = R3 − R9
y = −V/2            z = +H/2
```

Validade: `|R12 + R6 − (R3 + R9)| ≤ max(0,02 mm; 10% de |R6|)`. Se reprovar, o app pede para refazer e lista as causas prováveis (folga no suporte, relógio travando, eixo que não acompanhou a rotação, leitura errada).

### 6.4 Linha de centro da móvel

- Do relógio M obtém-se `(yM, zM)`, e a posição da móvel em `xM` é `(yM, zM)`.
- Do relógio S obtém-se `(yS, zS)`, e a posição da móvel em `xS` é `(−yS, −zS)`.

Para cada plano (vertical e horizontal), com `pM` = posição em `xM` e `pS` = posição em `xS`:

```
D = xM − xS
b = (pM − pS) / D          // angularidade (mm/mm)
a = pS − b · xS            // offset no centro do acoplamento (mm)
p(x) = a + b·x
```

### 6.5 Correção nos pés (com metas)

```
Correcao(F) = (a_alvo + b_alvo·F) − (a + b·F)       // sem metas: −p(F)
```

### 6.6 Metas térmicas (opcional por ativo)

```
ΔL = α · L · ΔT     α_aço = 11,5e-6 /°C   L = altura base→linha de centro (mm)
a_alvo_vertical = −(ΔL_móvel − ΔL_fixa),   b_alvo = 0
```

Também deve ser possível digitar diretamente os alvos do fabricante (offset e angularidade, em V e em H).

### 6.7 Exibição

- Offset: `a` (mm).
- Angularidade: `b·100` (mm/100 mm) e gap no acoplamento `b·Dc` (mm), onde `Dc` é o diâmetro do acoplamento.
- Gráfico com a linha de centro da fixa (reta em 0) e a da móvel (`p(x)`), com xS, xM, F1, F2 e o acoplamento marcados. A escala vertical deve ser exagerada e indicada.

### 6.8 Live move horizontal

Os relógios são posicionados às 3h e zerados. Alvos:

```
alvo_M = −H_M/2      alvo_S = −H_S/2
```

No modo multiponto, `H/2` equivale a `z`, ou seja, `alvo_M = −zM` e `alvo_S = −zS`. Com metas, somar a contribuição do alvo no plano de cada relógio.

### 6.9 Tolerâncias padrão (editáveis por ativo)

| RPM | Offset exc. | Offset aceit. | Ang. exc. (mm/100) | Ang. aceit. (mm/100) |
|---|---|---|---|---|
| 0–1000 | 0,07 | 0,13 | 0,06 | 0,10 |
| 1000–2000 | 0,05 | 0,10 | 0,05 | 0,08 |
| 2000–3000 | 0,03 | 0,07 | 0,04 | 0,07 |
| 3000–4000 | 0,02 | 0,05 | 0,03 | 0,06 |
| 4000–6000 | 0,01 | 0,03 | 0,02 | 0,05 |

Status por plano e por grandeza: **Excelente** (verde), **Aceitável** (amarelo), **Fora** (vermelho). Use ícone e texto além da cor.

---

## 7. Caso de teste obrigatório (testes unitários da Fase A)

Geometria: `xS = −100`, `xM = 100`, `F1 = 300`, `F2 = 800` mm. `Sag = −0,04` mm.

**Modo 4 posições**, leituras brutas zeradas às 12h:

| Relógio | 12h | 3h | 6h | 9h |
|---|---|---|---|---|
| M | 0,00 | −0,09 | −0,28 | −0,19 |
| S | 0,00 | +0,09 | +0,12 | +0,03 |

Resultados esperados (tolerância do teste ±0,0005 mm):

| | a (mm) | b (mm/mm) | Correção F1 | Correção F2 | Live move |
|---|---|---|---|---|---|
| Vertical | +0,10 | +0,0002 | −0,16 (remover) | −0,26 (remover) | — |
| Horizontal | +0,01 | +0,0004 | −0,13 (para 9h) | −0,33 (para 9h) | alvo_M = −0,05; alvo_S = −0,03 |

Validade: `R3 + R9 − R6 = 0` nos dois relógios.

**Modo multiponto**: gerar leituras pelo modelo 6.1 nos ângulos 30°, 70°, 110° e 150° para o mesmo desalinhamento. O relógio M mede `(y, z) = (p_vert(xM), p_horiz(xM))` e o relógio S mede `(−p_vert(xS), −p_horiz(xS))`. O ajuste deve devolver os **mesmos a e b** acima.

**Teste de ruído**: somar ruído uniforme de ±0,005 mm. O resultado deve variar no máximo ±0,01 mm em `a`.

---

## 8. Fluxo do app (wizard)

Cada fase é uma tela. O app **bloqueia** o avanço quando indicado, a não ser que o técnico registre uma justificativa, que vai para o relatório.

| # | Fase | Conteúdo | Bloqueio |
|---|---|---|---|
| 0 | Segurança e dados | Checklist de bloqueio (LOTO), ativo, RPM, tipo de acoplamento, tolerância | LOTO não confirmado |
| 1 | Inspeção | Fundação, chumbadores, calços (máx. 4 por pé), folga axial, tensão de tubulação (> 0,05 mm reprova) | Tensão de tubulação reprovada |
| 2 | Batimento | Leitura do TIR do eixo e do cubo (≤ 0,05 mm) | TIR > 0,05 mm |
| 3 | Pé manco | Leitura por pé (4 ou 6 pés). > 0,05 mm reprova; tipo do pé manco | Algum pé reprovado |
| 4 | Flecha | Sag de cada suporte (3 medições, dispersão ≤ 0,01 mm, usa a média) | Dispersão alta |
| 5 | Geometria | Desenho interativo; entrada de xS, xM, F1, F2, Dc | Campos faltando ou xS ≥ 0 ou xM ≤ 0 |
| 6 | Medição | 4 posições ou multiponto; **2 medições** com concordância ≤ 0,02 mm em `a` | Validade ou repetibilidade reprovada |
| 7 | Metas | Nenhuma / térmica / do fabricante | — |
| 8 | Resultado e correção | Gráfico, status, correções por extenso; ordem obrigatória: **vertical → nova medição → horizontal (live move) → medição final** | Horizontal antes de a vertical estar OK (aviso forte) |
| 9 | Relatório | "Como encontrado" × "como deixado", fotos, observações, assinatura na tela, PDF | — |

---

## 9. Telas principais

1. **Início / Ativos**: lista com busca, cadastro (TAG, local, RPM, acoplamento, Dc, fabricante, tolerância, metas) e histórico de alinhamentos por ativo.
2. **Dispositivos** (Fase C):
   - procurar relógios BLE e conectar;
   - **atribuir "Relógio M" e "Relógio S"**, lembrando pelo `id`;
   - leitura ao vivo, bateria e zerar;
   - **Modo diagnóstico**: lista serviços e características GATT, assina notificações e mostra e grava os bytes brutos com carimbo de tempo, exportando em CSV. Ferramenta para implementar novos drivers.
3. **Medição**:
   - mostrador circular com o ângulo atual e as posições-alvo;
   - leituras M e S grandes;
   - pontos capturados no círculo;
   - botão "capturar" manual sempre disponível.
4. **Resultado**: cartões V e H (offset e angularidade com status), gráfico das linhas de centro e correções por extenso para F1 e F2.
5. **Live Move**: dois números enormes (M e S), alvo de cada um, barra de aproximação, seta para 3h ou 9h, e o status muda para verde quando os dois estão dentro de ±0,01 mm do alvo.
6. **Relatório**: pré-visualização e compartilhamento do PDF.
7. **Configurações**: unidade, tabela de tolerâncias padrão, limites (validade, repetibilidade, estabilidade), som e voz, calibração do zero do ângulo.

---

## 10. Ângulo e captura automática

- **Ângulo**: acelerômetro do celular, que fica preso ao eixo da máquina fixa com a tela perpendicular ao eixo. `θ = atan2(g_x, g_y)` no plano da tela, com filtro passa-baixa (~200 ms).
- **Calibração**: botão "zerar ângulo às 12h". A direção de rotação deve ser confirmada pelo app pedindo um giro até as 3h.
- **Captura automática**, quando todas as condições abaixo forem verdadeiras ao mesmo tempo:
  1. ângulo a ±3° do alvo (4 posições) ou a ≥ 20° do último ponto (multiponto);
  2. leituras M e S estáveis (desvio-padrão < 0,002 mm na janela de 1 s);
  3. ângulo estável (variação < 0,5° na mesma janela);
  4. as duas leituras disponíveis e recentes (< 500 ms).
- **Retorno ao técnico**: vibração, bipe e **voz** (TTS em português: "Três horas, capturado"), porque a tela gira com o eixo.
- Com a `FonteManual`, o técnico digita as leituras e o ângulo é sugerido pelo sensor, podendo ser editado.

---

## 11. Modelo de dados (mínimo)

- `Ativo` (id, tag, local, rpm, tipoAcoplamento, dc, tolerancia, metas)
- `Alinhamento` (id, ativoId, data, tecnico, status, observacoes, assinatura)
- `Geometria` (xS, xM, f1, f2, dc)
- `Inspecao`, `Batimento`, `PeManco[]`, `Flecha` (média, medições)
- `Medicao` (tipo: comoEncontrado | intermediaria | comoDeixado; modo; pontos[] {θ, rM, rS}; resultadoV {a, b}; resultadoH {a, b}; validade; residuo)
- `Correcao` (planos, pés, valores aplicados de fato)
- `Foto` (caminho, legenda)

---

## 12. Relatório PDF

Deve conter:

- cabeçalho (logo configurável, empresa, TAG, local, data, técnico);
- dados do ativo e tolerância;
- inspeção, batimento, pé manco e flecha;
- geometria com desenho;
- tabela **como encontrado × como deixado** (offset e angularidade, V e H, com status);
- gráficos das linhas de centro;
- correções aplicadas, metas usadas, justificativas de bloqueios ignorados;
- fotos e assinatura.

---

## 13. Critérios de aceite

- [ ] Testes da Seção 7 passando: 4 posições, multiponto e ruído.
- [ ] Ciclo completo sem nenhum relógio conectado, usando a `FonteManual`, gerando PDF.
- [ ] Ciclo completo com a `FonteSimulada`, incluindo captura automática e live move.
- [ ] Modo diagnóstico BLE lista e grava os dados de qualquer periférico BLE.
- [ ] Nenhuma tela exibe correção sem a frase por extenso (adicionar/remover, esquerda/direita, pé dianteiro/traseiro).
- [ ] Funciona em modo avião (exceto Bluetooth).
- [ ] Compila e roda em Android 8+ e iOS 15+.

---

## 14. O que NÃO fazer

- Não alterar convenções de sinal nem fórmulas sem validar com o caso de teste.
- Não acoplar a lógica de cálculo à UI ou ao Bluetooth: o domínio é Dart puro.
- Não depender de serviço em nuvem, login ou internet.
- Não assumir o protocolo do relógio Werka: aguardar a especificação ou os dados do modo diagnóstico.

---

## 15. Antes de começar, responda

1. Estimativa de prazo e esforço por fase (A–D).
2. Gerência de estado e banco escolhidos, e por quê.
3. Dúvidas sobre a matemática ou as convenções.
4. Necessidade de algum dispositivo físico para testes (celulares Android e iOS).

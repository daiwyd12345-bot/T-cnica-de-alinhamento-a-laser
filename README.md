# Técnica de Alinhamento a Laser com Relógio Comparador — Método RRA

Projeto de um método e de um App de alinhamento de eixos com a lógica de um alinhador a laser, usando **dois relógios comparadores com Bluetooth** e o **celular** como inclinômetro.

## Documentos

| Arquivo | Conteúdo |
|---|---|
| [docs/METODO_ALINHAMENTO.md](docs/METODO_ALINHAMENTO.md) | Método RRA: convenções, fases do procedimento, matemática, tolerâncias e exemplo resolvido |
| [docs/ARQUITETURA_APP_E_SUPORTE.md](docs/ARQUITETURA_APP_E_SUPORTE.md) | Arquitetura do App, relógios Bluetooth, ângulo pelo celular e projeto dos suportes |
| [docs/PROMPT_PROGRAMADOR.md](docs/PROMPT_PROGRAMADOR.md) | Especificação completa para o programador (pronta para enviar) |
| [docs/MENSAGEM_WERKA.md](docs/MENSAGEM_WERKA.md) | Consulta ao fornecedor sobre o protocolo Bluetooth e critério para avaliar a resposta |

## Situação atual (26/11/2026)

**Decisões tomadas**
- Método: relógio reverso assistido (RRA), com modo 4 posições e modo multiponto.
- Relógios: **com Bluetooth integrado**; candidato principal **Werka 4209-012W** (≈ R$ 1.270 cada).
- Ângulo: acelerômetro do celular, preso em suporte próprio no eixo da máquina fixa.
- Suportes: kit com **dois suportes** (um por eixo) + suporte do celular.
- App: Flutter (Android e iOS), 100% offline, mm como unidade padrão (com mils). Ainda falta confirmação explícita destes três pontos.

**Pendente**
- [ ] **Resposta da Werka** sobre o protocolo Bluetooth → avaliar com a tabela em `docs/MENSAGEM_WERKA.md`.
- [ ] Enviar `docs/PROMPT_PROGRAMADOR.md` ao programador. As Fases A–C podem começar já; a Fase D depende da Werka.
- [ ] Comprar 1 relógio e testar com o app nRF Connect antes de comprar o segundo.
- [ ] Fabricar os suportes e medir a flecha em bancada.

## Ao receber a resposta da Werka

1. Avaliar com a tabela de `docs/MENSAGEM_WERKA.md`.
2. Se aprovada: registrar o protocolo (ou as capturas do nRF Connect) em `docs/` e completar a **Fase D** do `docs/PROMPT_PROGRAMADOR.md` com o driver Werka.
3. Se reprovada: trocar para Sylvac ou Motionics, repetir a consulta e atualizar o prompt.

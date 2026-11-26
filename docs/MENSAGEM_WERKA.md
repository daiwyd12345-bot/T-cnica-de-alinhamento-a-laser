# Mensagem ao fornecedor Werka — consulta sobre o protocolo Bluetooth

Status: **aguardando resposta** (enviada pelo usuário em nov/2026).

---

## Versão e-mail

**Assunto:** Relógio comparador Werka 4209-012W: dúvidas técnicas sobre a comunicação Bluetooth

Prezados,

Estamos desenvolvendo um aplicativo próprio para celular (Android e iOS) para alinhamento de eixos de máquinas rotativas. Pretendemos usar **dois relógios comparadores digitais Werka 4209-012W** (0–12,7 mm, resolução 0,01 mm, Bluetooth) ligados ao mesmo celular.

Antes da compra, precisamos confirmar alguns pontos técnicos da comunicação Bluetooth:

1. **Protocolo:** a comunicação usa Bluetooth Low Energy (BLE)? Vocês fornecem documentação do protocolo (serviços e características GATT, formato dos dados) ou um SDK para integração com aplicativo próprio?
2. **Envio das leituras:** o relógio envia as leituras de forma contínua, com o valor atualizado automaticamente? Se sim, com qual frequência (leituras por segundo)? Ou só envia quando se aperta um botão?
3. **Modo de operação:** o relógio funciona como "teclado Bluetooth" (HID), digitando o valor no aparelho? Existe outro modo de transmissão de dados?
4. **Dois relógios simultâneos:** é possível conectar dois relógios 4209-012W ao mesmo celular ao mesmo tempo? Cada unidade tem identificação própria (endereço MAC ou número de série) transmitida pelo Bluetooth?
5. **Comandos remotos:** é possível zerar o relógio ou trocar a unidade (mm/pol) pelo Bluetooth?
6. **Formato do valor:** como chegam os valores negativos e qual a resolução transmitida?
7. **Bateria:** qual a autonomia com o Bluetooth ligado, e o nível de bateria é informado pela conexão?
8. **Software:** existe aplicativo ou software Werka compatível com o relógio, que possamos usar como referência?
9. **Ficha técnica:** podem enviar o manual completo e a ficha técnica do modelo, incluindo peso e diâmetro da haste de fixação?

Se o protocolo for compatível, a intenção é comprar inicialmente **uma unidade para teste** e, depois de validada, a segunda unidade. Gostaríamos também de receber o **orçamento para 2 unidades**, com prazo de entrega e garantia.

Agradeço desde já.

Atenciosamente,
[Nome]
[Empresa]
[Telefone / e-mail]

---

## Versão curta (WhatsApp)

> Olá! Tenho interesse em 2 relógios comparadores Werka 4209-012W com Bluetooth para usar num aplicativo próprio de alinhamento de eixos. Antes de comprar, preciso saber:
> 1. É Bluetooth BLE? Vocês têm documentação do protocolo ou SDK?
> 2. Ele envia as leituras continuamente (quantas por segundo) ou só ao apertar um botão?
> 3. Funciona só como "teclado Bluetooth" ou tem outro modo?
> 4. Dá para conectar 2 relógios no mesmo celular ao mesmo tempo, cada um identificado?
> 5. Dá para zerar pelo Bluetooth?
>
> Se puderem, mandem também o manual e o orçamento para 2 unidades. Obrigado!

---

## Como avaliar a resposta

| Resposta | Decisão |
|---|---|
| Protocolo documentado + envio contínuo + 2 relógios simultâneos | ✅ Comprar 1 unidade, testar com nRF Connect, depois a 2ª; liberar a Fase D |
| Sem documentação, mas BLE com envio contínuo | ⚠️ Viável: comprar 1, capturar os dados no nRF Connect ou no modo diagnóstico do App e criar o driver a partir disso |
| Só funciona como "teclado Bluetooth" (HID) ou só envia ao apertar botão | ❌ Não serve: partir para Sylvac S_Dial Work Smart ou Motionics BlueDial |
| Não permite 2 relógios no mesmo celular | ❌ Não serve |

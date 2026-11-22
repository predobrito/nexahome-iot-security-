# Mapa de interfaces físicas – NexaSense (laboratório)

Ambiente: simulação no Wokwi (ESP32 + DHT22). Em simulação não há acesso aos
pinos reais. A análise usa o comportamento observado no monitor serial, o
pinout da placa e a análise do firmware (pasta firmware/).

## Resumo das interfaces

| Interface | Onde | O que permite | Evidência |
|---|---|---|---|
| UART0 | GPIO1 (TX) / GPIO3 (RX) | Ver logs de boot e enviar comandos ao console de debug | evidence/02-uart-config.png, 02b-uart-boot.png |
| JTAG | GPIO12 a 15 | Depuração do chip e acesso à memória, se habilitado | evidence/03-pinout-uart-jtag.png |
| Flash (SPI) | Chip de memória do módulo | Leitura do firmware, das credenciais e dos dados das partições | ../firmware/06-strings-busca.png, ../firmware/07-particoes-ota.png |
| USB | Conector da placa | Alimentação e gravação de firmware | evidence/01-circuito-wokwi.png |
| Sensor DHT22 | GPIO15 | Leitura de temperatura e umidade | evidence/01-circuito-wokwi.png |
| I²C / SPI de periféricos | Pinos livres | Expansão de periféricos | Pinout (não usados no protótipo) |
| Pontos de teste | Não observáveis na simulação | – | Limitação do ambiente |

## Análise por interface

### UART0
- **O que permite:** o firmware imprime no boot a configuração completa e
  aceita os comandos `config`, `version` e `ota` sem autenticação.
- **O que entrega:** usuário e senha MQTT, URL de atualização, versão e
  identificador. O código também registra usuário e senha a cada conexão
  MQTT (`MQTT connect user=... pass=...`).
- **Proteção existente:** nenhuma observada.
- **Risco em produto comercial:** um atacante com acesso físico poderia ligar
  um adaptador USB-serial e obter credenciais válidas para toda a frota, já
  que a senha é a mesma em todas as unidades.
- **Impacto:** alto. **Probabilidade:** média (exige acesso físico).
- **Conclusão:** a UART em si é normal em protótipos. O problema é a
  informação que ela entrega sem controle de acesso.

### JTAG
- **O que permite:** depuração em baixo nível e acesso à memória do chip.
- **Proteção existente:** não verificável na simulação.
- **Risco:** se permanecer habilitado e acessível em produção, facilita a
  extração de firmware e o controle da execução.
- **Observação:** o sensor DHT22 usa o GPIO15, que também é o pino JTAG TDO.
  O mesmo pino tem uso duplo e precisa ser planejado.
- **Mitigação (conceitual):** desabilitar o JTAG em produção.

### Flash (SPI)
- **O que permite:** ler o conteúdo da memória. A análise mostrou que ela
  guarda firmware em texto legível, as partições `nvs` e `coredump` e duas
  áreas de aplicação para atualização.
- **Proteção existente:** nenhuma observada; não há indício de criptografia.
- **Risco:** extrair a flash expõe credenciais e lógica do produto.
- **Mitigação (conceitual):** Flash Encryption e Secure Boot.

### USB
- **O que permite:** alimentar a placa e regravar o firmware.
- **Risco:** com acesso físico, o firmware pode ser substituído se não houver
  verificação de assinatura.
- **Mitigação (conceitual):** Secure Boot e restrição do modo de gravação.

### DHT22, I²C e SPI de periféricos
- Interfaces de dados do sensor. Risco baixo no contexto analisado, pois
  transportam apenas temperatura e umidade.

## Resposta à pergunta central
Para cada interface acima, o que ela permite e o risco de permanecer acessível
em produto comercial estão descritos na própria seção. Nenhuma interface foi
classificada como vulnerabilidade só por existir: a classificação considera o
que ela expõe e a ausência de proteção.

## Limitações
- Análise feita em simulador: não houve contato com pinos reais.
- JTAG e USB foram analisados pelo pinout e pela documentação, não por teste.

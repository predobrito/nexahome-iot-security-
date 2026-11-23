 NexaHome – Avaliação de segurança do dispositivo IoT NexaSense

Do Protótipo ao Produto Seguro: Avaliação de Segurança de um Dispositivo IoT**
Autor: < PEDRO AUGUSTO BRITO CASTILHO PEREIRA - RA: 100954 > · Data: 22/11/2026
 
 1. Objetivo
Avaliar a segurança do protótipo NexaSense (sensor de temperatura e umidade
com ESP32) antes de um projeto-piloto: identificar superfícies de ataque,
analisar hardware, firmware e comunicações, demonstrar uma fragilidade de
forma controlada, classificar os riscos e propor medidas de hardening.

2. Dispositivo analisado
Versão de laboratório simulada: ESP32 DevKit com sensor DHT22 (GPIO 15),
firmware educacional NexaSense v0.9.3-debug (Arduino), comunicação Wi-Fi e MQTT.
Projeto de simulação no Wokwi: <https://wokwi.com/projects/481389965859119105>

 3. Limites e escopo
- Todo o trabalho foi feito em simulador (Wokwi, com verificação do código no
  Arduino IDE), sem hardware físico.
- Apenas o dispositivo simulado e o projeto do próprio aluno foram testados.
- Nenhum teste contra dispositivos, serviços ou redes de terceiros.
- O tráfego capturado é o do próprio dispositivo, com dados fictícios, e o
  tópico MQTT é próprio. O servidor MQTT usado é um broker público.
- Nenhum mecanismo irreversível (eFuses) foi acionado; Secure Boot, Flash
  Encryption e JTAG foram tratados apenas de forma conceitual.
- Credenciais, usuário, senha e identificadores do firmware são **fictícios**,
  criados para o laboratório. Nenhuma credencial real foi usada ou publicada.

 4. Arquitetura
Hardware (MCU, memória, sensor, interfaces) → Firmware → Protocolos
(Wi-Fi, MQTT 1883) → Serviço (broker MQTT e plataforma de monitoramento).
Diagrama: `architecture/architecture-diagram.png`
Componentes: `architecture/components.md`

 5. Ferramentas utilizadas
| Ferramenta | Uso |
|---|---|
| Wokwi | simulação do ESP32, monitor serial (UART) e captura de rede (PCAP) |
| Arduino IDE 2.3.11 (pacote ESP32) | compilação do firmware e geração do .bin |
| PowerShell | hash SHA-256, leitura da estrutura e extração de textos do firmware |
| Wireshark | análise do tráfego MQTT |
| draw.io / navegador | diagrama de arquitetura |

 6. Organização das evidências
```
├── README.md
├── architecture/      diagrama e componentes
├── firmware/          hash, notas de análise, textos extraídos, prints
│   ├── firmware original/   imagem original preservada
│   └── firmware work/       cópia de trabalho usada na análise
├── hardware/          mapa de interfaces e evidence/ (UART, pinout, circuito)
├── network/           análise de protocolo e evidence/ (PCAP, Wireshark)
├── findings/          findings.md e demo-controlada.md
└── remediation/       hardening-plan.md
```

7. Resumo da análise de firmware
- Imagem original preservada e identificada por SHA-256 (`firmware/hashes.txt`);
  a análise foi feita em cópia idêntica (hashes conferidos).
- Imagem completa de 4 MiB: bootloader, tabela de partições e aplicação.
- Partições: nvs, otadata, app0, app1, spiffs e coredump (duas áreas de
  aplicação indicam atualização OTA).
- Encontrados em texto legível: credencial MQTT, URL de atualização em HTTP,
  broker e tópico, versão "debug" e mensagens de depuração.
Detalhes: `firmware/analysis-notes.md`

 8. Resumo da análise de hardware
- Interfaces analisadas: UART0, JTAG, Flash (SPI), USB, sensor DHT22 e
  I²C/SPI de periféricos; pontos de teste não observáveis na simulação.
- A UART entrega a configuração no boot e aceita comandos sem autenticação.
- JTAG e USB foram analisados pela documentação e pelo pinout, sem
  classificação de risco por falta de evidência no simulador.
Detalhes: `hardware/interface-map.md`

9. Resumo da análise dos protocolos
- MQTT sobre TCP, porta 1883, sem TLS.
- A captura de rede mostra usuário e senha em texto puro no pacote CONNECT e
  a telemetria legível nos pacotes PUBLISH.
Detalhes: `network/protocol-analysis.md`

10. Principais achados
| ID | Achado | Risco |
|---|---|---|
| F01 | Credencial MQTT gravada no firmware, igual em todas as unidades | Alto |
| F02 | Console da UART expõe credenciais sem autenticação | Alto |
| F03 | Comunicação MQTT sem criptografia | Alto |
| F04 | Atualização por HTTP, sem indício de verificação de assinatura | Médio |
| F05 | Build de debug e mensagens verbosas | Médio |
| F06 | Flash sem proteção evidente | Médio |
Detalhes e demonstrações: `findings/findings.md`, `findings/demo-controlada.md`

 11. Recomendações
Prioridade P1 (antes do piloto): credencial única por dispositivo, remover
logs sensíveis e console de debug, MQTT sobre TLS, build de produção.
Prioridade P2 (versão comercial): atualização por HTTPS com firmware
assinado e rollback, Secure Boot e Flash Encryption, desativação do JTAG.
Detalhes: `remediation/hardening-plan.md`

 12. Como reproduzir os testes (ambiente autorizado)
1. **Simulação:** criar um projeto ESP32 no Wokwi, adicionar um DHT22 (VCC→3V3,
   GND→GND, SDA→GPIO15), instalar PubSubClient e DHT sensor library e usar o
   firmware educacional. Trocar o identificador do tópico por um próprio.
2. **UART:** com a simulação rodando, observar o boot no monitor serial e
   digitar `config` e `ota`.
3. **Rede:** deixar rodar cerca de 30 s, baixar o PCAP pelo ícone de Wi-Fi,
   abrir no Wireshark com o filtro `mqtt` e inspecionar CONNECT e PUBLISH.
4. **Firmware:** compilar o mesmo código no Arduino IDE (ESP32 Dev Module) com
   "Export Compiled Binary" e usar o arquivo `*.merged.bin`.
5. **Preservação:** calcular o hash com
   `Get-FileHash -Algorithm SHA256 <arquivo>` e trabalhar numa cópia.
6. **Estrutura:** ler os bytes em 0x1000, 0x8000 e 0x10000 (E9, AA 50, E9) e
   decodificar a tabela de partições a partir de 0x8000.
7. **Textos:** extrair sequências imprimíveis do .bin e buscar por usuário,
   senha, URLs e mensagens de debug.
Os comandos exatos estão em `firmware/analysis-notes.md`.

 13. Vídeo de demonstração
<https://youtu.be/zX9LVlkDobQ>

## 14. Referências
OWASP IoT Security Testing Guide · NIST IR 8259A · ETSI EN 303 645 ·
MITRE EMB3D · Espressif ESP-IDF Security Documentation.

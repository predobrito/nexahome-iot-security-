# Componentes do dispositivo NexaSense (versão de laboratório)

Ambiente: simulação no Wokwi (ESP32 + DHT22), com verificação cruzada no Velxio.
Evidência do circuito em execução: `hardware/evidence/01-circuito-wokwi.png`

| Item solicitado | O que foi identificado |
|---|---|
| Microcontrolador | ESP32 (Espressif), dual-core, Wi-Fi e Bluetooth integrados |
| Memória | Flash externa (armazena firmware e configuração) e SRAM interna (execução) |
| Sensores | DHT22 (temperatura e umidade), ligado ao GPIO 15 |
| Interfaces de comunicação | Wi-Fi; protocolo MQTT sobre TCP, porta 1883 |
| Interfaces físicas | UART0 (TX/RX, monitor serial), JTAG (GPIO 12 a 15), SPI, I²C, USB |
| Alimentação | USB 5 V, regulada para 3,3 V na placa; o DHT22 usa o pino 3V3 |
| Mecanismos de armazenamento | Flash; credenciais e URL de atualização gravadas como constantes no firmware |
| Comunicação com outros sistemas | ESP32 → Wi-Fi → broker MQTT → plataforma de monitoramento da NexaHome |

## Fluxo de dados

Sensor DHT22 → ESP32 (firmware NexaSense v0.9.3-debug) → Wi-Fi → MQTT (1883, texto puro) → broker → plataforma.

Diagrama: `architecture/architecture-diagram.png`

Origem: o IP do ESP32 (10.10.0.2, que apareceu no monitor serial).
Destino: o IP do broker. Ele aparece na coluna Destination dos pacotes MQTT, e a consulta DNS mostra o nome broker.hivemq.com.
Porta: 1883.
Mensagens de controle: Connect, Connack, Publish, e talvez Ping Request/Response. Anote só as que aparecerem.
//////////////////////////////////////////////
//////////////////////////////////////////////
# Análise das comunicações – NexaSense (laboratório)

Ambiente: tráfego do ESP32 simulado no Wokwi, capturado em PCAP e analisado
no Wireshark. Evidência bruta: evidence/wokwi.pcap.
Escopo: apenas o tráfego do próprio dispositivo de laboratório, com dados fictícios.

## Resumo

| Item | Observação |
|---|---|
| Protocolo | MQTT sobre TCP, porta 1883 (Wi-Fi como enlace) |
| Origem | ESP32 simulado, 10.10.0.2 |
| Destino | broker.hivemq.com (IP visto na captura) |
| Tipo de informação | telemetria em JSON: temperatura, umidade, id do dispositivo, versão do firmware |
| Autenticação | existe usuário e senha no pacote CONNECT |
| Criptografia | nenhuma: sem TLS, conteúdo legível na captura |
| Identificadores expostos | device "NEXASENSE-0001", client ID, versão "0.9.3-debug" |
| Mensagens de controle | Connect, Connack, Publish (e outras que apareceram) |
| Dados sensíveis | usuário e senha MQTT em texto puro no CONNECT |

Evidências: evidence/01-mqtt-connect.png, 02-mqtt-publish.png, 03-dns-tcp.png

## Identificar o protocolo x julgar se é seguro
Identificar é reconhecer que o dispositivo usa MQTT na porta 1883. Isso é só
descrição. Julgar a segurança exige olhar como ele é usado: aqui o MQTT roda
sem TLS, e a credencial viaja em texto puro. Um atacante na mesma rede ou no
caminho poderia ler a senha e os dados e reutilizar a senha, que é a mesma em
todas as unidades.

## Análise
- **Confidencialidade:** ausente. Credencial e telemetria legíveis na captura.
- **Integridade:** não há assinatura nem verificação das mensagens.
- **Autenticação:** há usuário e senha, mas sem proteção do canal ela perde
  o valor; o dispositivo também não autentica o broker (sem certificado).
- **Exposição de identificadores:** id e versão do firmware fixos e visíveis.

## Limitações
- Laboratório simulado; o gateway do Wokwi é público e só permite conexões de saída.
- O broker é público e, por isso, provavelmente aceita qualquer credencial.
  As credenciais do teste são fictícias e o tópico é próprio.
- Não foram feitos testes contra serviços de terceiros além de publicar telemetria fictícia.

## Achados
- Credencial MQTT enviada sem criptografia (ver findings/findings.md).
- Telemetria sem proteção de confidencialidade ou integridade.

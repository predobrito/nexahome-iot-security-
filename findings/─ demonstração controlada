# Demonstração controlada

## Condições de controle
- Ambiente: dispositivo NexaSense simulado no Wokwi, projeto do próprio aluno.
- Credenciais e dados: fictícios, criados para o laboratório.
- Tópico MQTT: próprio (nexahome-lab/PEDRO0311/telemetry).
- Método: apenas observação do comportamento do dispositivo. Nenhum acesso a
  sistemas de terceiros, nenhuma exploração, nenhum acesso privilegiado.

---

## Demonstração A – Console de debug na UART expõe a configuração

**Fragilidade:** a interface serial entrega informação sensível sem
autenticação.

**Passos para reproduzir**
1. Abrir o projeto no Wokwi e iniciar a simulação.
2. Reiniciar a simulação e observar o monitor serial durante o boot.
3. Digitar `config` no campo de entrada do monitor e apertar Enter.
4. Digitar `ota` e apertar Enter.

**Resultado observado**
- No boot, o dispositivo imprime a configuração completa, incluindo as
  credenciais MQTT e a linha de log `MQTT connect user=... pass=...`.
- O comando `config` repete esses dados sem pedir senha.
- O comando `ota` mostra o endereço de atualização (HTTP).

**Evidências:** hardware/evidence/02b-uart-boot.png e 02-uart-config.png

**Impacto:** quem tiver acesso físico a uma unidade obtém uma credencial
que, por ser igual em todas as unidades, vale para toda a frota.

**Mitigação testável:** remover a impressão de credenciais e desativar o
console de debug em produção.

---

## Demonstração B – Credencial e telemetria trafegam sem proteção

**Fragilidade:** dado sensível transmitido sem criptografia.

**Passos para reproduzir**
1. Iniciar a simulação no Wokwi e aguardar cerca de 30 segundos.
2. Clicar no ícone de Wi-Fi e escolher "Download PCAP file".
3. Abrir o arquivo no Wireshark e aplicar o filtro `mqtt`.
4. Abrir o pacote "Connect Command" e expandir o protocolo MQTT.
5. Abrir um pacote "Publish Message".

**Resultado observado**
- O pacote CONNECT mostra usuário e senha em texto puro.
- O pacote PUBLISH mostra o tópico e o JSON com temperatura, umidade,
  identificador do dispositivo e versão do firmware.
- Não há TLS: a porta usada é a 1883.

**Evidências:** network/evidence/01-mqtt-connect.png e 02-mqtt-publish.png

**Impacto:** um atacante com acesso ao caminho da rede poderia ler a senha e
os dados e reutilizar a credencial.

**Mitigação testável:** MQTT sobre TLS (porta 8883) com credencial por dispositivo.

---

## Reprodutibilidade
As duas demonstrações podem ser repetidas por qualquer pessoa com o projeto
Wokwi e o Wireshark, seguindo os passos acima.

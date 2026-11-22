# Achados e mapeamento de riscos – NexaSense (laboratório)

Ambiente: dispositivo simulado (ESP32 + DHT22, Wokwi). Atacante considerado:
alguém com acesso físico a uma unidade ou com acesso ao caminho de rede.
Impacto e probabilidade são avaliações qualitativas do analista, sem medição
em hardware real. Demonstrações: findings/demo-controlada.md.

## Critério de classificação

| Impacto \ Probabilidade | Baixa | Média | Alta |
|---|---|---|---|
| Alto | Médio | Alto | Alto |
| Médio | Baixo | Médio | Médio |
| Baixo | Baixo | Baixo | Baixo |

## Tabela resumida

| Achado | Evidência | Impacto | Probabilidade | Risco | Recomendação |
|---|---|---|---|---|---|
| F01 | firmware/06-strings-busca.png | Credencial MQTT gravada no firmware, igual em todas as unidades: uma extração compromete a frota | Média | Alto | Credencial única por dispositivo, fora do firmware |
| F02 | hardware/evidence/02-uart-config.png, 02b-uart-boot.png | UART entrega credenciais e aceita comandos sem autenticação | Média | Alto | Remover logs sensíveis e desativar o console em produção |
| F03 | network/evidence/01-mqtt-connect.png, 02-mqtt-publish.png | Credencial e telemetria em texto puro na rede | Alta | Alto | MQTT sobre TLS (8883) com autenticação do servidor |
| F04 | firmware/06-strings-busca.png, 07-particoes-ota.png | Atualização por URL HTTP, sem indício de verificação de assinatura | Baixa | Médio | HTTPS e firmware assinado, com rollback |
| F05 | firmware/06-strings-busca.png | Build de debug e mensagens verbosas facilitam a engenharia reversa | Alta | Médio | Build de produção sem logs de debug |
| F06 | firmware/05-estrutura-bin.png, 07-particoes-ota.png | Imagem legível, com partições nvs e coredump sem proteção evidente | Baixa | Médio | Flash Encryption e Secure Boot |

## Detalhamento (cadeia de análise)

### F01 – Credencial MQTT embutida no firmware
- **Evidência:** `strings` na imagem encontra usuário e senha MQTT, identificador
  fixo do dispositivo e versão "0.9.3-debug".
- **Fragilidade:** segredo gravado no código, o mesmo em todas as unidades.
- **Ameaça:** atacante com acesso físico extrai a flash e lê a credencial.
- **Impacto:** a credencial vale para toda a frota; cada unidade comprometida
  compromete as demais.
- **Risco:** Alto (impacto alto, probabilidade média).
- **Mitigação:** credencial e identidade únicas por dispositivo, provisionadas
  na fabricação; Flash Encryption.

### F02 – Console de debug na UART sem autenticação
- **Evidência:** no boot, o dispositivo imprime a configuração e a linha
  `MQTT connect user=... pass=...`; o comando `config` repete esses dados.
- **Fragilidade:** interface física entrega informação sensível e aceita
  comandos sem controle de acesso.
- **Ameaça:** atacante com acesso físico liga um adaptador USB-serial e lê a credencial.
- **Impacto:** obtenção de uma credencial reutilizável (ver F01).
- **Risco:** Alto (impacto alto, probabilidade média).
- **Mitigação:** remover credenciais dos logs, desativar o console em produção.

### F03 – Comunicação sem criptografia
- **Evidência:** captura de rede mostra, no pacote CONNECT, usuário e senha em
  texto puro; o PUBLISH mostra o JSON da telemetria. Porta 1883, sem TLS.
- **Fragilidade:** ausência de confidencialidade e de autenticação do servidor.
- **Ameaça:** atacante no caminho da rede lê credencial e dados e pode
  reutilizar a credencial.
- **Impacto:** vazamento de credencial e de dados do ambiente monitorado.
- **Risco:** Alto (impacto alto, probabilidade alta).
- **Mitigação:** MQTT sobre TLS, validação de certificado, credencial por dispositivo.

### F04 – Atualização por HTTP
- **Evidência:** URL de atualização `http://...` no firmware; duas partições
  de aplicação e `otadata`.
- **Fragilidade:** canal de atualização sem criptografia e sem indício de
  verificação de assinatura.
- **Ameaça:** atacante na rede pode tentar entregar um firmware alterado.
- **Impacto:** execução de código não autorizado no dispositivo.
- **Risco:** Médio (impacto alto, probabilidade baixa: exige posição na rede
  e ausência de verificação, não comprovada neste laboratório).
- **Mitigação:** HTTPS, firmware assinado, Secure Boot, rollback seguro.

### F05 – Build de debug em uso
- **Evidência:** mensagens `[DEBUG]`, `CONFIG DUMP`, versão "0.9.3-debug".
- **Fragilidade:** informação interna disponível a quem observar o dispositivo.
- **Ameaça:** facilita reconhecimento e engenharia reversa.
- **Impacto:** médio, aumenta a superfície de ataque.
- **Risco:** Médio (impacto médio, probabilidade alta).
- **Mitigação:** build de produção, níveis de log, remoção de mensagens sensíveis.

### F06 – Flash sem proteção evidente
- **Evidência:** a imagem é legível por análise de textos; há partições `nvs`
  e `coredump` na tabela de partições.
- **Fragilidade:** sem indício de Flash Encryption; despejos de memória e
  configurações poderiam ficar legíveis.
- **Ameaça:** leitura da flash com acesso físico.
- **Impacto:** exposição de firmware, configurações e dados de falha.
- **Risco:** Médio (impacto alto, probabilidade baixa).
- **Mitigação:** Flash Encryption e Secure Boot; desabilitar o coredump em produção.

## Observação: interfaces não classificadas
JTAG e USB foram analisados apenas pela documentação e pelo pinout, sem
evidência no simulador de que estejam habilitados. Por isso não receberam
classificação de risco própria (ver hardware/interface-map.md).

## Relação com referências
- **ETSI EN 303 645:** F01 e F02 (credenciais e armazenamento seguro de
  parâmetros sensíveis), F03 (comunicação segura), F04 (atualizações e
  integridade do software), F05 (redução da superfície de ataque).
- **NIST IR 8259A:** F03 (proteção de dados), F02 (acesso lógico às
  interfaces), F04 (atualização de software), F01 (identificação do dispositivo).
- **MITRE EMB3D:** F06 (extração de firmware) e F02 (interface exposta).

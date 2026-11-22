Registro de preservação do firmware

Arquivo        : nexasense.ino.merged.bin
Tamanho        : <bytes> bytes
Origem         : compilado no Arduino IDE 2 (placa ESP32 Dev Module) a partir
                 do sketch nexasense.ino, o mesmo código simulado no Wokwi
Data da análise: <data e hora do Get-Date>
SHA-256        : <hash>

Cópia de trabalho: firmware/work/nexasense.ino.merged.bin
(original preservado em firmware/original/, somente leitura)

Observação: o binário foi compilado localmente a partir do mesmo código-fonte
usado no Wokwi. Não é byte a byte o arquivo que o simulador executou, pois o
editor web do Wokwi não permite exportar o firmware compilado.
//////////////////////////////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////////////////////////////
# Análise do firmware – nexasense.ino.merged.bin

Cópia de trabalho analisada: firmware/work (hash conferido em hashes.txt).
Método: leitura de bytes de estrutura e extração de textos legíveis com PowerShell.

## Formato e estrutura
- Imagem completa do ESP32 de 4 MiB.
- Bootloader em 0x1000, tabela de partições em 0x8000, aplicação em 0x10000.
- Evidência: 05-estrutura-bin.png

## Achados

| ID | O que foi encontrado | Evidência | Por que importa |
|---|---|---|---|
| F-FW1 | Usuário e senha MQTT gravados em texto no firmware | 06-strings-busca.png | Quem extrair a flash lê a credencial; como é igual em todas as unidades, compromete a frota |
| F-FW2 | URL de atualização usando HTTP | 06-strings-busca.png | Sem criptografia, a atualização pode ser interceptada ou trocada |
| F-FW3 | Broker MQTT e tópico fixos no código | 06-strings-busca.png | Revela o destino dos dados e o padrão de nomes de tópico |
| F-FW4 | Mensagens [DEBUG] e CONFIG DUMP | 06-strings-busca.png | Facilita engenharia reversa e vaza configuração |
| F-FW5 | Identificador e versão fixos (NEXASENSE-0001, 0.9.3-debug) | 06-strings-busca.png | Identidade igual em todas as unidades; versão "debug" em produção |
| F-FW6 | Partições otadata/app0/app1 | 07-particoes-ota.png | Atualização OTA prevista; precisa de validação de assinatura |

## Observação sobre dados sensíveis
As credenciais são fictícias, criadas para o laboratório. Neste arquivo
o valor da senha não é reproduzido; ele aparece apenas nos prints de evidência.

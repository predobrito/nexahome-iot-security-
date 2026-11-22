# Plano de hardening – NexaSense (versão comercial)

Escopo: proposta conceitual. Nada abaixo foi gravado em hardware; em especial,
nenhum mecanismo irreversível (eFuses) foi acionado.

## Princípios
1. Nenhum segredo no firmware e nenhuma credencial igual entre unidades.
2. Nada sai em texto puro pela rede.
3. Só firmware assinado executa e só firmware assinado é instalado.
4. Interfaces de depuração fechadas em produção.
5. Toda falha de atualização tem caminho de recuperação.

## Medidas por achado

| Achado | Medida | Mecanismo (ESP32 / padrão) | Prioridade |
|---|---|---|---|
| F01 Credencial no firmware | Credencial e identidade únicas por dispositivo, geradas e gravadas na fabricação, fora do código | Provisionamento em produção; armazenamento em NVS criptografado; Flash Encryption | P1 |
| F02 UART expõe configuração | Remover credenciais dos logs; desativar o console de comandos em produção | Build de produção sem o console; nível de log mínimo | P1 |
| F03 MQTT sem criptografia | MQTT sobre TLS (porta 8883), validação do certificado do servidor, preferencialmente autenticação mútua com certificado por dispositivo | esp-tls / mbedTLS com pacote de certificados | P1 |
| F05 Build de debug | Build de produção, versão sem sufixo "debug", sem mensagens internas | Configuração de log por nível; revisão de strings antes do release | P1 |
| F04 Atualização por HTTP | HTTPS para atualização, firmware assinado, verificação antes de instalar | OTA sobre HTTPS; Secure Boot; rollback do bootloader | P2 |
| F06 Flash sem proteção | Criptografar a flash e exigir assinatura na inicialização; desativar coredump em flash | Flash Encryption e Secure Boot da Espressif | P2 |

**P1:** antes do projeto-piloto com clientes (correções de baixo custo e alto efeito).
**P2:** antes da versão comercial (exigem fluxo de produção e planejamento de chaves).

## Interfaces físicas
- **JTAG:** desabilitar em produção. No ESP32 isso é feito por eFuse e é
  irreversível, então deve ser decidido com a linha de produção definida.
- **UART:** manter apenas se houver necessidade de manutenção, sem entregar
  informação sensível e sem comandos sem autenticação; considerar restringir
  o modo de gravação pela porta serial.
- **USB / gravação:** com Secure Boot, firmware não assinado não inicia,
  mesmo que alguém consiga gravá-lo.
- **Pontos de teste e pinos não usados:** remover ou isolar no projeto da placa.

## Atualização segura e recuperação
1. Imagem assinada na fábrica ou no pipeline de build.
2. Dispositivo baixa por HTTPS, verifica a assinatura e só então grava na
   partição inativa (app0/app1, já existentes na tabela de partições).
3. Na primeira inicialização, a nova versão precisa se confirmar; se falhar,
   o bootloader volta à versão anterior (rollback).
4. Proteção contra downgrade para versões vulneráveis.
5. Registro do resultado da atualização para acompanhamento da frota.

## Chaves e segredos
- Chave de assinatura do firmware guardada fora do dispositivo, com acesso
  restrito e processo de rotação.
- Certificados e chaves por dispositivo gravados no provisionamento, nunca
  no repositório de código.

## Medidas irreversíveis (apenas propostas)
Secure Boot, Flash Encryption e desativação do JTAG usam eFuses, que não
podem ser desfeitos. Por isso, foram analisados conceitualmente e devem ser
aplicados somente em unidades de produção, após validação em placas de
desenvolvimento.

## Como verificar cada medida (critérios de aceitação)
| Medida | Teste |
|---|---|
| Sem segredo no firmware | análise de textos na imagem não encontra credenciais |
| UART sem vazamento | boot e console não mostram credenciais |
| TLS ativo | captura de rede não mostra usuário, senha nem dados legíveis |
| Atualização assinada | imagem alterada é rejeitada |
| Rollback | atualização defeituosa volta à versão anterior |
| Flash protegida | leitura da flash não revela texto legível |

## Relação com as referências
- **ETSI EN 303 645:** credenciais únicas, armazenamento seguro de parâmetros,
  comunicação segura, atualizações e integridade do software, redução da
  superfície de ataque.
- **NIST IR 8259A:** identificação do dispositivo, proteção de dados, acesso
  lógico às interfaces, atualização de software.
- **Espressif (ESP-IDF Security):** Secure Boot e Flash Encryption como
  mecanismos complementares de proteção do software armazenado.

## Estado nesta avaliação
Não implementado: plano conceitual. Demonstração opcional de mitigação:
remover as impressões de credencial no firmware simulado e repetir a
Demonstração A.

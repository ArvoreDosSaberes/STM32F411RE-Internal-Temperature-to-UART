# Planejamento do projeto

## Roadmap

- **Fase 1** – Projeto base CubeMX + CMake + UART funcional.
- **Fase 2** – Leitura do sensor interno de temperatura via ADC.
- **Fase 3** – Conversão para graus Celsius e envio via UART.
- **Fase 4** – Documentação completa em `docs/`.

## Riscos

- Configuração incorreta do ADC (tempo de amostragem inadequado) afetando
  a precisão da leitura.
- Clock mal configurado impactando o baud rate da UART.

## Plano de testes

- Teste manual: observar saída de temperatura em terminal serial.
- Teste de variação: aquecer levemente o microcontrolador e observar aumento
  gradual da leitura.

## Plano de rollout/rollback

- Rollout: gravação do firmware via `st-flash`.
- Rollback: regravar versão anterior conhecida (mantida como `.bin`/`.elf`).

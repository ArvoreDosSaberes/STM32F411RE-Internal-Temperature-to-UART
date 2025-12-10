# STM32F411RE Internal Temperature to UART

![Visitors](https://visitor-badge.laobi.icu/badge?page_id=ArvoreDosSaberes.STM32F411RE-Internal-Temperature-to-UART)[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![MCU](https://img.shields.io/badge/MCU-STM32F411RE-blue.svg)](https://www.st.com/en/microcontrollers-microprocessors/stm32f411.html)
[![Family](https://img.shields.io/badge/Family-STM32F4-blue.svg)](https://www.st.com/en/microcontrollers-microprocessors/stm32f4-series.html)
[![HAL](https://img.shields.io/badge/Framework-STM32Cube%20HAL-brightgreen.svg)](https://www.st.com/en/embedded-software/stm32cube-mcu-packages.html)
[![Build](https://img.shields.io/badge/Build-CMake%20%2B%20Ninja-orange.svg)](https://cmake.org/)
[![Language](https://img.shields.io/badge/Language-C-blue.svg)](https://en.wikipedia.org/wiki/C_(programming_language))
[![GitHub](https://img.shields.io/github/stars/ArvoreDosSaberes/STM32F411RE-Internal-Temperature-to-UART?style=social)](https://github.com/ArvoreDosSaberes/STM32F411RE-Internal-Temperature-to-UART)
[![GitHub Issues](https://img.shields.io/github/issues/ArvoreDosSaberes/STM32F411RE-Internal-Temperature-to-UART)](https://github.com/ArvoreDosSaberes/STM32F411RE-Internal-Temperature-to-UART/issues)

> Projeto de exemplo com STM32F411RE que lê a **temperatura interna** do
> microcontrolador via **ADC** e envia a leitura em graus Celsius pela
> **UART** usando `printf`/`HAL_UART_Transmit`.

## Visão geral

- **MCU alvo**: STM32F411RE (linha STM32F4 da STMicroelectronics).
- **Função principal**: ler o sensor de temperatura interno (`ADC_CHANNEL_TEMPSENSOR`)
  e transmitir periodicamente o valor em °C pela USART2.
- **Stack de firmware**: STM32Cube HAL, projeto gerado a partir de `.ioc`.
- **Build system**: CMake + Ninja (via `CMakePresets.json`).
- **Deploy**: linha de comando com `st-flash`.

> Apesar dos _badges_ referenciarem um outro repositório de exemplo
> (`esp-i2c-scanner`), eles são mantidos aqui como modelo visual, conforme
> padrão definido pelo autor.

## Como compilar e gravar

O fluxo recomendado de build e deploy está descrito em detalhes em:

- `docs/how-to/deploy.md`

Resumo (preset **Debug**):

```bash
cmake --preset Debug
cmake --build --preset Debug

arm-none-eabi-objcopy \
  -O binary \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.elf \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.bin

st-flash write \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.bin \
  0x08000000
```

## Documentação

A documentação do projeto fica concentrada em `docs/`:

- `docs/README.md` – índice geral da documentação.
- `docs/REQUIREMENTS.md` – requisitos funcionais e não funcionais.
- `docs/PLAN.md` – planejamento, roadmap e riscos.
- `docs/CHANGELOG.md` – histórico de versões e mudanças.
- `docs/architecture.md` – arquitetura do firmware, fluxo de dados e
  **matemática do cálculo da temperatura interna**.
- `docs/how-to/*.md` – tutoriais passo a passo (build, testes, deploy, etc.).
- `docs/guidelines/*.md` – padrões de código, logging e segurança.

## Licença

Este projeto está licenciado sob os termos da
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

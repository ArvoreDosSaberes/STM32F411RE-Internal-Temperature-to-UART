# Requisitos do projeto

## Contexto e objetivos

Este projeto demonstra como ler a **temperatura interna** do microcontrolador
STM32F411RE usando o **ADC** e transmitir a leitura em graus Celsius pela
USART2 para um terminal serial.

## Requisitos funcionais (RF)

- **RF-001** – Ler periodicamente o valor do sensor interno de temperatura
  (`ADC_CHANNEL_TEMPSENSOR`).
- **RF-002** – Converter o valor bruto do ADC em temperatura em graus Celsius.
- **RF-003** – Enviar a temperatura formatada pela USART2 em intervalos
  configuráveis (ex.: 1 segundo).
- **RF-004** – Permitir monitorar a temperatura interna em um terminal serial
  no PC.

## Requisitos não funcionais (RNF)

- **RNF-001** – Firmware baseado em HAL/CubeMX para STM32F4.
- **RNF-002** – Build reproduzível via **CMake** + presets (`Debug`/`Release`).
- **RNF-003** – Deploy via linha de comando usando `st-flash`.
- **RNF-004** – Código comentado em português, com foco didático.

## Restrições

- Uso do microcontrolador **STM32F411RE**.
- Uso do sensor de temperatura interno apenas para **monitoramento relativo**
  (não é um termômetro industrial de alta precisão).

## Critérios de aceite

- Projeto compila em pelo menos um ambiente Linux com toolchain ARM instalada.
- Dispositivo grava corretamente via `st-flash`.
- Terminal serial exibe leituras de temperatura variando conforme aquecimento
  do chip.

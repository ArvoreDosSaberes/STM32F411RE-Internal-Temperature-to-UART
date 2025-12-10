# Arquitetura e conceitos do projeto

## Visão geral

O firmware é organizado segundo a estrutura padrão do STM32CubeMX:

- `Core/` – código de aplicação (`main.c`, inicialização de periféricos).
- `Drivers/` – bibliotecas HAL/LL fornecidas pela ST.
- `cmake/` – arquivos de integração CMake e toolchain ARM.

O ponto central do projeto é a leitura do **sensor interno de temperatura**
via **ADC1** e o envio periódico da temperatura calculada pela **USART2**.

## Fluxo principal

1. Inicialização de **clock**, **GPIO**, **ADC1** e **USART2**.
2. Em `while(1)`, chamada periódica de `print_temperature()`.
3. `print_temperature()` chama `read_internal_temperature()` para obter a
   leitura atual em °C e envia o texto formatado pela UART.

## Matemática do cálculo da temperatura interna

O sensor interno de temperatura do STM32F411 produz uma tensão `Vsense`
proporcional à temperatura do silício. A ST fornece uma fórmula aproximada:

\[ T(°C) = \left(\frac{V_{sense} - V_{25}}{Avg\_Slope}\right) + 25 \]

Onde:

- `Vsense` – tensão medida no canal do sensor de temperatura.
- `V25` – tensão típica do sensor a 25 °C (aprox. **0,76 V**).
- `Avg_Slope` – inclinação média da curva (aprox. **2,5 mV/°C** = 0,0025 V/°C).

### 1. Convertendo o valor do ADC em tensão

O ADC está configurado para **12 bits**, logo o valor bruto (`ADC_Value`)
varia de 0 a 4095. Com referência de 3,3 V, temos:

\[ V_{sense} = \frac{ADC\_Value \times V_{ref}}{4095} \]

No código:

```c
Vsense = (raw * 3.3f) / 4095.0f;
```

### 2. Aplicando a fórmula de temperatura

Com `Vsense` calculado, aplicamos a fórmula da ST:

```c
const float V25 = 0.76f;      // tensão típica a 25 °C
const float Avg_Slope = 0.0025f; // 2,5 mV/°C

temperature = ((Vsense - V25) / Avg_Slope) + 25.0f;
```

Interpretação passo a passo:

1. `Vsense - V25` – diferença de tensão em relação a 25 °C.
2. `(Vsense - V25) / Avg_Slope` – converte essa diferença de tensão em
   **variação de temperatura** (°C) relativa a 25 °C.
3. `+ 25.0f` – desloca o resultado para a escala absoluta de temperatura.

### 3. Exemplo numérico simplificado

Suponha um valor de ADC que resulte em `Vsense = 0,80 V`.

1. Diferença em relação a 25 °C:

   \[ \Delta V = 0{,}80 - 0{,}76 = 0{,}04\,V \]

2. Variação de temperatura:

   \[ \Delta T = \frac{0{,}04}{0{,}0025} = 16\,°C \]

3. Temperatura absoluta:

   \[ T = 16 + 25 = 41\,°C \]

Ou seja, o núcleo do microcontrolador estaria em torno de **41 °C**.

## Considerações de precisão

- Os valores de `V25` e `Avg_Slope` são **típicos** e variam entre chips.
- A leitura é adequada para **monitoramento térmico relativo**, não para
  medições de laboratório de alta precisão.
- Para maior exatidão, é possível calibrar empiricamente, medindo a saída
  em duas temperaturas conhecidas e ajustando os parâmetros.

## UART e formato da mensagem

A função `print_temperature()` formata a saída como texto simples, por
exemplo:

```text
Temperatura Interna: 32.45 C
```

Isso facilita a visualização em qualquer terminal serial e também permite
que ferramentas de logging capturem o valor para análise posterior.

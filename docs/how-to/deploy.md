# Deploy pela linha de comando (st-flash)

Este documento explica como **compilar** e **gravar** o firmware do projeto
`STM32F411RE-Internal-Temperature-to-UART` pela **linha de comando**, usando
os **presets do CMake** (`CMakePresets.json`) e a ferramenta `st-flash`
(STLink).

> Tempo estimado: **10–15 minutos** para configurar tudo pela primeira vez,
> depois disso o ciclo de build+flash leva poucos segundos.

---

## 1. Pré-requisitos

- **Toolchain ARM**:
  - `arm-none-eabi-gcc`
  - `arm-none-eabi-gdb`
  - `arm-none-eabi-binutils` (inclui `arm-none-eabi-objcopy`)
- **CMake** (3.16+ recomendado)
- **Make** (ou Ninja, se preferir)
- **stlink-tools** (para o `st-flash`):
  - Em muitas distros: `stlink-tools` ou `stlink` via gerenciador de pacotes
- **Permissão de acesso USB** ao ST-LINK:
  - No Linux, normalmente precisa estar em um grupo com permissão (por
    exemplo `plugdev` ou `dialout`) ou configurar regras `udev`.

Exemplo (Ubuntu/Debian, apenas como referência):

```bash
sudo apt-get install cmake make gcc-arm-none-eabi binutils-arm-none-eabi stlink-tools
```

> **Atenção**: adapte os nomes dos pacotes à sua distribuição. Em algumas,
> o pacote da toolchain pode chamar `gcc-arm-none-eabi` ou similar, e o
> pacote do ST-LINK pode se chamar apenas `stlink`.

---

## 2. Estrutura básica do projeto

Na raiz do repositório você deve ter (entre outros):

- `CMakeLists.txt`
- `STM32F411RE-Internal-Temperature-to-UART.ioc`
- Diretórios `Core/`, `Drivers/`, `cmake/`, etc.

O build será feito dentro da pasta `build/` (já existe, mas você pode
recriá-la se quiser um build limpo).

---

## 3. Gerando os arquivos de build (CMake Presets)

O projeto já vem configurado com **presets** no arquivo
`CMakePresets.json`, usando o gerador **Ninja** e a toolchain
`cmake/gcc-arm-none-eabi.cmake`.

- Presets de configuração:
  - `Debug`
  - `Release`

Cada preset gera os arquivos de build em um diretório próprio:

- `build/Debug` para o preset **Debug**
- `build/Release` para o preset **Release**

### 3.1. Configurar o preset Debug

Na raiz do projeto:

```bash
cmake --preset Debug
```

### 3.2. Configurar o preset Release

Na raiz do projeto:

```bash
cmake --preset Release
```

> Você só precisa rodar o `cmake --preset ...` novamente se mudar
> configurações importantes do projeto (como o `CMakeLists.txt`).

---

## 4. Compilando o firmware

Após configurar o preset desejado (Debug ou Release):

### 4.1. Compilar Debug

```bash
cmake --build --preset Debug
```

### 4.2. Compilar Release

```bash
cmake --build --preset Release
```

Ao final da compilação, você deve obter pelo menos um arquivo `.elf`.
Para o preset **Debug**, o ELF ficará em algo como:

```text
build/Debug/
  STM32F411RE-Internal-Temperature-to-UART.elf
  ...
```

> Se o nome do ELF for diferente na sua configuração, ajuste nos
> comandos do passo seguinte.

---

## 5. Gerando o arquivo .bin a partir do .elf

Para gravar com `st-flash`, é comum usar um **arquivo binário cru (.bin)**.
Geramos o `.bin` a partir do `.elf` com `arm-none-eabi-objcopy`.

No terminal, na raiz do projeto, usando o ELF gerado em **Debug**:

```bash
arm-none-eabi-objcopy \
  -O binary \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.elf \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.bin
```

- `-O binary` → saída em formato binário puro.
- Entrada: o `.elf` gerado pela compilação.
- Saída: o `.bin` que será gravado na flash.

> Dica: você pode colocar esse comando em um script `Makefile`/`CMake`
> customizado ou em um script shell (`build_and_flash.sh`) para facilitar.

---

## 6. Gravando na flash com st-flash

Com o `.bin` gerado, conecte a placa STM32F411 (Nucleo/Discovery ou outra)
via ST-LINK ao PC.

O endereço de início da flash interna do **STM32F411RE** é:

- **0x08000000**

O comando básico do `st-flash` para gravar o binário gerado em **Debug** é:

```bash
st-flash write \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.bin \
  0x08000000
```

Explicação:

- `write` → modo de gravação.
- Primeiro parâmetro → caminho do arquivo `.bin`.
- Segundo parâmetro → endereço inicial da memória flash onde o binário será escrito.

Se tudo estiver correto, você verá uma saída semelhante a:

```text
st-flash 1.7.0
2025-..-..T..:..:.. INFO common.c: Loading device parameters....
INFO common.c: Device connected is: F411...
INFO common.c: Flash size        : 512 KiB
INFO common.c: Device UID        : ...
INFO common.c: Flash written and verified! jolly good!
```

Após a gravação, a placa normalmente **reinicia automaticamente** e começa a
executar o firmware (no seu caso, imprimindo a temperatura interna na USART2).

---

## 7. Verificando a saída serial

1. Conecte o cabo USB que expõe a UART (normalmente USART2) ao PC.
2. Descubra o dispositivo serial criado (exemplos típicos):
   - `/dev/ttyUSB0`
   - `/dev/ttyACM0`
3. Abra um terminal serial (exemplos: `screen`, `minicom`, `picocom`):

   ```bash
   screen /dev/ttyACM0 115200
   ```

   ou

   ```bash
   minicom -D /dev/ttyACM0 -b 115200
   ```

4. Você deverá ver linhas como:

   ```text
   Temperatura Interna: 32.45 C
   Temperatura Interna: 32.62 C
   ...
   ```

> Lembre-se de usar **115200 8N1**, sem paridade e sem controle de fluxo.

---

## 8. Workflow resumido (build + flash, usando preset Debug)

A partir do momento em que o preset **Debug** já foi configurado uma vez
(`cmake --preset Debug`):

```bash
# 1) Compilar (Debug)
cmake --build --preset Debug

# 2) Gerar binário (a partir do ELF em build/Debug)
arm-none-eabi-objcopy \
  -O binary \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.elf \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.bin

# 3) Gravar com st-flash
st-flash write \
  build/Debug/STM32F411RE-Internal-Temperature-to-UART.bin \
  0x08000000
```

Opcionalmente, você pode colocar tudo isso em um **script shell** ou em
um **target customizado de CMake** para rodar com um único comando.

---

## 9. Solução de problemas comuns

- **`st-flash: Failed to connect to target`**
  - Verifique se a placa está ligada e bem conectada via USB.
  - Confira se não há outro programa usando o ST-LINK (por exemplo,
    um IDE aberto com sessão de debug ativa).
  - No Linux, veja se você tem permissão de acesso ao dispositivo
    (`lsusb`, `dmesg`, grupos de usuário, regras `udev`).

- **Erro de permissão (Permission denied) ao acessar /dev/ttyUSB0 ou /dev/ttyACM0**
  - Inclua seu usuário no grupo apropriado (ex.: `dialout`):

    ```bash
    sudo usermod -aG dialout "$USER"
    # faça logoff/login após o comando
    ```

- **Erro na compilação (headers HAL não encontrados, etc.)**
  - Confirme se você está rodando o `cmake` e o `cmake --build` na **raiz
    correta do projeto**, onde está o `CMakeLists.txt` principal.
  - Verifique se a toolchain ARM está instalada e no `PATH`.

- **Nada aparece no terminal serial**
  - Confirme que está usando a **porta certa** (`/dev/ttyUSB0` x `/dev/ttyACM0`).
  - Confirme o baud rate (115200) e parâmetros 8N1.
  - Garanta que o firmware compilou e foi gravado sem erros.
  - Se necessário, adicione mais mensagens via UART para debug.

---

Com isso, você consegue manter um fluxo de desenvolvimento totalmente
baseado em **linha de comando**, sem depender do IDE para compilar e
gravar o STM32F411RE.

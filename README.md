# 📌 Sistema Integrado de Monitoramento Ambiental com Arduino

Este projeto é um sistema de monitoramento automático de temperatura, umidade e luminosidade, utilizando sensores, comunicação I²C, alertas visuais e sonoros, armazenamento de registros na EEPROM e exibição em display LCD 16x2 – tudo isso com uma interface **gamificada** que possibilita a troca de personagens durante a navegação dos menus.

---
### Personagens Disponíveis

- 👨‍🚀 Among Us (padrão)
- 🧙 Maguinho
- 🎃 Abóbora
- 🧩 Cefsa
- ➡️ Setinha
- 🎈 Balão
- ⚫ Bolinha
- 🧱 Cubo
- 🧍 Stickman
- 👻 Fantasma
- **Troca de personagem:** feita no menu inicial ao selecionar o ícone de configurações (posição `(11,1)`) e pressionar o botão **Selecionar**.


## 📝 Especificações Técnicas

### 🔧 Hardware Utilizado

- **Microcontrolador:** Arduino Uno/Nano/compatível
- **Display:** LCD 16x2 com módulo I²C (Endereço: `0x27`)
- **Sensor de Temperatura e Umidade:** DHT22 (Digital 9)
- **Sensor de Luminosidade:** LDR (conectado ao pino analógico A0)
- **Módulo RTC:** DS3231 (comunicação I²C)
- **Memória:** EEPROM interna para registro de dados
- **LEDs Indicadores:** LED Verde (pino 10) e LED Amarelo (pino 11)
- **Buzzer:** Pino 8 para alertas sonoros
- **Botões:**  
  - **Avançar:** Digital 5  
  - **Selecionar:** Digital 6  
  - **Voltar:** Digital 7

---

### 🔌 Conexões e Pinagem

| Componente         | Pino Arduino              |
|--------------------|---------------------------|
| LCD I²C            | SDA (A4), SCL (A5)        |
| DHT22              | Digital 9                 |
| LDR                | Analógico A0              |
| RTC DS3231         | SDA (A4), SCL (A5)        |
| Botão Avançar      | Digital 5                 |
| Botão Selecionar   | Digital 6                 |
| Botão Voltar       | Digital 7                 |
| LED Verde          | Digital 10                |
| LED Amarelo        | Digital 11                |
| Buzzer             | Digital 8                 |

---

### Diagrama Elétrico

![DIAGRAMA ELETRICO 2 0](https://github.com/user-attachments/assets/3bc87457-ac44-42ed-b805-9a68a2ade254)


---

## 📖 Manual de Operação

### ✅ Inicialização

- Ao ligar o dispositivo, o sistema executa animações iniciais (funções como `wizard1()`, `wizard2()` e `magic()`) que exibem personagens e efeitos sonoros.
- Após as animações, o usuário é direcionado ao **menu principal gamificado**.

### 📌 Navegação dos Menus

Utilize os botões para interagir com o sistema:
- **Botão Avançar (Digital 5):** Move o personagem para a direita.
- **Botão Voltar (Digital 7):** Move o personagem para a esquerda.
- **Botão Selecionar (Digital 6):** Confirma a seleção da opção atual.

### 🔖 Opções Disponíveis no Menu

- **ESCALA TEMP:** Permite selecionar a unidade de medida da temperatura (Celsius, Fahrenheit ou Kelvin).
- **HOME:** Exibe as leituras atuais de temperatura, umidade e luminosidade.
- **RTC:** Exibe a data e hora atuais do módulo RTC.
- **Configurações de Personagem:** Permite alternar entre diferentes personagens (Among Us, Maguinho, Abóbora, entre outros) para uma experiência interativa e lúdica.

---

## 🌡️ Escalas de Temperatura

O sistema suporta três escalas de temperatura, podendo ser selecionadas através do menu **ESCALA TEMP**:

1. **Celsius (°C)**
   - Ponto de congelamento da água: 0 °C  
   - Ponto de ebulição da água: 100 °C  
   - Escala padrão utilizada no sistema.

2. **Fahrenheit (°F)**
   - Ponto de congelamento da água: 32 °F  
   - Ponto de ebulição da água: 212 °F  
   - Conversão:  
     \[ °F = (°C \times \frac{9}{5}) + 32 \]

3. **Kelvin (K)**
   - Escala absoluta (não utiliza o símbolo de grau)  
   - Zero absoluto: 0 K (equivalente a -273,15 °C)  
   - Conversão:  
     \[ K = °C + 273.15 \]

---

## 🚨 Alertas e Limites Pré-definidos

Os alertas são acionados automaticamente quando os valores medidos se encontram fora dos limites definidos:

| Parâmetro        | Limite Mínimo | Limite Máximo | Indicador             | Unidade de Medida            |
|------------------|---------------|---------------|-----------------------|------------------------------|
| **Temperatura**  | 20 °C         | 25 °C         | LED Amarelo + Buzzer  | (conforme a escala escolhida)|
| **Umidade**      | 30 %          | 60 %          | LED Amarelo + Buzzer  | % de Umidade Relativa        |
| **Luminosidade** | Variável*     | -             | LED Amarelo + Buzzer  | Percentual calculado         |

> *A faixa de luminosidade pode variar conforme as condições ambientais.

---

## 💾 Registro Automático de Anomalias (EEPROM)

- Condições que ultrapassem os limites definidos são registradas automaticamente na EEPROM.
- Essa funcionalidade permite o armazenamento e análise posterior dos dados registrados.

---

## 💻 Código Fonte Comentado

### 📍 Bibliotecas e Definições Principais

```cpp
#include <Wire.h>               // Comunicação I²C
#include <LiquidCrystal_I2C.h>  // LCD com interface I²C
#include <RTClib.h>             // Manipulação do RTC DS3231
#include <DHT.h>                // Leitura do sensor DHT22
#include <EEPROM.h>             // Armazenamento persistente
```

## 📍 Estrutura do Código
Animações e Introdução: Funções como wizard1(), wizard2() e magic() criam uma introdução animada com personagens.
Gerenciamento do Display: Funções e classes para gerenciar a exibição dos menus e animações gamificadas.
Leitura de Sensores: Realiza leituras periódicas (a cada 5 segundos) dos sensores DHT22 (temperatura e umidade) e LDR (luminosidade).
Alertas: Ativação do LED Amarelo e buzzer quando os valores medidos estiverem fora dos limites.
Interação com o Usuário: Navegação entre menus e troca de personagem para uma experiência interativa.
Registro de Dados: Armazena automaticamente os dados de anomalias na EEPROM.




## 🛠️ Bibliotecas Necessárias
Instale as seguintes bibliotecas via Gerenciador de Bibliotecas do Arduino IDE:
LiquidCrystal_I2C
RTClib
DHT sensor library
EEPROM (biblioteca padrão da IDE do Arduino)



## 💬 Código Fonte (Resumo com Comentários)

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <RTClib.h>
#include <DHT.h>
#include <EEPROM.h>

LiquidCrystal_I2C lcd(0x27, 16, 2); // Configuração do LCD 16x2

// Variáveis globais
const int LDR_PIN = A0; // Pino onde o LDR está conectado
int valorLDR = 0;       // Valor lido do LDR
int lightLevel = 0;     // Valor mapeado (0-100)
bool mostrarPercentual = true; // Controla se exibe percentual ou valor bruto
unsigned long ultimaLeitura = 0; // Armazena o tempo da última leitura
const unsigned long intervaloLeitura = 5000; // Intervalo de 5 segundos

RTC_DS3231 rtc; // RTC

// Enum para gerenciar as telas
enum TelaAtual {
  TELA_HOME,
  TELA_LUMINOSIDADE,
  TELA_DATAHORA,
  TELA_UMIDADE,
  TELA_TEMPERATURA
};

TelaAtual telaAtual = TELA_HOME; // Tela inicial

// DHT Sensor
#define DHTPIN 9
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

float temp = 0.0;    // Temperatura atual
float humid = 0.0;   // Umidade atual

// Endereços da EEPROM
const int addrLDR = 0;
const int addrTemp = sizeof(int);
const int addrHumid = addrTemp + sizeof(float);

// Forward declarations
class Tela;
class TelaHome;
class TelaLuminosidade;
class Figura;
class Display;

// ---- CLASSE FIGURA ---- //
class Figura {
  private:
    String nome;
    byte pixeisFigura[8];
    int indice;

  public:
    Figura(String nome, byte pixeis[8], int indice) {
      this->nome = nome;
      memcpy(this->pixeisFigura, pixeis, sizeof(pixeisFigura));
      this->indice = indice;
    }

    String getNome() { return nome; }
    byte* getPixeis() { return pixeisFigura; }
    int getIndice() { return indice; }
};

// ---- CLASSE DISPLAY ---- //
class Display {
  private:
    int personagemX;
    int personagemY;
    bool personagemEspelhado;

  public:
    Display() : personagemX(1), personagemY(1), personagemEspelhado(false) {}

    byte* espelharFigura(Figura figura) {
      static byte byteEspelhado[8];
      byte* original = figura.getPixeis();

      for (int i = 0; i < 8; i++) {
        byte linha = original[i];
        byte novoByte = 0;
        for (int j = 0; j < 5; j++) {
          if (linha & (1 << j)) {
            novoByte |= (1 << (4 - j));
          }
        }
        byteEspelhado[i] = novoByte;
      }

      return byteEspelhado;
    }

    void imprimirObjeto(Figura figura, int x, int y) {
      lcd.createChar(figura.getIndice(), figura.getPixeis());
      lcd.setCursor(x, y);
      lcd.write(figura.getIndice());
    }

    void imprimirFigura(Figura figura, int x, int y, bool espelhar) {
      byte* pixeis = espelhar ? espelharFigura(figura) : figura.getPixeis();
      Figura figuraTemp(figura.getNome(), pixeis, figura.getIndice());
      imprimirObjeto(figuraTemp, x, y);
    }

    void deslizarFigura(Figura figura, int xInicio, int xFinal, int y) {
      int diferenca = xFinal - xInicio;
      bool paraEsquerda = diferenca < 0;
      int direcao = paraEsquerda ? -1 : 1;

      if (figura.getNome() == "AmongUs") {
        personagemEspelhado = paraEsquerda;
      }

      imprimirFigura(figura, xInicio, y, paraEsquerda);

      for (int x = xInicio; paraEsquerda ? x > xFinal : x < xFinal; x += direcao) {
        lcd.setCursor(x, y);
        lcd.write(' ');
        int novaPos = x + direcao;
        imprimirFigura(figura, novaPos, y, paraEsquerda);
        delay(200);
      }

      if (figura.getNome() == "AmongUs") {
        personagemX = xFinal;
      }
    }

    void limparCelula(int x, int y) {
      lcd.setCursor(x, y);
      lcd.write(' ');
    }

    void limparColunaCima() {
      for (int x = 0; x < 16; x++) {
        lcd.setCursor(x, 0);
        lcd.write(' ');
      }
      limparCelula(0, 1);
      limparCelula(15, 1);
    }

    void pularFigura(Figura figura, int x, int y, bool espelhar) {
      byte* pixeis = espelhar ? espelharFigura(figura) : figura.getPixeis();
      byte figuraPulando[8];

      for (int i = 0; i < 7; i++) {
        figuraPulando[i] = pixeis[i + 1];
      }
      figuraPulando[7] = B00000;

      Figura figuraTemp(figura.getNome(), figuraPulando, figura.getIndice());
      imprimirObjeto(figuraTemp, x, y);
      delay(600);

      byte* pixeisRestaurados = espelhar ? espelharFigura(figura) : figura.getPixeis();
      Figura figuraRestaurada(figura.getNome(), pixeisRestaurados, figura.getIndice());
      imprimirObjeto(figuraRestaurada, x, y);
      delay(300);
    }

    int getPersonagemX() { return personagemX; }
    int getPersonagemY() { return personagemY; }
    bool getPersonagemEspelhado() { return personagemEspelhado; }
};

// ---- CLASSE ABSTRATA TELA ---- //
class Tela {
  protected:
    int destinoAtual = 0; // Destino atual do personagem

  public:
    virtual void imprimirTela(Display& display) = 0;

    void moverDestino(Display& display, Figura& personagem, int destino, const int* destinos, int numDestinos) {
      if (destino < 0 || destino >= numDestinos) return; // Verifica se o destino é válido

      int xAtual = display.getPersonagemX();
      int yAtual = display.getPersonagemY();
      int xFinal = destinos[destino]; // Pega o ponto de parada correspondente

      display.deslizarFigura(personagem, xAtual, xFinal, yAtual);
      destinoAtual = destino;
    }

    virtual void selecionar(Display& display, Figura& personagem) {
      int xAtual = display.getPersonagemX();
      int yAtual = display.getPersonagemY();
      bool espelhar = display.getPersonagemEspelhado();
      display.pularFigura(personagem, xAtual, yAtual, espelhar);
    }

    int getDestinoAtual() { return destinoAtual; }
    void setDestinoAtual(int novoDestino) { destinoAtual = novoDestino; }
};

// ---- SUBCLASSE TELAHOME ---- //
class TelaHome : public Tela {
  private:
    Figura figuraConfiguracoes;
    Figura figuraLuminosidade;
    Figura figuraUmidade;
    Figura figuraTemperatura;
    Figura figuraDataHora;

    // Pontos de parada da TelaHome
    const int destinos[7] = {1, 3, 5, 7, 9, 11, 14};
    const int numDestinos = 7;

  public:
    TelaHome(Figura configuracoes, Figura luminosidade, Figura umidade, Figura temperatura, Figura datahora)
      : figuraConfiguracoes(configuracoes), figuraLuminosidade(luminosidade), figuraUmidade(umidade), figuraTemperatura(temperatura), figuraDataHora(datahora) {}

    void imprimirTela(Display& display) override {
      display.limparColunaCima();
      display.imprimirFigura(figuraConfiguracoes, 11, 0, false);
      display.imprimirFigura(figuraLuminosidade, 3, 0, false);
      display.imprimirFigura(figuraUmidade, 5, 0, false);
      display.imprimirFigura(figuraTemperatura, 7, 0, false);
      display.imprimirFigura(figuraDataHora, 9, 0, false);
    }

    void moverDestino(Display& display, Figura& personagem, int destino) {
      Tela::moverDestino(display, personagem, destino, destinos, numDestinos);
    }
};

// VALOR LDR
void imprimirValorLDR() {
  lcd.setCursor(7, 0);
  lcd.print("    "); // Limpa a área antes de exibir o valor
  lcd.setCursor(7, 0);
  if (mostrarPercentual) {
    char buffer[5]; // Buffer para armazenar a string formatada
    snprintf(buffer, sizeof(buffer), "%3d%%", lightLevel); // Formata o valor para ocupar 3 caracteres + %
    lcd.print(buffer);
  } else {
    char buffer[5];
    snprintf(buffer, sizeof(buffer), "%4d", valorLDR); // Formata o valor para ocupar 4 caracteres
    lcd.print(buffer);
  }
}

// ---- SUBCLASSE TELALUMINOSIDADE ---- //
class TelaLuminosidade : public Tela {
  private:
    Figura figuraHome;
    Figura figuraLuminosidade;

    // Pontos de parada da TelaLuminosidade
    const int destinos[3] = {1, 3, 11};
    const int numDestinos = 3;

  public:

    TelaLuminosidade(Figura home, Figura luminosidade)
      : figuraHome(home), figuraLuminosidade(luminosidade) {}

    void imprimirTela(Display& display) override {
      display.limparColunaCima();
      display.imprimirFigura(figuraHome, 3, 0, false);
      display.imprimirFigura(figuraLuminosidade, 11, 0, false);

      // Exibe o valor do LDR
      imprimirValorLDR();
    }

    void moverDestino(Display& display, Figura& personagem, int destino) {
      Tela::moverDestino(display, personagem, destino, destinos, numDestinos);
    }
};

// ---- SUBCLASSE TELADATAHORA ---- //
class TelaDataHora : public Tela {
  private:
    Figura figuraHome;

    // Pontos de parada da TelaDataHora
    const int destinos[2] = {9, 0}; // Posições (9,1) e (0,1)
    const int numDestinos = 2;

  public:
    TelaDataHora(Figura home)
      : figuraHome(home) {}

    void imprimirTela(Display& display) override {
      display.limparColunaCima();

      // Exibe o ícone Home
      display.imprimirFigura(figuraHome, 0, 0, false);

      // Obtém a data e hora atual do RTC
      DateTime now = rtc.now();

      // Formata e exibe a hora (HH:MM)
      lcd.setCursor(2, 0);
      lcd.print(now.hour() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
      lcd.print(now.hour());
      lcd.print(":");
      lcd.print(now.minute() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
      lcd.print(now.minute());

      // Formata e exibe a data (DD/MM/YY)
      lcd.setCursor(8, 0);
      lcd.print(now.day() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
      lcd.print(now.day());
      lcd.print("/");
      lcd.print(now.month() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
      lcd.print(now.month());
      lcd.print("/");
      lcd.print(now.year() % 100); // Pega os últimos dois dígitos do ano
    }

    void moverDestino(Display& display, Figura& personagem, int destino) {
      Tela::moverDestino(display, personagem, destino, destinos, numDestinos);
    }
};

// ---- SUBCLASSE TELAUmidade ---- //
class TelaUmidade : public Tela {
  private:
    Figura figuraHome;
    Figura figuraUmidade;

    // Pontos de parada da TelaUmidade
    const int destinos[3] = {1, 3, 5}; // Posições (1,1), (3,1), (5,1)
    const int numDestinos = 3;

  public:
    TelaUmidade(Figura home, Figura umidade)
      : figuraHome(home), figuraUmidade(umidade) {}

    void imprimirTela(Display& display) override {
      display.limparColunaCima();

      // Exibe o ícone Home
      display.imprimirFigura(figuraHome, 3, 0, false);

      // Exibe o ícone Umidade
      display.imprimirFigura(figuraUmidade, 11, 0, false);

      // Exibe o valor da umidade
      lcd.setCursor(7, 0);
      lcd.print("   "); // Limpa a área antes de exibir o valor
      lcd.setCursor(7, 0);
      lcd.print(humid, 1); // Exibe a umidade com 1 casa decimal
      lcd.setCursor(10, 0);
      lcd.print("%");
    }

    void moverDestino(Display& display, Figura& personagem, int destino) {
      Tela::moverDestino(display, personagem, destino, destinos, numDestinos);
    }
};

// ---- SUBCLASSE TELATemperatura ---- //
class TelaTemperatura : public Tela {
  private:
    Figura figuraHome;
    Figura figuraTemperatura;
    int temperaturaScale = 1; // 1 = Celsius, 2 = Fahrenheit, 3 = Kelvin

    // Pontos de parada da TelaTemperatura
    const int destinos[4] = {1, 3, 7, 11}; // Posições (1,1), (3,1), (11,1)
    const int numDestinos = 4;

    // Método para converter temperatura
    float converterTemperatura(float tempCelsius, int escala) {
      switch (escala) {
        case 1: // Celsius
          return tempCelsius;
        case 2: // Fahrenheit
          return tempCelsius * 9 / 5 + 32;
        case 3: // Kelvin
          return tempCelsius + 273.15;
        default:
          return tempCelsius;
      }
    }

  public:
    TelaTemperatura(Figura home, Figura temperatura)
      : figuraHome(home), figuraTemperatura(temperatura) {}

    void imprimirTela(Display& display) override {
      display.limparColunaCima();

      // Exibe o ícone Home
      display.imprimirFigura(figuraHome, 3, 0, false);

      // Exibe o ícone Temperatura
      display.imprimirFigura(figuraTemperatura, 11, 0, false);

      // Converte a temperatura para a escala atual
      float tempConvertida = converterTemperatura(temp, temperaturaScale);

      // Exibe o valor da temperatura
      lcd.setCursor(6, 0);
      lcd.print("    "); // Limpa a área antes de exibir o valor
      lcd.setCursor(6, 0);
      lcd.print(tempConvertida, 1); // Exibe a temperatura com 1 casa decimal

      // Exibe a escala de temperatura
      lcd.setCursor(10, 0);
      switch (temperaturaScale) {
        case 1:
          lcd.print("C");
          break;
        case 2:
          lcd.print("F");
          break;
        case 3:
          lcd.print("K");
          break;
      }

      // Exibe o símbolo de grau (apenas para Celsius e Fahrenheit)
      if (temperaturaScale == 1 || temperaturaScale == 2) {
        lcd.setCursor(9, 0);
        lcd.print((char)223); // Símbolo de grau
      }
    }

    void moverDestino(Display& display, Figura& personagem, int destino) {
      Tela::moverDestino(display, personagem, destino, destinos, numDestinos);
    }

    void trocarEscala() {
      temperaturaScale = (temperaturaScale % 3) + 1; // Alterna entre 1, 2 e 3
    }

    int getEscala() { return temperaturaScale; }
};

// ---- MÉTODOS PARA RETORNAR OS BYTES DOS ÍCONES ---- //
byte* getHome() {
  static byte home[] = { B00000, B11111, B11011, B10001, B10101, B10001, B11111, B00000 };
  return home;
}

byte* getConfiguracoes() {
  static byte configuracoes[] = { B00000, B00000, B01010, B11111, B01010, B11111, B01010, B00000 };
  return configuracoes;
}

byte* getLuminosidade() {
  static byte luminosidade[] = { B01110, B10001, B10101, B10101, B01110, B00000, B01110, B01110 };
  return luminosidade;
}

byte* getUmidade() {
  static byte umidade[] = { B00100, B00100, B01110, B01110, B11111, B10111, B10011, B01110 };
  return umidade;
}

byte* getTemperatura() {
  static byte temperatura[] = { B01110, B01010, B01010, B01010, B11111, B11111, B11111, B01110 };
  return temperatura;
}

byte* getDataHora() {
  static byte datahora[] = { B00000, B00000, B01110, B10101, B10111, B10001, B01110, B00000 };
  return datahora;
}

// ---- CRIAÇÃO DE FIGURA ---- //
byte amongusSprite[8] = {
  B00000,
  B01111,
  B11000,
  B11111,
  B11111,
  B11111,
  B11111,
  B01001
};

// ---- FUNÇÃO PARA RETORNAR O BYTE DO AMONGUS ---- //
byte* getAmongUs() {
  static byte amongusSprite[] = {
    B00000,
    B01111,
    B11000,
    B11111,
    B11111,
    B11111,
    B11111,
    B01001
  };
  return amongusSprite;
}

// ---- FUNÇÃO PARA RETORNAR O BYTE DO MAGUINHO ---- //
byte* getMaguinho() {
  static byte maguinho[] = {
    B00000,
    B01000,
    B01000,
    B11101,
    B01001,
    B11111,
    B01001,
    B10100
  };
  return maguinho;
}

// ---- DEFINIÇÃO DOS NOVOS BYTES ---- //
byte abobora[] = { B00000, B00010, B00100, B01110, B10101, B11111, B10001, B01110 };
byte cefsa[] = { B00000, B00000, B00000, B00100, B01000, B01010, B10011, B01111 };
byte setinha[] = { B00000, B00100, B01110, B11111, B00100, B00100, B00100, B00100 };
byte balao[] = { B00000, B01110, B10001, B10011, B10001, B01110, B00100, B01000 };
byte bolinha[] = { B00000, B00000, B00000, B01110, B01110, B01110, B00000, B00000 };
byte cubo[] = { B00000, B00000, B00000, B11111, B10001, B10101, B10001, B11111 };
byte stickman[] = { B00000, B00100, B01010, B00100, B01110, B10101, B00100, B01010 };
byte fantasma[] = { B00000, B01110, B11111, B11101, B11101, B11111, B11111, B10101 };

// ---- LISTA DE BYTES DOS PERSONAGENS ---- //
byte* getPersonagem(int indice) {
  static byte* personagens[] = {
    getAmongUs(), // AmongUs
    getMaguinho(), // Maguinho
    abobora,      // Abóbora
    cefsa,        // Cefsa
    setinha,      // Setinha
    balao,        // Balão
    bolinha,      // Bolinha
    cubo,         // Cubo
    stickman,     // Stickman
    fantasma      // Fantasma
  };
  return personagens[indice];
}


// ---- VARIÁVEL PARA CONTROLAR O ÍNDICE DO PERSONAGEM ATUAL ---- //
int indicePersonagem = 0; // Começa com o AmongUs (índice 0)

// ---- VARIÁVEL PARA CONTROLAR O ESTADO DO PERSONAGEM ---- //
bool isAmongUs = true; // Começa com o AmongUs

// ---- FUNÇÃO PARA ALTERNAR ENTRE OS PERSONAGENS ---- //
void alternarPersonagem(Figura& personagem) {
  indicePersonagem = (indicePersonagem + 1) % 10; // Avança para o próximo personagem (10 personagens no total)
  personagem = Figura("AmongUs", getPersonagem(indicePersonagem), 7); // Atualiza o personagem
}


Figura amongus("AmongUs", amongusSprite, 7);
Display display;

// ---- BOTÕES ---- //
const int botaoVoltar = 7;
const int botaoSelecionar = 6;
const int botaoAvancar = 5;

// ---- INSTÂNCIAS DAS TELAS ---- //
Figura figHome("Home", getHome(), 5);
Figura figLuminosidadeTela("Luminosidade", getLuminosidade(), 6);
Figura figConfiguracoes("Configuracoes", getConfiguracoes(), 0);
Figura figLuminosidade("Luminosidade", getLuminosidade(), 1);
Figura figUmidade("Umidade", getUmidade(), 2);
Figura figTemperatura("Temperatura", getTemperatura(), 3);
Figura figDataHora("DataHora", getDataHora(), 4);

TelaHome telaHome(figConfiguracoes, figLuminosidade, figUmidade, figTemperatura, figDataHora);
TelaLuminosidade telaLuminosidade(figHome, figLuminosidadeTela);
TelaDataHora telaDataHora(figHome);
TelaUmidade telaUmidade(figHome, figUmidade);
TelaTemperatura telaTemperatura(figHome, figTemperatura);

// Função para carregar dados da EEPROM
void carregarDadosEEPROM() {
  EEPROM.get(addrLDR, valorLDR);
  EEPROM.get(addrTemp, temp);
  EEPROM.get(addrHumid, humid);
}

// Defina o pino do buzzer
#define BUZZER_PIN 8  // Altere para o pino correto do seu buzzer

// Primeira pose do mago
void wizard1() {
  byte name1x4[] = {
    B00000, B00000, B00000, B00000,
    B00000, B00000, B00000, B00000
  };
  byte name0x0[] = {
    B00000, B00000, B00000, B00000,
    B00000, B00000, B00000, B00001
  };
  byte name0x1[] = {
    B00000, B01000, B10100, B00100,
    B01110, B01110, B11111, B11111
  };
  byte name0x2[] = {
    B00000, B00000, B00000, B00000,
    B00000, B00000, B00000, B10000
  };
  byte name1x0[] = {
    B00000, B00000, B00001, B00010,
    B00010, B00010, B00010, B00010
  };
  byte name1x1[] = {
    B10001, B11011, B10001, B01010,
    B00100, B00000, B00100, B00100
  };
  byte name1x2[] = {
    B00010, B00101, B10010, B01010,
    B01010, B01110, B01010, B01000
  };

  lcd.createChar(0, name1x4);
  lcd.createChar(1, name0x0);
  lcd.createChar(2, name0x1);
  lcd.createChar(3, name0x2);
  lcd.createChar(4, name1x0);
  lcd.createChar(5, name1x1);
  lcd.createChar(6, name1x2);

  lcd.setCursor(4, 1);
  lcd.write(0);
  lcd.setCursor(0, 0);
  lcd.write(1);
  lcd.setCursor(1, 0);
  lcd.write(2);
  lcd.setCursor(2, 0);
  lcd.write(3);
  lcd.setCursor(0, 1);
  lcd.write(4);
  lcd.setCursor(1, 1);
  lcd.write(5);
  lcd.setCursor(2, 1);
  lcd.write(6);

  delay(400);
  lcd.clear();
}

// Segunda pose do mago (lança feitiço)
void wizard2() {
  byte name1x2[] = {
    B00000, B00000, B10000, B01000,
    B01001, B01110, B01010, B01000
  };
  byte name0x0[] = {
    B00000, B00000, B00000, B00000,
    B00000, B00000, B00000, B00001
  };
  byte name0x1[] = {
    B00000, B01000, B10100, B00100,
    B01110, B01110, B11111, B11111
  };
  byte name0x2[] = {
    B00000, B00000, B00000, B00000,
    B00000, B00000, B00000, B10000
  };
  byte name1x0[] = {
    B00000, B00000, B00001, B00010,
    B00010, B00010, B00010, B00010
  };
  byte name1x1[] = {
    B10001, B11011, B10001, B01010,
    B00100, B00000, B00100, B00100
  };
  byte name1x3[] = {
    B00100, B01010, B01100, B10000,
    B00000, B00000, B00000, B00000
  };

  lcd.createChar(0, name1x2);
  lcd.createChar(1, name0x0);
  lcd.createChar(2, name0x1);
  lcd.createChar(3, name0x2);
  lcd.createChar(4, name1x0);
  lcd.createChar(5, name1x1);
  lcd.createChar(6, name1x3);

  lcd.setCursor(2, 1);
  lcd.write(0);
  lcd.setCursor(0, 0);
  lcd.write(1);
  lcd.setCursor(1, 0);
  lcd.write(2);
  lcd.setCursor(2, 0);
  lcd.write(3);
  lcd.setCursor(0, 1);
  lcd.write(4);
  lcd.setCursor(1, 1);
  lcd.write(5);
  lcd.setCursor(3, 1);
  lcd.write(6);
}

// Animação "FORTUNATA!"
void magic() {
  // Palavra a ser exibida
  String word = "FORTUNATA!";
 
  // Define o "ball" que será utilizado na animação
  byte ball[] = {
    B00100, B01110, B00100, B00000,
    B00000, B00000, B00000, B00000
  };
  lcd.createChar(7, ball);

  // Definindo a melodia (primeira linha do tema de Hedwig)
  // Notas: Si - Mi - Sol - Fá# - Mi - Ré - Dó - Si
  // Frequências aproximadas (em Hz):
  // Si (B4) = 494, Mi (E5) = 659, Sol (G5) = 784, Fá# (F#5) = 740,
  // Ré (D5) = 587, Dó (C5) = 523.
  int melody[]    = {494, 659, 784, 740, 659, 587, 523, 494};
  // Durações (em ms) para cada nota, seguindo o ritmo "pam pam pamnana pam pam paaaam"
  int durations[] = {200, 200, 200, 400, 200, 200, 200, 600};
  int numNotes = sizeof(melody) / sizeof(melody[0]);
  int noteIndex = 0;

  // Configura a animação no LCD
  int startPos = 4;
  int endPos = 15;
  int frameDelay = 200;  // Tempo (ms) entre atualizações da animação
  int pos = startPos + 1;

  // Timers para animação e reprodução da melodia
  unsigned long prevFrameTime = millis();
  unsigned long noteStartTime = millis();

  // Exibe o "ball" na posição inicial
  lcd.setCursor(startPos, 1);
  lcd.write(byte(7));

  // Loop que intercalará animação e som
  while (pos <= endPos) {
    unsigned long currentTime = millis();

    // Atualiza a animação a cada frameDelay milissegundos
    if (currentTime - prevFrameTime >= frameDelay) {
      // Apaga o caractere anterior e desenha o ball na nova posição
      lcd.setCursor(pos - 1, 1);
      lcd.print(" ");
      lcd.setCursor(pos, 1);
      lcd.write(byte(7));

      // A partir da posição 6, revela as letras da palavra
      if (pos >= 6) {
        int letterIndex = pos - 6;
        if (letterIndex < (int)word.length()) {
          lcd.setCursor(pos - 1, 1);
          lcd.print(word[letterIndex]);
        }
      }
      pos++;
      prevFrameTime = currentTime;
    }

    // Enquanto a animação ocorre, toca as notas conforme seus tempos
    if (noteIndex < numNotes && (currentTime - noteStartTime >= durations[noteIndex])) {
      int noteDuration = durations[noteIndex];
      if (melody[noteIndex] == 0) {
        noTone(BUZZER_PIN);  // Pausa
      } else {
        tone(BUZZER_PIN, melody[noteIndex], noteDuration);
      }
      noteStartTime = currentTime;
      noteIndex++;
    }
  }

  // Finaliza a animação e desliga o buzzer
  delay(500);
  lcd.setCursor(endPos, 1);
  lcd.print(" ");
  delay(500);
  noTone(BUZZER_PIN);
  lcd.clear();
}

// Defina os pinos dos LEDs e do buzzer
#define LED_VERDE 10  // Pino do LED verde
#define LED_AMARELO 11 // Pino do LED amarelo
#define BUZZER_PIN 8   // Pino do buzzer

// Variáveis para controle do buzzer
bool buzzerOn = false; // Indica se o buzzer está ligado

void setup() {
  // Inicializa tudo
  Serial.begin(9600);
  dht.begin();
  lcd.init();
  lcd.backlight();
    // Configura os pinos dos LEDs e do buzzer
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AMARELO, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  // Executa a animação de inicialização
  wizard1();
  wizard2();
  byte vazio[8] = {B00000, B00000, B00000, B00000, B00000, B00000, B00000, B00000};
  for (int i = 0; i < 8; i++) {
    lcd.createChar(i, vazio);
  }
  magic();

  // Inicializa o RTC
  if (!rtc.begin()) { // SE O RTC NÃO FOR INICIALIZADO, FAZ
    Serial.println("DS3231 não encontrado"); // IMPRIME O TEXTO NO MONITOR SERIAL
    while (1); // SEMPRE ENTRE NO LOOP
  }

  if (rtc.lostPower()) { // SE RTC FOI LIGADO PELA PRIMEIRA VEZ / FICOU SEM ENERGIA / ESGOTOU A BATERIA, FAZ
    Serial.println("DS3231 OK! Ajustando data/hora..."); // IMPRIME O TEXTO NO MONITOR SERIAL
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); // Ajusta a data/hora com base no momento da compilação
  }

  // Carrega os dados da EEPROM
  carregarDadosEEPROM();

  // Botões
  pinMode(botaoVoltar, INPUT_PULLUP);
  pinMode(botaoSelecionar, INPUT_PULLUP);
  pinMode(botaoAvancar, INPUT_PULLUP);

  telaHome.imprimirTela(display);
  display.imprimirFigura(amongus, 1, 1, false);
}

void loop() {
  // Verifica o botão de voltar
  if (digitalRead(botaoVoltar) == LOW) {
    if (telaAtual == TELA_DATAHORA) {
      // Na TelaDataHora, o botão de voltar move o personagem de (9,1) para (2,1)
      int destinoAtual = telaDataHora.getDestinoAtual();
      if (destinoAtual == 0) { // Se estiver em (9,1), move para (2,1)
        telaDataHora.moverDestino(display, amongus, 1);
      }
    } else if (telaAtual == TELA_HOME) {
      int destinoAtual = telaHome.getDestinoAtual();
      if (destinoAtual > 0) {
        telaHome.moverDestino(display, amongus, destinoAtual - 1);
      }
    } else if (telaAtual == TELA_LUMINOSIDADE) {
      int destinoAtual = telaLuminosidade.getDestinoAtual();
      if (destinoAtual > 0) {
        telaLuminosidade.moverDestino(display, amongus, destinoAtual - 1);
      }
    } else if (telaAtual == TELA_UMIDADE) {
      int destinoAtual = telaUmidade.getDestinoAtual();
      if (destinoAtual > 0) {
        telaUmidade.moverDestino(display, amongus, destinoAtual - 1);
      }
    } else if (telaAtual == TELA_TEMPERATURA) {
      int destinoAtual = telaTemperatura.getDestinoAtual();
      if (destinoAtual > 0) {
        if (destinoAtual == 3) { // pular posição inicial
          telaTemperatura.moverDestino(display, amongus, destinoAtual - 2);
        } else {
          telaTemperatura.moverDestino(display, amongus, destinoAtual - 1);
        }
      }
    }
    delay(200); // Debounce
  }

  // Verifica o botão de avançar
  if (digitalRead(botaoAvancar) == LOW) {
    if (telaAtual == TELA_HOME) {
      int destinoAtual = telaHome.getDestinoAtual();
      if (destinoAtual < 6) {
        telaHome.moverDestino(display, amongus, destinoAtual + 1);
      }
    } else if (telaAtual == TELA_LUMINOSIDADE) {
      int destinoAtual = telaLuminosidade.getDestinoAtual();
      if (destinoAtual < 2) {
        telaLuminosidade.moverDestino(display, amongus, destinoAtual + 1);
      }
    } else if (telaAtual == TELA_UMIDADE) {
      int destinoAtual = telaUmidade.getDestinoAtual();
      if (destinoAtual < 1) {
        telaUmidade.moverDestino(display, amongus, destinoAtual + 1);
      }
    } else if (telaAtual == TELA_TEMPERATURA) {
      int destinoAtual = telaTemperatura.getDestinoAtual();
      if (destinoAtual < 3) {
        if (destinoAtual == 1) { // pular posicao inicial
          telaTemperatura.moverDestino(display, amongus, destinoAtual + 2);
        } else {
          telaTemperatura.moverDestino(display, amongus, destinoAtual + 1);
        }
      }
    }
    // Na TelaDataHora, o botão de avançar não faz nada
    delay(200); // Debounce
  }

  // Verifica o botão de selecionar
  if (digitalRead(botaoSelecionar) == LOW) {
    int xAtual = display.getPersonagemX();
    int yAtual = display.getPersonagemY();
   
    if (telaAtual == TELA_HOME) {
      telaHome.selecionar(display, amongus);
       if (xAtual == 11 && yAtual == 1) {
         // Alterna entre os personagens
        alternarPersonagem(amongus);
        // Atualiza a exibição do personagem na tela
        display.imprimirFigura(amongus, xAtual, yAtual, display.getPersonagemEspelhado());
      } else if (xAtual == 5 && yAtual == 1) {
        // Muda para TelaUmidade
        telaAtual = TELA_UMIDADE;
        telaUmidade.imprimirTela(display);
        telaUmidade.setDestinoAtual(2);
      } else if (xAtual == 7 && yAtual == 1) {
        // Muda para TelaTemperatura
        telaAtual = TELA_TEMPERATURA;
        telaTemperatura.imprimirTela(display);
        telaTemperatura.setDestinoAtual(2);
      }
      if (xAtual == 9 && yAtual == 1) {
        // Muda para TelaDataHora
        telaAtual = TELA_DATAHORA;
        telaDataHora.imprimirTela(display);

        // Reinicia o destinoAtual para 0 (posição inicial)
        telaDataHora.moverDestino(display, amongus, 0);
      } else if (xAtual == 3 && yAtual == 1) {
        // Muda para TelaLuminosidade
        telaAtual = TELA_LUMINOSIDADE;
        telaLuminosidade.imprimirTela(display);
        telaLuminosidade.setDestinoAtual(1);
      }
    } else if (telaAtual == TELA_DATAHORA) {
      telaDataHora.selecionar(display, amongus);
      if (xAtual == 0 && yAtual == 1) {
        // Volta para TelaHome
       
        telaAtual = TELA_HOME;
        telaHome.imprimirTela(display);
        telaHome.setDestinoAtual(1);
        // Limpa a posição (9,1) antes de mover o personagem para (2,1)
        display.limparCelula(9, 1);
        // Move o personagem para (0,1) e mantém o espelhamento
        bool espelhar = display.getPersonagemEspelhado();
        display.imprimirFigura(amongus, 0, 1, espelhar);
        // Reinicia o destinoAtual para 0 (posição inicial)
        telaHome.moverDestino(display, amongus, 0);
      }
    } else if (telaAtual == TELA_LUMINOSIDADE) {
      telaLuminosidade.selecionar(display, amongus);
      if (xAtual == 3 && yAtual == 1) {
        // Volta para TelaHome
        telaAtual = TELA_HOME;
        telaHome.imprimirTela(display);
      } else if (xAtual == 11 && yAtual == 1) {
        // Alterna entre percentual e valor bruto
        mostrarPercentual = !mostrarPercentual;
        telaLuminosidade.imprimirTela(display); // Atualiza a tela
      }
    } else if (telaAtual == TELA_TEMPERATURA) {
      telaTemperatura.selecionar(display, amongus);
      if (xAtual == 11 && yAtual == 1) {
      telaTemperatura.trocarEscala();
      telaTemperatura.imprimirTela(display); // Atualiza a tela com a nova escala
      } else if (xAtual == 3 && yAtual == 1) {
          // Volta para TelaHome
          telaAtual = TELA_HOME;
          telaHome.imprimirTela(display);
          telaHome.setDestinoAtual(1);
      }
    } else if (telaAtual == TELA_UMIDADE) {
      telaUmidade.selecionar(display, amongus);
      if (xAtual == 3 && yAtual == 1) {
        telaAtual = TELA_HOME;
        telaHome.imprimirTela(display);
        telaHome.setDestinoAtual(1);
      }
    } else {
      // Executa o pulo normal
      display.pularFigura(amongus, xAtual, yAtual, display.getPersonagemEspelhado());
    }
    delay(200); // Debounce
  }

  // Leitura do LDR e atualização da data/hora e DHT
  unsigned long tempoAtual = millis();

  // Verifica se passaram 5 segundos desde a última leitura
  if (tempoAtual - ultimaLeitura >= intervaloLeitura || tempoAtual == 0) {
    ultimaLeitura = tempoAtual;

    // Leitura do DHT22
    float tempRaw = dht.readTemperature(); // Celsius
    float humRaw = dht.readHumidity();

    if (!isnan(tempRaw)) {
      temp = tempRaw; // Armazena a temperatura lida
    }

    if (!isnan(humRaw)) {
      humid = humRaw; // Armazena a umidade lida
    }

    // Faz a leitura do LDR
    valorLDR = analogRead(LDR_PIN);
    lightLevel = map(valorLDR, 300, 40, 100, 0);
    lightLevel = constrain(lightLevel, 0, 100);

    // Salva os dados na EEPROM
    salvarDadosEEPROM();

    // Atualiza a tela de luminosidade se estiver ativa
    if (telaAtual == TELA_LUMINOSIDADE) {
      telaLuminosidade.imprimirTela(display);
    }

    // Atualiza a tela de data/hora se estiver ativa
    if (telaAtual == TELA_DATAHORA) {
      telaDataHora.imprimirTela(display);
    }

    // Verifica as condições para ligar o LED amarelo e o buzzer
    if ((lightLevel < 10) || (lightLevel > 40)) {
      digitalWrite(LED_AMARELO, HIGH); // Liga o LED amarelo
      if (!buzzerOn) {
        tone(BUZZER_PIN, 1000); // Liga o buzzer
        buzzerOn = true;
      }
    } else if ((temp < 20) || (temp > 25)) {
      digitalWrite(LED_AMARELO, HIGH); // Liga o LED amarelo
      if (!buzzerOn) {
        tone(BUZZER_PIN, 1000); // Liga o buzzer
        buzzerOn = true;
      }
    } else if (humid < 30 || humid > 60) {
      digitalWrite(LED_AMARELO, HIGH); // Liga o LED amarelo
      if (!buzzerOn) {
        tone(BUZZER_PIN, 1000); // Liga o buzzer
        buzzerOn = true;
      }
    } else {
      digitalWrite(LED_AMARELO, LOW); // Desliga o LED amarelo
      if (buzzerOn) {
        noTone(BUZZER_PIN); // Desliga o buzzer
        buzzerOn = false;
      }
    }
  }
}

// Função para salvar dados na EEPROM e imprimir no serial
void salvarDadosEEPROM() {
  EEPROM.put(addrLDR, valorLDR);
  EEPROM.put(addrTemp, temp);
  EEPROM.put(addrHumid, humid);

  // Imprime os valores salvos na EEPROM no monitor serial
  Serial.println("Dados salvos na EEPROM:");
  Serial.print("Valor LDR: ");
  Serial.println(valorLDR);
  Serial.print("Temperatura: ");
  Serial.println(temp);
  Serial.print("Umidade: ");
  Serial.println(humid);
}

```

### 📦 Estrutura Modular

- Classes:
  - `Figura`: Define o sprite do personagem/ícone.
  - `Display`: Gerencia o desenho no LCD (animações, movimento).
  - `Tela`: Classe base para telas (menu, sensores etc).
  - Subclasses específicas para cada tela:
    - `TelaHome`
    - `TelaLuminosidade`
    - `TelaTemperatura`
    - `TelaUmidade`
    - `TelaDataHora`

### 🧠 Funções-Chave

```cpp
void alternarPersonagem(Figura& personagem);
// Alterna para o próximo sprite da lista de personagens.

void imprimirValorLDR();
// Exibe o valor do LDR, em percentual ou valor bruto.

void salvarDadosEEPROM();
// Salva os dados dos sensores na EEPROM.

void carregarDadosEEPROM();
// Recupera os dados ao iniciar o sistema.
```

### 🔄 Loop principal
Monitora botões e movimenta personagem entre ícones.
Executa ações como alternar personagens, mudar telas, trocar escala de temperatura etc.
Atualiza leituras dos sensores e salva EEPROM a cada 5s.
Gera alertas visuais e sonoros quando os valores saem da faixa ideal.

### 🧪 Como Usar
Monte os componentes conforme os pinos especificados.
Carregue o código para o Arduino.
Abra o Monitor Serial (opcional) para ver os dados salvos.
Utilize os botões para explorar os menus e brincar com os personagens!

### 🎉 Destaque: Interface Amigável e Lúdica
Este projeto vai além de uma simples estação de monitoramento. Ele proporciona uma experiência lúdica e divertida, ideal para:

Apresentações escolares e universitárias
Ambientes educativos
Interações com crianças
Demonstração de conceitos de IoT com interface amigável

### ▶️ Video no Youtube

https://www.youtube.com/watch?v=VkIX3WWGT2w


## ✨ Fim do README.md

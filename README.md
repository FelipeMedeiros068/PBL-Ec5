
# 📌 Sistema Integrado de Monitoramento Ambiental com Arduino

Este projeto é um sistema de monitoramento automático de temperatura, umidade e luminosidade, utilizando sensores, comunicação I²C, alertas visuais e sonoros, armazenamento de registros em memória EEPROM e exibição em display LCD 16x2.

---

## 📝 Especificações Técnicas

### 🔧 Hardware Utilizado

- **Microcontrolador:** Arduino (Uno, Nano ou compatível)
- **Display:** LCD 16x2 com módulo I²C (Endereço: `0x27`)
- **Sensor de temperatura e umidade:** DHT22
- **Sensor de luminosidade:** Fotoresistor (LDR)
- **Módulo RTC:** DS3231 (Real-Time Clock)
- **EEPROM interna:** Armazenamento de até 100 registros de anomalias
- **LEDs indicadores:** Vermelho, Amarelo, Verde
- **Buzzer:** Alertas sonoros ativos
- **Botões:** UP, DOWN, SELECT, BACK para navegação de menus

---

### 🔌 Conexões e Pinagem

| Componente     | Pino Arduino              |
|----------------|---------------------------|
| LCD I²C        | SDA (A4), SCL (A5)        |
| Sensor DHT22   | Digital 9                 |
| LDR            | Analógico A0              |
| RTC DS3231     | SDA (A4), SCL (A5)        |
| Botão UP       | Digital 3                 |
| Botão DOWN     | Digital 4                 |
| Botão SELECT   | Digital 5                 |
| Botão BACK     | Digital 2                 |
| LED Verde      | Digital 6                 |
| LED Amarelo    | Digital 7                 |
| LED Vermelho   | Digital 8                 |
| Buzzer         | Digital 13                |

---

### Diagrama Elétrico

![DIAGRAMA ELETRICO DATALOGGER](https://github.com/user-attachments/assets/d3a90ce5-811a-49dc-8751-14028dc1fd14)


---

## 📖 Manual de Operação

### ✅ Inicialização

- Ao ligar o dispositivo, o sistema exibe uma animação inicial no LCD.
- Realiza a verificação dos sensores e módulo RTC.
- Após o término da animação, o usuário é direcionado ao menu principal.

### 📌 Navegação dos Menus

Utilize os botões para interagir com o sistema:

- `UP` e `DOWN`: Navegam entre as opções.
- `SELECT`: Seleciona uma opção do menu.
- `BACK`: Retorna ao menu anterior ou principal.

---

### 🔖 Opções Disponíveis no Menu

- **ESCALA TEMP**: Seleciona a unidade de medida (Celsius, Fahrenheit ou Kelvin).
- **HOME**: Mostra leituras atuais de temperatura, umidade e luminosidade.
- **RTC**: Mostra data e hora atuais do módulo RTC.

---

## 🚨 Alertas e Limites Pré-definidos

Os alertas são acionados automaticamente com base nos seguintes limites definidos:

| Parâmetro          | Limite Mínimo 🚩 | Limite Máximo 🚩 | Indicador      | Unidade de Medida | Precisão       |
|--------------------|------------------|------------------|----------------|--------------------|----------------|
| 🌡️ **Temperatura**  | 15.0 °C          | 25.0 °C          | 🔴 LED Vermelho| Graus Celsius (°C) | ±2.0°C         |
| 💧 **Umidade**       | 30.0 %           | 50.0 %           | 🟢 LED Verde   | Umidade Relativa (%RH) | ±5% RH   |
| 💡 **Luminosidade**  | 0.0 %            | 30.0 %           | 🟡 LED Amarelo | Intensidade Luminosa (%) | Depende do LDR |

- Ao **exceder algum limite acima**, o respectivo LED será acionado **juntamente com o buzzer**.
- Para **silenciar alertas**, pressione o botão `BACK` para retornar ao **menu principal**.
- O **buzzer será ativado** se pelo menos **um dos parâmetros estiver fora dos limites definidos**.

---

## 💾 Registro Automático de Anomalias (EEPROM)

- Registros automáticos de condições fora dos limites são salvos na EEPROM.
- Capacidade máxima de armazenamento: **100 registros**.
- Cada registro inclui: data/hora, temperatura e umidade.

---

## 💻 Código Fonte Comentado

### 📍 Bibliotecas e Definições Principais

```cpp
#include <LiquidCrystal_I2C.h> // Comunicação LCD via I²C
#include <Wire.h>              // Comunicação I²C
#include <DHT.h>               // Sensor DHT22
#include <RTClib.h>            // RTC DS3231
#include <EEPROM.h>            // Armazenamento EEPROM

RTC_DS3231 rtc; // Objeto para manipulação do RTC
```

### 📍 Código Comentado

---
```cpp/************************************************************
 *                   INCLUDES & DEFINES                     *
 ************************************************************/
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#include <DHT.h>
#include <RTClib.h>
#include <EEPROM.h>

RTC_DS3231 rtc; //OBJETO DO TIPO RTC_DS3231
 
// COnfigurações dos dias da semana
char daysOfTheWeek[7][12] = {"Domingo", "Segunda", "Terça", "Quarta", "Quinta", "Sexta", "Sábado"};
#define UTC_OFFSET -3    // Ajuste de fuso horário para UTC-3
#define LOG_OPTION 1     // Opção para ativar a leitura do log

// Configurações da EEPROM
const int maxRecords     = 100;
const int recordSize     = 8; // Tamanho de cada registro em bytes
int       startAddress   = 0;
int       endAddress     = maxRecords * recordSize;
int       currentAddress = 0;

int lastLoggedMinute = -1;

// Triggers de temperatura e umidade
float trigger_t_min = 15.0; // Exemplo: valor mínimo de temperatura
float trigger_t_max = 25.0; // Exemplo: valor máximo de temperatura
float trigger_u_min = 30.0; // Exemplo: valor mínimo de umidade
float trigger_u_max = 50.0; // Exemplo: valor máximo de umidade
float trigger_l_min = 0.0; // Exemplo: valor mínimo de luminosidade
float trigger_l_max = 30.0; // Exemplo: valor máximo de lumisodidade

// Endereço e dimensões do LCD I2C
#define I2C_ADDR     0x27
#define LCD_COLUMNS  16
#define LCD_LINES    2

// DHT Sensor
#define DHTPIN       9       // Pino do DHT
#define DHTTYPE      DHT11   // Tipo de sensor DHT

// Botões
#define UP_BUTTON     3
#define DOWN_BUTTON   4
#define SELECT_BUTTON 5
#define BACK_BUTTON   2

// LEDs
#define LED_RED  8
#define LED_YEL  7
#define LED_GRE  6

// Buzzer
#define BUZZER_PIN     13
bool buzzerOn          = false; // Indica se o buzzer está ligado
bool buzzerTempReason  = false; // Indica se o buzzer foi ligado especificamente por causa da temperatura
bool buzzerHumdReason  = false; // Indica se o buzzer foi ligado especificamente por causa da umidade
bool buzzerLightReason = false; // Indica se o buzzer foi ligado especificamente por causa da luminosidade


// LDR
#define LDR_PIN A0

/************************************************************
 *               OBJETOS & VARIÁVEIS GLOBAIS                *
 ************************************************************/
// LiquidCrystal_I2C lcd(Endereço, colunas, linhas)
LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);

// DHT
DHT dht(DHTPIN, DHTTYPE);

// Menu
int   menu              = 1;
bool  rtcMenuActive     = false;
bool  subMenuTempActive = false; 
bool  homePageActive    = false;
int   subMenuIndex      = 1;    // Índice no submenu de temperatura

// Escala de temperatura (1=Celsius, 2=Fahrenheit, 3=Kelvin)
int  temperatureScale = 1;
float lastAvgTemp; // Armazena a última média de temperatura
float lastAvgHumd; // Armazena a última média de umidade

// Leituras de sensores
float temp         = 0.0;
float humid        = 0.0;
int   lightLevel   = 0;

// Arrays de leituras (se quiser usar média futuramente)
float tempReadings[10]; 
float humdReadings[10];
int   currentIndex = 0;

unsigned long lastHomeUpdate = 0;
long totalLeituras = 0;

DateTime adjustedTime;
DateTime now;


/************************************************************
 *                 FUNÇÃO PARA DESLIGAR ALERTAS             *
 ************************************************************/
void turnOffAllAlerts() {
  // Desliga LEDs
  digitalWrite(LED_RED, LOW);
  digitalWrite(LED_GRE, LOW);
  digitalWrite(LED_YEL, LOW);

  // Desliga Buzzer
  noTone(BUZZER_PIN);

  // Zera flags
  buzzerOn          = false;
  buzzerTempReason  = false;
  buzzerHumdReason  = false;
  buzzerLightReason = false;
}


/************************************************************
 *                          SETUP                           *
 ************************************************************/
void setup() {
  Serial.begin(9600); // Inicializa a comunicação serial
  rtc.begin();    // Inicialização do Relógio em Tempo Real
  rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  EEPROM.begin();

  if(! rtc.begin()) { // SE O RTC NÃO FOR INICIALIZADO, FAZ
    Serial.println("DS3231 não encontrado"); //IMPRIME O TEXTO NO MONITOR SERIAL
    while(1); //SEMPRE ENTRE NO LOOP
  }
  if(rtc.lostPower()){ //SE RTC FOI LIGADO PELA PRIMEIRA VEZ / FICOU SEM ENERGIA / ESGOTOU A BATERIA, FAZ
    Serial.println("DS3231 OK!"); //IMPRIME O TEXTO NO MONITOR SERIAL
    rtc.adjust(DateTime(2025, 3, 20, 19, 30, 45)); //(ANO), (MÊS), (DIA), (HORA), (MINUTOS), (SEGUNDOS)
  }
  delay(100); //INTERVALO DE 100 MILISSEGUNDOS
  
  // Inicialização dos pinos de botões
  pinMode(UP_BUTTON,     INPUT_PULLUP);
  pinMode(DOWN_BUTTON,   INPUT_PULLUP);
  pinMode(SELECT_BUTTON, INPUT_PULLUP);
  pinMode(BACK_BUTTON,   INPUT_PULLUP);

  // Buzzer e LEDs
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_RED,    OUTPUT);
  pinMode(LED_YEL,    OUTPUT);
  pinMode(LED_GRE,    OUTPUT);

  // LDR
  pinMode(LDR_PIN, INPUT);

  // Inicializa o DHT
  dht.begin();

  // LCD
  lcd.init();
  lcd.backlight();

  // Animações de introdução
  lcd.clear();
  lcd.setCursor(0, 0);
  wizard1();
  wizard2();
  magic();
  lcd.clear();
  welcome();
  delay(1000);
  
  lcd.clear();
  lcd.setCursor(4, 0);
  lcd.print("Loading");
  delay(1500);

  // Exibe o menu principal ao iniciar
  exibir_menu();
}


/************************************************************
 *                          LOOP                            *
 ************************************************************/
void loop() {

  // Controla a frequência da impressão serial
  static unsigned long lastSerialTime = 0;

  // ==== LEITURA DE SENSORES ====
  int valorLDR   = analogRead(LDR_PIN);
  lightLevel     = map(valorLDR, 40, 950, 0, 100);

  float tempRaw  = dht.readTemperature(); // Celsius
  float humRaw   = dht.readHumidity();

  if (!isnan(tempRaw)) {
    // Converte a temperatura conforme a escala
    switch (temperatureScale) {
      case 1: // Celsius
        temp = tempRaw;
        break;
      case 2: // Fahrenheit
        temp = (tempRaw * 1.8) + 32;
        break;
      case 3: // Kelvin
        temp = tempRaw + 273.15;
        break;
    }
  }

  if (!isnan(humRaw)) {
    humid = humRaw; // umidade permanece em %
  }

  tempReadings[currentIndex] = temp;  // Armazena a leitura atual da temperatura
  humdReadings[currentIndex] = humid;  // Armazena a leitura atual da umidade

  currentIndex = (currentIndex + 1) % 10; // Atualiza o índice para a próxima leitura

  if (currentIndex == 9) {
    tenthRead();
  }

  // Exibição serial periódica (a cada 1s, por exemplo)
  if (millis() - lastSerialTime >= 1000UL) {
    lastSerialTime = millis();
    // Incrementa a contagem total de leituras
    totalLeituras++;
    serialLog(temp, humid, valorLDR, totalLeituras);
  }

  recordEEPROM();

  if (homePageActive) {
    // Verifica luminosidade só enquanto em HOME
    checkLightAlert();

    // A cada 1s, atualiza a tela
    if (millis() - lastHomeUpdate >= 1000UL) {
      lastHomeUpdate = millis();
      showHomeValues();
    }
  }
  else {
    // (NOVO) Se não está na HOME, desliga todos os alertas
    turnOffAllAlerts();
  }

  // ==== SUBMENU DE TEMPERATURA ====
  if (subMenuTempActive) {
    if (!digitalRead(BACK_BUTTON)) {
      subMenuTempActive = false;
      exibir_menu();
      delay(100);
      while (!digitalRead(BACK_BUTTON));
    }

    if (!digitalRead(DOWN_BUTTON)) {
      subMenuIndex++;
      if (subMenuIndex > 3) subMenuIndex = 3;
      exibir_submenu_temp();
      delay(100);
      while (!digitalRead(DOWN_BUTTON));
    }

    if (!digitalRead(UP_BUTTON)) {
      subMenuIndex--;
      if (subMenuIndex < 1) subMenuIndex = 1;
      exibir_submenu_temp();
      delay(100);
      while (!digitalRead(UP_BUTTON));
    }

    if (!digitalRead(SELECT_BUTTON)) {
      executeActionTemp();   // Define a escala de temperatura
      subMenuTempActive = false;
      exibir_menu();
      delay(100);
      while (!digitalRead(SELECT_BUTTON));
    }

    // Se esta no submenu, encerra o loop
    return;
  }

  // ==== TELA HOME ====
  if (homePageActive) {
    if (!digitalRead(BACK_BUTTON)) {
      homePageActive = false;
      exibir_menu();
      delay(100);
      while (!digitalRead(BACK_BUTTON));
    }

    // Se esta na home, encerra o loop
    return;
  }

  // ==== MENU PRINCIPAL ====
  if (!digitalRead(DOWN_BUTTON)) {
    menu++;
    exibir_menu();
    delay(100);
    while (!digitalRead(DOWN_BUTTON));
  }
  if (!digitalRead(UP_BUTTON)) {
    menu--;
    exibir_menu();
    delay(100);
    while (!digitalRead(UP_BUTTON));
  }
  if (!digitalRead(SELECT_BUTTON)) {
    executeAction();
    // Se a ação não ativou o submenu nem home, reexibe o menu
    if (!subMenuTempActive && !homePageActive) {
      exibir_menu();
    }
    delay(100);
    while (!digitalRead(SELECT_BUTTON));
  }

  if (rtcMenuActive) {
        displayRTC();
        if (!digitalRead(BACK_BUTTON)) {
            rtcMenuActive = false;
            exibir_menu();
            delay(100);
            while (!digitalRead(BACK_BUTTON));
        }
        return;
    }

    if (!digitalRead(DOWN_BUTTON)) {
        menu++;
        exibir_menu();
        delay(100);
        while (!digitalRead(DOWN_BUTTON));
    }
    if (!digitalRead(UP_BUTTON)) {
        menu--;
        exibir_menu();
        delay(100);
        while (!digitalRead(UP_BUTTON));
    }
    if (!digitalRead(SELECT_BUTTON)) {
        executeAction();
        delay(100);
        while (!digitalRead(SELECT_BUTTON));
    }
}


/************************************************************
 *                       FUNÇÕES MENU                       *
 ************************************************************/
// Exibe o menu principal
void exibir_menu() {
  switch (menu) {
    case 0:
      menu = 1;
      break;
    case 1:
      lcd.clear();
      lcd.print(">ESCALA TEMP.");
      lcd.setCursor(0, 1);
      lcd.print(" HOME");
      break;
    case 2:
      lcd.clear();
      lcd.print(" ESCALA TEMP.");
      lcd.setCursor(0, 1);
      lcd.print(">HOME");
      break;
    case 3:
      lcd.clear();
      lcd.print(" HOME");
      lcd.setCursor(0, 1);
      lcd.print(">RTC");
      break;
    case 4:
      menu = 3;
      break;
  }
}

// Exibe o submenu de temperatura
void exibir_submenu_temp() {
  lcd.clear();
  switch (subMenuIndex) {
    case 0:
      subMenuIndex = 1;
      // Força o usuário a manter o range do submenu
    case 1:
      lcd.clear();
      lcd.print(">CELSIUS");
      lcd.setCursor(0, 1);
      lcd.print(" FAHRENHEIT");
      break;

    case 2:
      lcd.clear();
      lcd.print(" CELSIUS");
      lcd.setCursor(0, 1);
      lcd.print(">FAHRENHEIT");
      break;

    case 3:
      lcd.clear();
      lcd.print(" FAHRENHEIT");
      lcd.setCursor(0, 1);
      lcd.print(">KELVIN");
      break;

    case 4:
      // Força o usuário a manter o range do submenu
      subMenuIndex = 3;
      break;
  }
}

// Define a escala de temperatura e exibe a confirmação
void executeActionTemp() {
  switch (subMenuIndex) {
    case 1: temperatureScale = 1; break; // Celsius
    case 2: temperatureScale = 2; break; // Fahrenheit
    case 3: temperatureScale = 3; break; // Kelvin
  }
  lcd.clear();
  lcd.print("Escala defin.");
  delay(1000);
}

// Executa ação do menu principal
void executeAction() {
  switch (menu) {
    case 1:
      subMenuTempActive = true;
      subMenuIndex = 1;
      exibir_submenu_temp();
      break;
    case 2:
      showHomePage();
      break;
    case 3:
      rtcMenuActive = true; // Ativa o menu do RTC
      lcd.clear();
      while (rtcMenuActive) {
        displayRTC(); // Atualiza a tela com data e hora
        delay(1000);

        // Verifica se o botão BACK foi pressionado para voltar ao menu principal
        if (!digitalRead(BACK_BUTTON)) {
          rtcMenuActive = false;
          exibir_menu();
          delay(100);
          while (!digitalRead(BACK_BUTTON)); // Aguarda o botão ser solto
        }
      }
      break;
  }
}

void showHomeValues() {

  // --- CRIA SUFIXO DE TEMPERATURA ---
  char tempSuffix = 'C'; // Por padrão, Celsius
  if (temperatureScale == 2)      tempSuffix = 'F';
  else if (temperatureScale == 3) tempSuffix = 'K';

  String tempStr = String(lastAvgTemp, 0) + tempSuffix;
  String lightStr = String(lightLevel) + "%";
  String humStr = String(lastAvgHumd, 0) + "%";

  // Limpa toda a linha 1 antes de imprimir novos valores
  lcd.setCursor(0, 1);
  lcd.print("                ");

  //  Exibe temperatura no (0,1), luminosidade no (6,1), umidade no (12,1)
  lcd.setCursor(0, 1);
  lcd.print(tempStr);

  lcd.setCursor(6, 1);
  lcd.print(lightStr);

  lcd.setCursor(12, 1);
  lcd.print(humStr);
}

// Mostra a tela home
void showHomePage() {
  lcd.clear();
  homePageActive = true;
  homePage();

  // Registra as entradas na página home
  lastHomeUpdate = millis();
}


/************************************************************
 *                       FUNÇÕES HOME                       *
 ************************************************************/
// Desenha a Home Page
void homePage() {
  byte name0x1[]  = { B01110, B01010, B01010, B01010, B11111, B11111, B11111, B01110 };
  byte name0x7[]  = { B00001, B00010, B00100, B01000, B11111, B00010, B00100, B01000 };
  byte name0x13[] = { B00100, B00100, B01110, B01110, B11111, B11111, B11111, B01110 };

  lcd.createChar(0, name0x1);
  lcd.createChar(1, name0x7);
  lcd.createChar(2, name0x13);

  lcd.setCursor(1, 0); 
  lcd.write((uint8_t)0);

  lcd.setCursor(7, 0);
  lcd.write((uint8_t)1);

  lcd.setCursor(13, 0);
  lcd.write((uint8_t)2);

  showHomeValues();
}


/************************************************************
 *                FUNÇÕES DE ANIMAÇÃO/TELAS                 *
 ************************************************************/
// Exibe slogan animado no LCD
void welcome() {
  String line = "VEJA O OCULTO";
  for (int i = 0; i < (int)line.length(); i++) {
    lcd.setCursor(i + 1, 0);
    lcd.print(line[i]);
    delay(150);

    // Efeito de letras 'caindo'
    lcd.setCursor(i + 1, 0);
    lcd.print(" ");
    lcd.setCursor(i + 1, 1);
    lcd.print(line[i]);

    // Buzzer
    if (!isWhitespace(line[i])) {
      tone(BUZZER_PIN, 250);
      delay(150);
      noTone(BUZZER_PIN);
    }
  }
}

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
}
void displayRTC() {
    DateTime nowRTC = rtc.now();
    
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("DATA: ");
    lcd.print(nowRTC.day() < 10 ? "0" : "");
    lcd.print(nowRTC.day());
    lcd.print("/");
    lcd.print(nowRTC.month() < 10 ? "0" : ""); 
    lcd.print(nowRTC.month());
    lcd.print("/");
    lcd.print(nowRTC.year());

    lcd.setCursor(0, 1);
    lcd.print("HORA: ");
    lcd.print(nowRTC.hour() < 10 ? "0" : ""); 
    lcd.print(nowRTC.hour());
    lcd.print(":");
    lcd.print(nowRTC.minute() < 10 ? "0" : ""); 
    lcd.print(nowRTC.minute());
    lcd.print(":");
    lcd.print(nowRTC.second() < 10 ? "0" : ""); 
    lcd.print(nowRTC.second());
}




/************************************************************
 *                FUNÇÕES DE LEITURA/SENSORES               *
 ************************************************************/
// Calcula a média das últimas 10 leituras e exibe as telas de cada campo
void tenthRead() {
  float sumTemp = 0;
  float sumHumd = 0;

  for (int i = 0; i < 10; i++) {
    sumTemp += tempReadings[i];
    sumHumd += humdReadings[i];
  }

  lastAvgTemp = sumTemp / 10;
  lastAvgHumd = sumHumd / 10;

  if (homePageActive) {
    checkTempAlert();
    checkHumdAlert();
  }
}

void checkTempAlert() {
  float minTempThreshold = 15.0;
  float maxTempThreshold = 25.0;

  // Ajusta os limites de temperatura de acordo com a escala selecionada
  switch (temperatureScale) {
    case 2: // Fahrenheit
      minTempThreshold = (15.0 * 1.8) + 32;  // Convertendo para Fahrenheit
      maxTempThreshold = (25.0 * 1.8) + 32;
      break;
    case 3: // Kelvin
      minTempThreshold = 15.0 + 273.15;  // Convertendo para Kelvin
      maxTempThreshold = 25.0 + 273.15;
      break;
  }

  if ((lastAvgTemp < minTempThreshold) || (lastAvgTemp > maxTempThreshold)) {
    // Faixa perigosa
    digitalWrite(LED_RED, HIGH);

    // Se o buzzer não estiver ligado, ligamos agora
    if (!buzzerOn) {
      tone(BUZZER_PIN, 1000);  // Exemplo de frequência
      buzzerOn         = true;
      buzzerTempReason = true;
    }
  }
  else {
    // Faixa segura
    digitalWrite(LED_RED, LOW);

    if (buzzerTempReason) {
      buzzerTempReason = false;
      // Se não há outro motivo, desliga
      if (!buzzerHumdReason && !buzzerLightReason) {
        noTone(BUZZER_PIN);
        buzzerOn = false;
      }
    }
  }
}


void checkHumdAlert() {
  // Fora da faixa => acende ledGre, liga buzzer se não estiver ligado
  if ((lastAvgHumd < 40.0) || (lastAvgHumd > 65.0)) {
    digitalWrite(LED_GRE, HIGH);

    // Se o buzzer ainda não estiver ligado, ligue-o
    if (!buzzerOn) {
      tone(BUZZER_PIN, 1000); 
      buzzerOn = true;
      buzzerHumdReason = true;
    }
  }
  else {
    // Dentro da faixa => desliga ledGre
    digitalWrite(LED_GRE, LOW);

    // Se o buzzer estava ligado especificamente pela umidade...
    if (buzzerHumdReason) {
      buzzerHumdReason = false;
      if (!buzzerTempReason && !buzzerLightReason) {
        noTone(BUZZER_PIN);
        buzzerOn = false;
      }
    }
  }
}

void checkLightAlert() {
  // Fora da faixa => acende ledYel, liga buzzer
  if ((lightLevel < 0) || (lightLevel > 30)) {
    digitalWrite(LED_YEL, HIGH);

    // Se o buzzer não estiver ligado, ligue-o
    if (!buzzerOn) {
      tone(BUZZER_PIN, 1000);
      buzzerOn          = true;
      buzzerLightReason = true;
    }
  }
  else {
    // Dentro da faixa => desliga ledYel
    digitalWrite(LED_YEL, LOW);

    // Se o buzzer estava ligado por causa da luminosidade...
    if (buzzerLightReason) {
      buzzerLightReason = false;
      // Se não há outro motivo, desliga
      if (!buzzerTempReason && !buzzerHumdReason) {
        noTone(BUZZER_PIN);
        buzzerOn = false;
      }
    }
  }
}

void getNextAddress() {
    currentAddress += recordSize;
    if (currentAddress >= endAddress) {
        currentAddress = 0; // Volta para o começo se atingir o limite
    }
}

void get_log() {
    Serial.println("Data stored in EEPROM:");
    Serial.println("Timestamp\t\tTemperatura\tUmidade\tLuminosidade");

    for (int address = startAddress; address < endAddress; address += recordSize) {
        long timeStamp;
        int tempInt, humiInt, lumiInt;

        // Ler dados da EEPROM
        EEPROM.get(address, timeStamp);
        EEPROM.get(address + 4, tempInt);
        EEPROM.get(address + 6, humiInt);

        // Converter valores
        float temperature = tempInt / 100.0;
        float humidity = humiInt / 100.0;

        // Verificar se os dados são válidos antes de imprimir
        if (timeStamp != 0xFFFFFFFF) { // 0xFFFFFFFF é o valor padrão de uma EEPROM não inicializada
            DateTime dt(timeStamp);
            // Serial.print(dt.timestamp(DateTime::TIMESTAMP_FULL));
            
            // Formata manualmente a data e a hora
            Serial.print(dt.year());
            Serial.print("-");
            Serial.print(dt.month() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
            Serial.print(dt.month());
            Serial.print("-");
            Serial.print(dt.day() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
            Serial.print(dt.day());
            Serial.print(" ");
            Serial.print(dt.hour() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
            Serial.print(dt.hour());
            Serial.print(":");
            Serial.print(dt.minute() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
            Serial.print(dt.minute());
            Serial.print(":");
            Serial.print(dt.second() < 10 ? "0" : ""); // Adiciona zero à esquerda se necessário
            Serial.print(dt.second());
            
            Serial.print("\t");
            Serial.print(temperature);
            Serial.print(" C\t\t");
            Serial.print(humidity);
            Serial.println(" %");
        }
    }
}

void recordEEPROM(){
  now = rtc.now();

  // Calculando o deslocamento do fuso horário
  int offsetSeconds = UTC_OFFSET * 3600; // Convertendo horas para segundos
  now = now.unixtime() + offsetSeconds; // Adicionando o deslocamento ao tempo atual

  // Convertendo o novo tempo para DateTime
  adjustedTime = DateTime(now);

  // Verifica se o minuto atual é diferente do minuto do último registro
  if (adjustedTime.minute() != lastLoggedMinute) {
      lastLoggedMinute = adjustedTime.minute();

      // Verifica se os valores estão fora dos limites estabelecidos
      if (lastAvgTemp < trigger_t_min || lastAvgTemp > trigger_t_max || 
          lastAvgHumd < trigger_u_min || lastAvgHumd > trigger_u_max || 
          lightLevel < trigger_l_min || lightLevel > trigger_l_max) {
          
          // Converter valores para int para armazenamento na EEPROM
          int tempInt = (int)(lastAvgTemp * 100);
          int humiInt = (int)(lastAvgHumd * 100);

          // Escrever dados na EEPROM
          EEPROM.put(currentAddress, now.unixtime());  // Armazena timestamp
          EEPROM.put(currentAddress + 4, tempInt);     // Armazena temperatura
          EEPROM.put(currentAddress + 6, humiInt);     // Armazena umidade

          // Exibir os dados gravados no monitor serial
          Serial.println("Registro de Anomalia Gravado:");
          Serial.print("Data/Hora: ");
          Serial.print(adjustedTime.year());
          Serial.print("-");
          Serial.print(adjustedTime.month() < 10 ? "0" : ""); Serial.print(adjustedTime.month());
          Serial.print("-");
          Serial.print(adjustedTime.day() < 10 ? "0" : ""); Serial.print(adjustedTime.day());
          Serial.print(" ");
          Serial.print(adjustedTime.hour() < 10 ? "0" : ""); Serial.print(adjustedTime.hour());
          Serial.print(":");
          Serial.print(adjustedTime.minute() < 10 ? "0" : ""); Serial.print(adjustedTime.minute());
          Serial.print(":");
          Serial.print(adjustedTime.second() < 10 ? "0" : ""); Serial.println(adjustedTime.second());

          Serial.print("Temperatura: "); Serial.print(lastAvgTemp); Serial.println("°C");
          Serial.print("Umidade: "); Serial.print(lastAvgHumd); Serial.println("%");
          Serial.print("Luminosidade: "); Serial.print(lightLevel); Serial.println("%");
          Serial.println("---------------------------------");

          // Atualiza o endereço para o próximo registro
          getNextAddress();
      }
  }
}

// Exibe informações no monitor serial
void serialLog(float temp, float humid, int valorLDR, long leituraNum) {
  now = rtc.now();

  // Calculando o deslocamento do fuso horário
  int offsetSeconds = UTC_OFFSET * 3600; // Convertendo horas para segundos
  now = now.unixtime() + offsetSeconds; // Adicionando o deslocamento ao tempo atual

  // Convertendo o novo tempo para DateTime
  adjustedTime = DateTime(now);
  
  // Determina o caractere de escala
  String scaleChar = "°C";
  if (temperatureScale == 2)      scaleChar = "°F";
  else if (temperatureScale == 3) scaleChar = "°K";

  Serial.println("Leitura: " + String(leituraNum + 1));
  Serial.println("Temp: " + String(temp) + scaleChar);
  Serial.println("Ultima Temp Media: " + String(lastAvgTemp) + scaleChar);
  Serial.println("Umidade: " + String(humid) + " %");
  Serial.println("Ultima Umidade Media: " + String(lastAvgHumd) + " %");
  Serial.println("Luminosidade: " + String(lightLevel) + " %");
  Serial.println("ValorLDR: " + String(valorLDR));
  Serial.print(adjustedTime.day());
  Serial.print("/");
  Serial.print(adjustedTime.month());
  Serial.print("/");
  Serial.print(adjustedTime.year());
  Serial.print(" ");
  Serial.print(adjustedTime.hour() < 10 ? "0" : ""); // Adiciona zero à esquerda se hora for menor que 10
  Serial.print(adjustedTime.hour());
  Serial.print(":");
  Serial.print(adjustedTime.minute() < 10 ? "0" : ""); // Adiciona zero à esquerda se minuto for menor que 10
  Serial.print(adjustedTime.minute());
  Serial.print(":");
  Serial.print(adjustedTime.second() < 10 ? "0" : ""); // Adiciona zero à esquerda se segundo for menor que 10
  Serial.print(adjustedTime.second());
  Serial.print("\n");
  Serial.println("---");
}
```
---

## 🚧 Recomendações e Observações Gerais

- Garanta a correta conexão elétrica e a alimentação adequada dos sensores.
- Descarregue periodicamente os dados registrados para não ultrapassar o limite de armazenamento da EEPROM.
- O RTC precisa ser configurado previamente.

---

## 🛠️ Bibliotecas Necessárias

Instale as seguintes bibliotecas através do **Gerenciador de Bibliotecas do Arduino IDE**:

- ✅ `LiquidCrystal_I2C`
- ✅ `RTClib` (Adafruit)
- ✅ `DHT sensor library` (Adafruit)

---

## 📅 Histórico de Versões

| Versão | Data       | Descrição                     |
|--------|------------|-------------------------------|
| v1.0   | 19/03/2025 | Lançamento inicial            |

---

📌 **Observação:**  
Para contribuições, melhorias ou dúvidas, entre em contato com o autor do projeto.

---

✨ **Fim do README.md**

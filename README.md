# Challenge-Edge-Computing

Integrantes:

Matheus Henriques do Amaral - 556957
Bruno Carneiro Leão - 555563
Guilherme Melo - 555310 
Paulo Akira Okama-556840
Victor Capp - 555753



Código:

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Arduino.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int leftButtonPin = 2;   // Pino do botão esquerdo
const int rightButtonPin = 3;  // Pino do botão direito
const int redLEDPin = 13;      // Pino do LED vermelho
const int greenLEDPin = 12;    // Pino do LED verde

const int debounceDelay = 50;      // Milissegundos para eliminar ruídos
const int maxRandomDelay = 5000;   // Milissegundos para o atraso aleatório máximo

unsigned long startTime;        // Tempo de início da medição de reação
unsigned long reactionTime;     // Tempo de reação
unsigned long totalReactionTime = 0;  // Tempo de reação total
int roundsCompleted = 0;        // Número de rodadas completas

void setup() {
  pinMode(leftButtonPin, INPUT_PULLUP);   // Configura o pino do botão esquerdo como entrada com pull-up
  pinMode(rightButtonPin, INPUT_PULLUP);  // Configura o pino do botão direito como entrada com pull-up
  pinMode(redLEDPin, OUTPUT);             // Configura o pino do LED vermelho como saída
  pinMode(greenLEDPin, OUTPUT);           // Configura o pino do LED verde como saída

  lcd.init();         // Inicializa o LCD
  lcd.backlight();    // Liga a luz de fundo
  showMenu();         // Exibe o menu inicial
}

void loop() {
  if ((digitalRead(leftButtonPin) == LOW) || (digitalRead(rightButtonPin) == LOW)) {
    delay(debounceDelay);   // Elimina ruídos
    if ((digitalRead(leftButtonPin) == LOW) || (digitalRead(rightButtonPin) == LOW)) {
      startReactionTimerGame();   // Inicia o jogo de tempo de reação
      showMenu();                 // Volta ao menu inicial
    }
  }
}

void showMenu() {
  lcd.clear();        // Limpa o LCD
  lcd.setCursor(0, 0);
  lcd.print("Press any button");
  lcd.setCursor(0, 1);
  lcd.print("to start!");
}

void startReactionTimerGame() {
  lcd.clear();        // Limpa o LCD
  lcd.setCursor(0, 0);
  lcd.print("Get Ready!");
  delay(1000);
  delay(random(maxRandomDelay));   // Espera um tempo aleatório

  for (int round = 1; round <= 10; round++) {
    digitalWrite(redLEDPin, LOW);     // Desliga o LED vermelho
    digitalWrite(greenLEDPin, LOW);   // Desliga o LED verde

    // Determina qual LED acender (verde para botão esquerdo, vermelho para botão direito)
    if (random(2) == 0) {
      digitalWrite(greenLEDPin, HIGH);  // Liga o LED verde
      startTime = millis();
      while (digitalRead(leftButtonPin) == HIGH) {
        // Espera pelo pressionamento do botão esquerdo
      }
    } else {
      digitalWrite(redLEDPin, HIGH);    // Liga o LED vermelho
      startTime = millis();
      while (digitalRead(rightButtonPin) == HIGH) {
        // Espera pelo pressionamento do botão direito
      }
    }

    reactionTime = millis() - startTime;     // Calcula o tempo de reação
    totalReactionTime += reactionTime;       // Adiciona ao tempo total de reação
    roundsCompleted++;                      // Incrementa o número de rodadas completas

    // Exibe o tempo de reação da rodada atual
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Reaction time:");
    lcd.setCursor(0, 1);
    lcd.print(reactionTime);
    lcd.print(" ms");
    delay(1000);   // Mostra o tempo de reação por 1 segundo antes da próxima rodada

    // Desliga ambos os LEDs
    digitalWrite(redLEDPin, LOW);
    digitalWrite(greenLEDPin, LOW);
    delay(2000);   // Pequeno atraso antes de começar a próxima rodada
  }

  // Calcula e exibe o tempo médio de reação
  unsigned long averageReactionTime = totalReactionTime / roundsCompleted;
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Average time:");
  lcd.setCursor(0, 1);
  lcd.print(averageReactionTime);
  lcd.print("ms");
  delay(5000);   // Mostra o tempo médio de reação por 5 segundos antes de retornar ao menu

  // Exibe mensagem baseada no tempo médio de reação
  if (averageReactionTime > 175) {
    lcd.setCursor(0, 0);
    lcd.print("You were slower");
    lcd.setCursor(0, 1);
    lcd.print("than a racer.");
  } else {
    lcd.setCursor(0, 0);
    lcd.print("You were on par");
    lcd.setCursor(0, 1);
    lcd.print("with a racer.");
  }
  delay(5000);
}

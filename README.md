#include <LiquidCrystal.h>
#include <DHT.h>

#define PINO_SENSOR 6
#define TIPO_SENSOR DHT11

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
DHT sensorDHT(PINO_SENSOR, TIPO_SENSOR);

const byte NUM_AMOSTRAS = 10;
const unsigned long INTERVALO_LEITURA = 900000UL;  

float vetorTemp[NUM_AMOSTRAS];
float vetorUmid[NUM_AMOSTRAS];
byte indiceAtual = 0;

unsigned long momentoAnterior = 0;

void registrarLeitura();
void exibirNoLCD(float t, float u);
float calcularMediaFiltrada(float v[], byte tamanho);
void ordenarArray(float v[], byte tamanho);

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2);
  lcd.print("Inicializando...");
  sensorDHT.begin();
}

void loop() {
  unsigned long agora = millis();

  if (agora - momentoAnterior >= INTERVALO_LEITURA) {
    momentoAnterior = agora;
    registrarLeitura();
  }
}

void registrarLeitura() {
  float umid = sensorDHT.readHumidity();
  float temp = sensorDHT.readTemperature();

  if (isnan(umid) || isnan(temp)) {
    lcd.setCursor(0, 1);
    lcd.print("Erro na leitura ");
    return;
  }

  vetorTemp[indiceAtual] = temp;
  vetorUmid[indiceAtual] = umid;
  indiceAtual++;

  Serial.print("Amostra ");
  Serial.print(indiceAtual);
  Serial.print(" de ");
  Serial.println(NUM_AMOSTRAS);

  if (indiceAtual >= NUM_AMOSTRAS) {
    float tempAux[NUM_AMOSTRAS];
    float umidAux[NUM_AMOSTRAS];

    for (byte i = 0; i < NUM_AMOSTRAS; i++) {
      tempAux[i] = vetorTemp[i];
      umidAux[i] = vetorUmid[i];
    }

    ordenarArray(tempAux, NUM_AMOSTRAS);
    ordenarArray(umidAux, NUM_AMOSTRAS);

    float mediaTemp = calcularMediaFiltrada(tempAux, NUM_AMOSTRAS);
    float mediaUmid = calcularMediaFiltrada(umidAux, NUM_AMOSTRAS);

    Serial.print("Temp filtrada: ");
    Serial.print(mediaTemp, 1);
    Serial.println("C");

    Serial.print("Umid filtrada: ");
    Serial.print(mediaUmid, 1);
    Serial.println("%");

    exibirNoLCD(mediaTemp, mediaUmid);
    indiceAtual = 0;
  }
}

void exibirNoLCD(float t, float u) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("T:");
  lcd.print(t, 1);
  lcd.print((char)223);
  lcd.print("C");

  lcd.setCursor(0, 1);
  lcd.print("U:");
  lcd.print(u, 1);
  lcd.print("%");
}

void ordenarArray(float v[], byte tamanho) {
  for (byte i = 1; i < tamanho; i++) {
    float atual = v[i];
    int j = i - 1;
    while (j >= 0 && v[j] > atual) {
      v[j + 1] = v[j];
      j--;
    }
    v[j + 1] = atual;
  }
}

float calcularMediaFiltrada(float v[], byte tamanho) {
  if (tamanho < 3) return v[0];
  float soma = 0;
  for (byte i = 1; i < tamanho - 1; i++) soma += v[i];
  return soma / (tamanho - 2);y}
}

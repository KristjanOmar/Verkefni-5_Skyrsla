# Skýrsla

## Myndband

---

## Lýsing

Róbotinn fer upp og niður, opnar og lokar kjálka, átti að öskra (virkaði ekki af einhverri ástæðu) og augun blikka

---

## Ljósmyndir

![20230301_171447](https://user-images.githubusercontent.com/117899282/222213864-9d2990cc-18a3-44de-9c2d-12a3dee110ac.jpg)

![20230301_171501](https://user-images.githubusercontent.com/117899282/222214055-9a06db58-9ca1-4ef9-a898-06024913d2b4.jpg)

![20230301_171509](https://user-images.githubusercontent.com/117899282/222214246-da419a07-754b-4493-b453-2eec22a39fe6.jpg)

---

## Kóði

```cpp
// Perukóði
#include "tdelay.h" // sambærilegt og import í python

const int LED1 = 7; // breytur fyrir led perurnar

bool LED1_on = true; // stöðubreytur, halda utan um hvort kveikt eða slökkt er á perunum

TDelay led1_delay; // "delay" breytur fyrir hverja peru

int lagmarks_bidtimi = 250;
int hamarks_bidtimi = 750;
// Perukóði


// Servo kóði
#include <Servo.h> // Sambærilegt og import í python

Servo motor; // bý til tilvik af Servo klasanum
int motor_pinni = 9; // pinninn sem ég nota til að stjórna mótornum
int stefna = 0; // stefnan sem mótorinn á að snúa í
bool hreyfing_til_haegri = true; // er mótorinn að snúast til hægri eða vinstri

TDelay motor_bid(10); // bíða í 50 ms. milli færslna
// Servo kóði


// DC kóði
const int HRADI = 6; // Verður að vera PWM pinni
const int STEFNA_A = 5;
const int STEFNA_B = 4;

/*  motórinn á að bíða í 1 sek
    síðan að fara áfram í 2 sek
    síðan að stoppa í 3 sek
    síðan að bakka í 2 sek 
    og endurtaka */
// bý til senu (e. scene) með viðburðum
// ákveð: 0 er stopp, 1 er áfram og -1 er bakka
int sena[] = {0, 1, 0, -1};
// hversu lengi á hver viðburður að vara
int senu_timi[] = {1000, 2000, 3000, 2000}; 
// teljari sem veit hvar senan er stödd
int senu_teljari = 0;
// breyta sem veit hversu margir viðburðir eru í senunni
const int FJOLDI_VIDBURDA = 3;

TDelay motor_delay(1000); // hversu lengi er fyrsti viðburður

void afram(int hradi);
void bakka(int hradi);
void stoppa();
// DC kóði


// Fjarlægðarmælir
const int ECHO = 2; 
const int TRIG = 3; 
int distance = 0;  
int fjarlaegd();  // fall sem sér um fjarlægðamælinguna, skilar fjarlægð í cm. 
// Fjarlægðarmælir


void setup() {
  // Servo kóði
  motor.attach(motor_pinni); // segi servo tilvikinu hvaða pinna á að nota
  motor.write(stefna); // í þessu tilfelli á mótorinn að byrja í 0°

  // Servo kóði


  // Perur
  pinMode(LED1, OUTPUT); // skilgreina hvernig pinninn virkar sem peran er tengd í

  randomSeed(analogRead(A0)); // frumstilla random fallið

  led1_delay.setBidtimi(random(lagmarks_bidtimi, hamarks_bidtimi)); // setja upphafsbiðtíma fyrir hverja peru
  // Perur


  // DC kóði
  pinMode(HRADI, OUTPUT);
    pinMode(STEFNA_A, OUTPUT);
    pinMode(STEFNA_B, OUTPUT);

    stoppa(); // upphafsstaðan, verður stopp þar til annað er ákveðið
  // DC kóði


  // Fjarlægðarmælir
  Serial.begin(9600); 
  pinMode(TRIG,OUTPUT);
  pinMode(ECHO,INPUT);
  // Fjarlægðarmælir
}

void loop() {
    // Fjarlægðarmælir
  distance = fjarlaegd();           // fáum fjarlægðamælingu í cm.
  Serial.print("Fjarlaegd: ");     // til að sjá mælingu í skjánum
  Serial.println(fjarlaegd());     // til að sjá mælingu í skjánum

  

  // fjarlægð getur ekki verið mínustala
  if(distance < 0) { 
    distance = 0;
  } 

  // ef fjarlægð í hlut er minna en 50 cm, má ekki vera neikvæð tala.
  if(distance < 50 && distance != 0) {  
    // setja sýninguna af stað

  // Servo kóði
  if(motor_bid.timiLidinn()) {
    if(hreyfing_til_haegri == true) {
      stefna++; // jafngildir stefna += 1 og stefna = stefna + 1
    } else { // mótorinn er að hreyfast til hægri
      stefna--; // jafngildir stefna -= 1 og stefna = stefna - 1
    }
    if(stefna == 0 || stefna == 120) { // ef mótirnn er kominn út á enda, víxla áttunum
      hreyfing_til_haegri = !hreyfing_til_haegri;
    }
    motor.write(stefna);
  }
  // Servo kóði


  // Perukóði
  if(led1_delay.timiLidinn()) { // ef biðtiminn er liðinn
    LED1_on = !LED1_on; // víxla (e. toggle) stöðubreytu perunnar
    led1_delay.setBidtimi(random(lagmarks_bidtimi, hamarks_bidtimi)); // setja nýjan random biðtíma
  }

  digitalWrite(LED1, LED1_on); // kveikir eða slekkur á perunni true jafngildir HIGH (5V)
  // Perukóði


  // DC kóði
  if(motor_delay.timiLidinn() == true) {
        senu_teljari = (senu_teljari + 1) % FJOLDI_VIDBURDA;
        motor_delay.setBidtimi(senu_timi[senu_teljari]);
    }
    if(sena[senu_teljari] == 1) {
        afram(255);
    } else if(sena[senu_teljari] == -1) {
        bakka(255);
    } else {
        stoppa();
    }
  // DC kóði


  } 
  
  // Fjarlægðarmælir
}

void afram(int hradi) {
    digitalWrite(STEFNA_A, HIGH);
    digitalWrite(STEFNA_B, LOW);
    analogWrite(HRADI, hradi);
}

void bakka(int hradi) {
    digitalWrite(STEFNA_A, LOW);
    digitalWrite(STEFNA_B, HIGH);
    analogWrite(HRADI, hradi);
}

void stoppa() {
    digitalWrite(STEFNA_A, LOW);
    digitalWrite(STEFNA_B, LOW);
    analogWrite(HRADI, 0);
}

int fjarlaegd() {
  
    // sendir út púlsa
    digitalWrite(TRIG,LOW);
    delayMicroseconds(2); // of stutt delay til að skipta máli
    digitalWrite(TRIG,HIGH);
    delayMicroseconds(10); // of stutt delay til að skipta máli
    digitalWrite(TRIG,LOW);

    // mælt hvað púlsarnir voru lengi að berast til baka
    return (int)pulseIn(ECHO,HIGH)/58.0; // deilt með 58 til að breyta í cm
  
}
```
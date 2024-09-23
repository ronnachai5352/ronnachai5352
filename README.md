 if (fanC_ == "ON ") {#include <Wire.h>
#include <LiquidCrystal.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 2, 1, 0, 4, 5, 6, 7);
#include <Keypad.h>
#include <Keypad_I2C.h>
#define I2CADDR 0x20
float value_output;
const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  { '1', '2', '3', 'A' },
  { '4', '5', '6', 'B' },
  { '7', '8', '9', 'C' },
  { '*', '0', '#', 'D' }
};
byte rowPins[ROWS] = { 7, 6, 5, 4 };
byte colPins[COLS] = { 3, 2, 1, 0 };
Keypad_I2C keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS, I2CADDR, PCF8574);
char key = NO_KEY;
#include <OneWire.h>
#include <DallasTemperature.h>
#define ONE_WIRE_BUS_2 2
OneWire oneWire_2(ONE_WIRE_BUS_2);
DallasTemperature sensors_2(&oneWire_2);  //ในโรงเรือน
#define ONE_WIRE_BUS_3 3
OneWire oneWire_3(ONE_WIRE_BUS_3);
DallasTemperature sensors_3(&oneWire_3);  //นอกโรงเรือน
int up__ = 1000;
String ValueA_ = "";
String ValueB_ = "";
String ValueC_ = "";
const int pumpA_23 = 37;
const int pumpB_25 = 33;
const int pumpfog_27 = 35;
const int fanA_29 = 29;
const int fanB_31 = 31;
const int FanC_33 = 27;
const int lamp_35 = 25;
const int spare_37 = 23;
String pump_ = "OFF";
String fog_ = "OFF";
String fer_ = "OFF";
String fanA_ = "OFF";
String fanB_ = "OFF";
String fanC_ = "OFF";
int punpall = 0;
int fogall = 0;
int fanall = 0;
int ValueA = 0;
int ValueB = 0;
int ValueC = 0;
String page = "A";
String status = "STOP ";
String mode = "MANUAL";
String temp_1_ = "000.0C";
String temp_2_ = "000.0C";
String moistur_ = "000%";
String ph_ = "00";
float temp_1;
float temp_2;
int moistur;
float ph;
int interlock_page = 0;
int delay_time = 0;
int sec = 100;
int temp_swap = 0;
int settingRow = 0;
int settingCol = 1;
int settingLine = 0;
float temp_min = 25;
float temp_max = 35;
int moistur_int = 40;
String temp_min_ = "";
String temp_max_ = "";
String moistur_int_ = "";
int readA = 100;

void setup() {
  Serial.begin(9600);
  lcd.begin(20, 4);
  lcd.setBacklightPin(3, POSITIVE);
  lcd.setBacklight(HIGH);
  pinMode(pumpA_23, OUTPUT);
  pinMode(pumpB_25, OUTPUT);
  pinMode(pumpfog_27, OUTPUT);
  pinMode(fanA_29, OUTPUT);
  pinMode(fanB_31, OUTPUT);
  pinMode(FanC_33, OUTPUT);
  pinMode(lamp_35, OUTPUT);
  pinMode(spare_37, OUTPUT);
  off_mode();
  sensors_2.begin();
  sensors_3.begin();
  Wire.begin();
  keypad.begin(makeKeymap(keys));
}

void loop() {
  delay(1);
  readA++;
  delay_time++;
  key = keypad.getKey();
  if (page == "A") {
    page_A();
  } else if (page == "B") {
    page_B();
  } else if (page == "C") {
    page_C();
  } else if (page == "D") {
    page_D();
  }
  if (readA >= 25) {
    readA = 0;
    moistur_ph();
  }


  if (page == "A" || page == "B") {
    if (key == '*') {
      status = "START";
    } else if (key == '#') {
      status = "STOP ";
    }
  }
  if (page == "A") {
    if (status == "STOP ") {
      if (key == 'A' && mode == "MANUAL") {
        mode = "  AUTO";
      } else if (key == 'A' && mode == "  AUTO") {
        mode = "MANUAL";
      }
    }
  }
  if (key == 'A' && page != "A") {
    page = "A";
    interlock_page = 0;
  } else if (key == 'B' && page != "B") {
    page = "B";
    interlock_page = 0;
  } else if (key == 'C' && page != "C" && status == "STOP ") {
    page = "C";
    interlock_page = 0;
  } else if (key == 'D' && page != "D") {
    up__ = 1000;
    page = "D";
    interlock_page = 0;
  }
  if (page == "A" || page == "B") {
    if (delay_time >= 25) {
      delay_time = 0;
      sec++;
    }

    if (sec >= 1) {
      if (page == "B" && mode == "  AUTO") {
        temp_read();
      } else if (page == "A") {
        temp_read();
      }
      sec = 0;
    }
  }
  if (status == "STOP ") {
    digitalWrite(lamp_35, HIGH);
    off_mode();
  } else {
    digitalWrite(lamp_35, LOW);
    if (mode == "MANUAL") {
      outPutManual();
    }
  }
}
void page_D() {
  if (interlock_page == 0) {
    interlock_page++;
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("MOISTUR TOTAL: ");
    lcd.setCursor(0, 1);
    lcd.print("MOISTUR A: ");
    lcd.setCursor(0, 2);
    lcd.print("MOISTUR B: ");
    lcd.setCursor(0, 3);
    lcd.print("MOISTUR C: ");
  }
  mois();
  up__++;
  if (up__ >= 100) {
    up__ = 0;
    lcd.setCursor(15, 0);
    lcd.print(moistur_);
    lcd.setCursor(11, 1);
    lcd.print(ValueA_);
    lcd.setCursor(11, 2);
    lcd.print(ValueB_);
    lcd.setCursor(11, 3);
    lcd.print(ValueC_);
  }
}
void page_A() {
  if (interlock_page == 0) {
    settingCol = 1;
    interlock_page++;
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("CONTROL:START MANUAL");
    lcd.setCursor(0, 1);
    lcd.print("T1:000.0C  T2:000.0C");
    lcd.setCursor(0, 2);
    lcd.print("MOISTUR:000%  pH:0.0");
    lcd.setCursor(0, 3);
    lcd.print("PUMP:0  FOG:0  FAN:0");
  }
  if (status == "START" && mode == "  AUTO") {
    if (temp_1 <= temp_min) {
      fog_ = "OFF";
      fanA_ = "OFF";
      fanB_ = "OFF";
      fanC_ = "OFF";
    } else if (temp_1 >= temp_max) {
      fog_ = "ON ";
      fanA_ = "ON ";
      fanB_ = "ON ";
      fanC_ = "ON ";
    }
    if (moistur <= moistur_int) {
      pump_ = "ON ";
    } else {
      pump_ = "OFF";
    }
  }

  update();
  lcd.setCursor(8, 0);
  lcd.print(status);
  lcd.setCursor(14, 0);
  lcd.print(mode);
  lcd.setCursor(3, 1);
  lcd.print(temp_1_);
  lcd.setCursor(14, 1);
  lcd.print(temp_2_);
  lcd.setCursor(8, 2);
  lcd.print(moistur_);
  lcd.setCursor(17, 2);
  lcd.print(ph_);
  lcd.setCursor(5, 3);
  lcd.print(punpall);
  lcd.setCursor(12, 3);
  lcd.print(fogall);
  lcd.setCursor(19, 3);
  lcd.print(fanall);
}
void page_B() {
  if (interlock_page == 0) {
    settingCol = 1;
    interlock_page++;
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("STATUS:");
    lcd.setCursor(0, 1);
    lcd.print("1PUMP:");
    lcd.setCursor(0, 2);
    lcd.print("2P_FOG:");
    lcd.setCursor(0, 3);
    lcd.print("3P_FER:");
    lcd.setCursor(11, 1);
    lcd.print("4FANA:");
    lcd.setCursor(11, 2);
    lcd.print("5FANB:");
    lcd.setCursor(11, 3);
    lcd.print("6FANC:");
  }
  if (status == "START" && mode == "  AUTO") {
    if (temp_1 <= temp_min) {
      fog_ = "OFF";
      fanA_ = "OFF";
      fanB_ = "OFF";
      fanC_ = "OFF";
    } else if (temp_1 >= temp_max) {
      fog_ = "ON ";
      fanA_ = "ON ";
      fanB_ = "ON ";
      fanC_ = "ON ";
    }
    if (moistur <= moistur_int) {
      pump_ = "ON ";
    } else {
      pump_ = "OFF";
    }
  }
  update();
  lcd.setCursor(8, 0);
  lcd.print(status);
  lcd.setCursor(14, 0);
  lcd.print(mode);

  lcd.setCursor(6, 1);
  lcd.print(pump_);
  lcd.setCursor(7, 2);
  lcd.print(fog_);
  lcd.setCursor(7, 3);
  lcd.print(fer_);
  lcd.setCursor(17, 1);
  lcd.print(fanA_);
  lcd.setCursor(17, 2);
  lcd.print(fanB_);
  lcd.setCursor(17, 3);
  lcd.print(fanC_);
}
void page_C() {
  temp_min_ = String(temp_min, 0);
  if (temp_min < 10) {
    temp_min_ = "00" + String(temp_min, 0);
  } else if (temp_min < 100) {
    temp_min_ = "0" + String(temp_min, 0);
  }

  temp_max_ = String(temp_max, 0);
  if (temp_max < 10) {
    temp_max_ = "00" + String(temp_max, 0);
  } else if (temp_max < 100) {
    temp_max_ = "0" + String(temp_max, 0);
  }
  moistur_int_ = String(moistur_int);
  if (moistur_int < 10) {
    moistur_int_ = "00" + String(moistur_int);
  } else if (moistur_int < 100) {
    moistur_int_ = "0" + String(moistur_int);
  }
  lcd.clear();
  lcd.setCursor(2, 0);
  lcd.print("SETTING CONTROL");
  lcd.setCursor(0, 1);
  lcd.print("TEMP1 MAX: ");
  lcd.print(temp_max_);
  lcd.print(" C");
  lcd.setCursor(0, 2);
  lcd.print("TEMP1 MIN: ");
  lcd.print(temp_min_);
  lcd.print(" C");
  lcd.setCursor(1, 3);
  lcd.print("MOISTUR : ");
  lcd.print(moistur_int_);
  lcd.print(" %");
  lcd.setCursor(19, settingCol);
  lcd.print("<");
  char ketB = "";
  if (settingCol == 1) {
    ketB = function_inputKeypadToLcd(3, temp_max, 0, 11, settingCol, page);
    temp_max = value_output;
    if (temp_max < temp_min) {
      temp_min = temp_max - 1;
      if (temp_max <= 0) {
        temp_max = 1;
        temp_min = 0;
      }
    }
  } else if (settingCol == 2) {
    ketB = function_inputKeypadToLcd(3, temp_min, 0, 11, settingCol, page);
    temp_min = value_output;
    if (temp_max < temp_min) {
      temp_max = temp_min + 1;
      if (temp_max > 999) {
        temp_max = 999;
        temp_min = 998;
      }
    }
  } else if (settingCol == 3) {
    ketB = function_inputKeypadToLcd(3, moistur_int, 0, 11, settingCol, page);
    moistur_int = value_output;
  }
  settingCol++;
  if (settingCol > 3) {
    settingCol = 1;
  }
  if (ketB == 'A') {
    page = "A";
  } else if (ketB == 'B') {
    page = "B";
  }
}
void update() {
  punpall = 0;
  if (pump_ == "ON ") {
    punpall++;
    digitalWrite(pumpA_23, LOW);
  } else {
    digitalWrite(pumpA_23, HIGH);
  }
  if (fer_ == "ON ") {
    punpall++;
    digitalWrite(pumpB_25, LOW);
  } else {
    digitalWrite(pumpB_25, HIGH);
  }
  fogall = 0;
  if (fog_ == "ON ") {
    fogall++;
    digitalWrite(pumpfog_27, LOW);
  } else {
    digitalWrite(pumpfog_27, HIGH);
  }
  fanall = 0;
  if (fanA_ == "ON ") {
    fanall++;
    digitalWrite(fanA_29, LOW);
  } else {
    digitalWrite(fanA_29, HIGH);
  }
  if (fanB_ == "ON ") {
    fanall++;
    digitalWrite(fanB_31, LOW);
  } else {
    digitalWrite(fanB_31, HIGH);
  }

    fanall++;
    digitalWrite(FanC_33, LOW);
  } else {
    digitalWrite(FanC_33, HIGH);
  }
}
void temp_read() {
  if (temp_swap == 0) {
    temp_swap++;
    sensors_2.requestTemperatures();
  } else {
    temp_swap = 0;
    sensors_3.requestTemperatures();
  }
  temp_1 = sensors_2.getTempCByIndex(0);
  temp_2 = sensors_3.getTempCByIndex(0);
  temp_1_ = String(temp_1, 1);
  if (temp_1 < 10) {
    temp_1_ = "00" + String(temp_1, 1);
  } else if (temp_1 < 100) {
    temp_1_ = "0" + String(temp_1, 1);
  }
  temp_2_ = String(temp_2, 1);
  if (temp_2 < 10) {
    temp_2_ = "00" + String(temp_2, 1);
  } else if (temp_2 < 100) {
    temp_2_ = "0" + String(temp_2, 1);
  }
  temp_1_ = temp_1_ + "C";
  temp_2_ = temp_2_ + "C";
}

void moistur_ph() {
  mois();
  int Value = analogRead(A0);
  int value_ph = map(Value, 830, 890, 80, 35);
  ph = value_ph;
  ph = ph / 10;
  if (ph > 8.0) {
    ph = 0;
  } else if (ph < 3.5) {
    ph = 3.5;
  }
  ph_ = String(ph, 1);
}
void mois() {
  ValueA = map(analogRead(A1), 0, 400, 0, 100);
  ValueB = map(analogRead(A2), 0, 900, 0, 100);
  ValueC = map(analogRead(A3), 0, 700, 0, 100);

  if (ValueA > 100) {
    ValueA = 100;
  }
  if (ValueB > 100) {
    ValueB = 100;
  }
  if (ValueC > 100) {
    ValueC = 100;
  }


  if (ValueA < 10) {
    ValueA_ = "00" + String(ValueA);
  } else if (ValueA < 100) {
    ValueA_ = "0" + String(ValueA);
  } else {
    ValueA_ = String(ValueA);
  }
  if (ValueB < 10) {
    ValueB_ = "00" + String(ValueB);
  } else if (ValueB < 100) {
    ValueB_ = "0" + String(ValueB);
  } else {
    ValueB_ = String(ValueB);
  }
  if (ValueC < 10) {
    ValueC_ = "00" + String(ValueC);
  } else if (ValueC < 100) {
    ValueC_ = "0" + String(ValueC);
  } else {
    ValueC_ = String(ValueC);
  }
  ValueA_ = ValueA_ + "%";
  ValueB_ = ValueB_ + "%";
  ValueC_ = ValueC_ + "%";
  if (ValueA == 0 || ValueB == 0 || ValueC == 0) {
    moistur = 0;
  } else {
    moistur = (ValueA + ValueB + ValueC) / 3;
  }
  moistur_ = String(moistur);
  if (moistur < 10) {
    moistur_ = "00" + String(moistur);
  } else if (moistur < 100) {
    moistur_ = "0" + String(moistur);
  }
  moistur_ = moistur_ + "%";
}
void outPutManual() {
  if (key == '1') {
    if (pump_ == "OFF") {
      pump_ = "ON ";
    } else {
      pump_ = "OFF";
    }
  } else if (key == '2') {
    if (fog_ == "OFF") {
      fog_ = "ON ";
    } else {
      fog_ = "OFF";
    }
  } else if (key == '3') {
    if (fer_ == "OFF") {
      fer_ = "ON ";
    } else {
      fer_ = "OFF";
    }
  } else if (key == '4') {
    if (fanA_ == "OFF") {
      fanA_ = "ON ";
    } else {
      fanA_ = "OFF";
    }
  } else if (key == '5') {
    if (fanB_ == "OFF") {
      fanB_ = "ON ";
    } else {
      fanB_ = "OFF";
    }
  } else if (key == '6') {
    if (fanC_ == "OFF") {
      fanC_ = "ON ";
    } else {
      fanC_ = "OFF";
    }
  }
}
void off_mode() {
  digitalWrite(pumpA_23, HIGH);
  digitalWrite(pumpB_25, HIGH);
  digitalWrite(pumpfog_27, HIGH);
  digitalWrite(fanA_29, HIGH);
  digitalWrite(fanB_31, HIGH);
  digitalWrite(FanC_33, HIGH);
  digitalWrite(spare_37, HIGH);
  pump_ = "OFF";
  fog_ = "OFF";
  fer_ = "OFF";
  fanA_ = "OFF";
  fanB_ = "OFF";
  fanC_ = "OFF";
}
int function_inputKeypadToLcd(int numChar, int vaLue, float float_or, int rows_lcd, int col_lcd, String page_lcd) {
  int det_float = 0;
  int cournapp = 0;
  int intpurChar = 0;
  int float_add = numChar;
  int input_keys = 0;
  if (float_or == 1) {
    float_add++;
  }
  char add_vaLue[float_add];
  while (1) {
    delay(1);
    char keys = keypad.getKey();
    int number = String(keys).toInt();
    if (keys != '0' && keys != '1' && keys != '2' && keys != '3' && keys != '4' && keys != '5' && keys != '6' && keys != '7' && keys != '8' && keys != '9' && keys != NO_KEY) {
      String convert_char = "";
      float outPut = 0.00;
      for (int i = 0; i < float_add; i++) {
        convert_char = convert_char + String(add_vaLue[i]);
      }
      outPut = convert_char.toFloat();
      if (outPut <= 0) {
        outPut = vaLue;
      } else if (input_keys == 0) {
        outPut = vaLue;
      }
      value_output = outPut;
      if (keys == 'A' || keys == 'B' || keys == 'D') {
        return keys;
      }
      //break;
    } else if (number >= 0 && number <= 9 && keys != NO_KEY) {
      input_keys = 1;
      if (float_or == 1 && cournapp != numChar) {
        add_vaLue[intpurChar] = keys;
        intpurChar++;
        cournapp++;
        lcd.setCursor(rows_lcd, col_lcd);
        lcd.print(number);
        rows_lcd++;
        if (cournapp == numChar - 1) {
          intpurChar++;
          rows_lcd++;
        }
        if (det_float == 0) {
          det_float = 1;
          for (int i = rows_lcd + 1; i < rows_lcd + numChar + 1; i++) {
            lcd.print("_");
          }
          lcd.setCursor((rows_lcd + numChar) - 2, col_lcd);
          lcd.print(".");
          add_vaLue[float_add - 2] = '.';
        }
      } else if (float_or == 0 && cournapp != numChar) {
        add_vaLue[intpurChar] = keys;
        intpurChar++;
        cournapp++;
        lcd.setCursor(rows_lcd, col_lcd);
        lcd.print(number);
        rows_lcd++;
        if (det_float == 0) {
          det_float = 1;
          for (int i = rows_lcd + 1; i < rows_lcd + numChar; i++) {
            lcd.print("_");
          }
        }
      }
    }
  }
}

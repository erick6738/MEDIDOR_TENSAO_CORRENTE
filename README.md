# Medidor de Tensão e Corrente DC com Arduino + LCD 16×2  
### Explicação completa com fórmulas, divisor de tensão e resistor shunt

<img width="833" height="759" alt="image" src="https://github.com/user-attachments/assets/146fe0b2-b4c4-4d9e-a3fc-f075476be0b8" />


*Tensão até ≈ 50 V DC | Corrente até ≈ 50 mA (fácil de aumentar)*

Projeto 100 % funcional no Tinkercad e no hardware real. Perfeito para laboratório, bancada, fontes, baterias, painéis solares pequenos, etc.

---

### Esquemático rápido (mesmo que você já tenha montado)
R1 = 14 kΩ
Ventrada (+) ─────┬───────┐
│       │
A0       R2 = 10 kΩ
│       │
GND      GND
Corrente da carga passa pelo shunt:
(+) da carga ─── Shunt 100 Ω ─── (–) da carga
│
A1 do Arduino

### Medição de Tensão – Divisor Resistivo (fórmula completa)

O Arduino só lê até 5 V → usamos dois resistores para reduzir a tensão alta com segurança.

**Fórmula do divisor de tensão:**

V_{A0} = V_{entrada} * {R2}/{R1 + R2}}

V_{entrada} = V_{A0} * {R1 + R2}/{R2}


Com R1 = 14 kΩ e R2 = 10 kΩ → fator de correção = 2,4  
→ Tensão máxima segura = 5 V × 2,4 = **50 V**

No código isso vira:

float tensao_real = v_adc * ((R1 + R2) / R2);

Medição de Corrente – Resistor Shunt (fórmula completa)
A corrente passa por um resistor de valor conhecido → medimos a queda de tensão nele → calculamos a corrente.
Lei de Ohm no shunt:
I = V/R
Com Rshunt = 100 Ω:

50 mA → 5 V no pino A1 → limite seguro do Arduino

Quer medir mais corrente no futuro?
→ 500 mA → use shunt de 10 Ω
→ 5 A → use shunt de 1 Ω ou sensor dedicado (ACS712/INA219)
No código:
C++Copiarfloat corrente = v_shunt / Rshunt;           // em Ampères
float corrente_mA = corrente * 1000.0;        // em miliampères

Código completo (pronto para upload)
C++ 
#include <LiquidCrystal.h>

// LCD: RS, EN, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// Configurações do divisor de tensão
float R1 = 14000.0;   // 14k ohms
float R2 = 10000.0;   // 10k ohms

// Resistor shunt para medir corrente em A
float Rshunt = 100;   // ohm -> as casas decimais podem estar erradas, alterar esse valor para 100 te da a medição em A. Rshunt = 1 te da em mA

void setup() {
  lcd.begin(16, 2); 
  lcd.print("Medidor pronto");
  delay(1500);
  lcd.clear();
}

void loop() {
  // ----- Medição de tensão -----
  int leituraV = analogRead(A0);                 // lê tensão do divisor
  float v_adc = leituraV * (5.0 / 1023.0);      // converte para volts Arduino
  float tensao_real = v_adc * ((R1 + R2) / R2); // corrige pelo divisor

  // ----- Medição de corrente -----
  int leituraI = analogRead(A1);                // lê tensão no shunt
  float v_shunt = leituraI * (5.0 / 1023.0);   // converte para volts
  float corrente = v_shunt / Rshunt;           // calcula corrente

  // ----- Exibir no LCD -----
  lcd.setCursor(0, 0);
  lcd.print("V: ");
  lcd.print(tensao_real, 2); // duas casas decimais
  lcd.print(" V   ");         // espaços para limpar restos

  lcd.setCursor(0, 1);
  lcd.print("I: ");
  lcd.print(corrente, 2);    // duas casas decimais
  lcd.print(" mA   ");
  delay(300); // atualiza a cada 0,3s
}

Feito em 2025 por Adam – Técnico em Eletrônica

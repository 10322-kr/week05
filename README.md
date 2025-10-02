# week05
//===================================================================
// 초음파 센서 3색 LED 및 부저 경고 시스템 (핀 충돌 문제 해결)
// BUZZER_PIN: D9으로 이동하여 시리얼 충돌 해결
//===================================================================

// -------- 핀 정의 (수정됨) --------
const int TRIG_PIN = 10;
const int ECHO_PIN = 11;
const int RED_LED_PIN = 12;
const int BLUE_LED_PIN = 13;
const int WHITE_LED_PIN = 14;  // D14 (A0)
const int BUZZER_PIN = 9;      // 수동 부저 핀: D9으로 변경!

// 7세그먼트 핀 연결 (a, b, c, d, e, f, g, dp 순서) - D1 핀 재사용
const int segPins[8] = {2, 3, 4, 5, 6, 7, 8, 1}; // D9 -> D1로 변경

// -------- 7세그먼트 패턴 (0~8) --------
const byte digits_map[9] = {
    0b00000000,  // [0] : 모두 꺼짐
    0b00000110,  // [1]
    0b01011011,  // [2]
    0b01001111,  // [3] (안전거리)
    0b01100110,  // [4]
    0b01101101,  // [5]
    0b01111101,  // [6]
    0b00000111,  // [7]
    0b01111111   // [8] (가장 가까움/위험)
};

// -------- 거리 측정 함수 (mm 단위 반환) --------
long measureDistance() {
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);
    
    long duration = pulseIn(ECHO_PIN, HIGH); 
    
    // 펄스 타임아웃/오류 처리
    if (duration == 0 || duration < 116) return 400;

    long distance_mm = (duration / 58) * 10; 
    if (distance_mm > 4000) return 4000;
    
    return distance_mm;
}

// -------- 7세그먼트 숫자 표시 함수 --------
void displayDigit(int num) {
    if (num < 1 || num > 8) num = 0; 

    byte pattern = digits_map[num];
    for (int i = 0; i < 7; i++) {
        digitalWrite(segPins[i], bitRead(pattern, i));
    }
    // dp 핀 (D1)은 끔
    digitalWrite(segPins[7], LOW);
}

// -------- 설정 (Setup) --------
void setup() {
    // D1 핀을 사용하지 않아 시리얼 통신이 안정적으로 작동합니다.
    Serial.begin(9600);
    
    pinMode(TRIG_PIN, OUTPUT);
    pinMode(ECHO_PIN, INPUT);
    
    pinMode(RED_LED_PIN, OUTPUT);
    pinMode(BLUE_LED_PIN, OUTPUT);
    pinMode(WHITE_LED_PIN, OUTPUT);
    pinMode(BUZZER_PIN, OUTPUT); // D9 핀으로 변경

    for (int i = 0; i < 8; i++) {
        pinMode(segPins[i], OUTPUT);
        digitalWrite(segPins[i], LOW);
    }
}

// -------- 메인 루프 (Loop) --------
void loop() {
    long distance_mm = measureDistance(); 
    int output_num = 0;
    
    digitalWrite(RED_LED_PIN, LOW);
    digitalWrite(BLUE_LED_PIN, LOW);
    digitalWrite(WHITE_LED_PIN, LOW);
    noTone(BUZZER_PIN);

    // 1. 거리에 따른 7세그먼트 출력 숫자 결정 (25mm 단위)
    if (distance_mm < 25) {
        output_num = 8;
    } else if (distance_mm < 50) {
        output_num = 7;
    } else if (distance_mm < 75) {
        output_num = 6;
    } else if (distance_mm < 100) {
        output_num = 5;
    } else if (distance_mm < 125) {
        output_num = 4;
    } else if (distance_mm < 150) {
        output_num = 3;
    } else if (distance_mm < 175) {
        output_num = 2;
    } else if (distance_mm < 200) {
        output_num = 1;
    } else { 
        output_num = 0;
    }

    displayDigit(output_num);
    
    // 3. LED 및 부저 제어
    if (output_num >= 6) {
        digitalWrite(RED_LED_PIN, HIGH);
        
        if (output_num == 8) {
            tone(BUZZER_PIN, 4500); 
            delay(100); 
            noTone(BUZZER_PIN);
        } else {
            tone(BUZZER_PIN, 2500); 
            delay(50); 
            noTone(BUZZER_PIN);
        }
    } else if (output_num >= 4 && output_num <= 5) {
        digitalWrite(WHITE_LED_PIN, HIGH);
    } else if (output_num == 3) {
        digitalWrite(BLUE_LED_PIN, HIGH);
    }

    // D1 핀을 사용하지 않아 시리얼 통신이 안정적으로 작동합니다.
    Serial.print("거리: ");
    Serial.print(distance_mm);
    Serial.print("mm, 출력 숫자: ");
    Serial.println(output_num);

    delay(200); 
}

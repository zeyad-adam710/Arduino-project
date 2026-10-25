// تعريف المستشعرات والمنافذ
const int trigPinFront = A1; // مستشعر أمامي
const int echoPinFront = A0;
const int trigPinLeft = A3;  // مستشعر يساري
const int echoPinLeft = A2;
const int trigPinRight = A5; // مستشعر يميني
const int echoPinRight = A4;

// تعريف المحركات
const int in1 = 7;
const int in2 = 6;
const int in3 = 9;
const int in4 = 8;
const int ENA = 10;
const int ENB = 11;

// تعريف السرعات والمسافات الآمنة
const int forwardSpeed = 100; // السرعة الأمامية
const int turnSpeed = 80;     // السرعة أثناء الانعطاف
const int safeDistanceFront = 20; // مسافة آمنة أمامية (20 سم)
const int safeDistanceSide = 10;  // مسافة آمنة جانبية (10 سم)

void setup() {
    pinMode(trigPinFront, OUTPUT);
    pinMode(echoPinFront, INPUT);
    pinMode(trigPinLeft, OUTPUT);
    pinMode(echoPinLeft, INPUT);
    pinMode(trigPinRight, OUTPUT);
    pinMode(echoPinRight, INPUT);

    pinMode(in1, OUTPUT);
    pinMode(in2, OUTPUT);
    pinMode(in3, OUTPUT);
    pinMode(in4, OUTPUT);
    pinMode(ENA, OUTPUT);
    pinMode(ENB, OUTPUT);

    Serial.begin(9600);
}

void loop() {
    long frontDistance = measureDistance(trigPinFront, echoPinFront);
    long leftDistance = measureDistance(trigPinLeft, echoPinLeft);
    long rightDistance = measureDistance(trigPinRight, echoPinRight);

    Serial.print("Front: ");
    Serial.print(frontDistance);
    Serial.print(" cm, Left: ");
    Serial.print(leftDistance);
    Serial.print(" cm, Right: ");
    Serial.println(rightDistance);

    if (frontDistance > safeDistanceFront) {
        if (leftDistance < safeDistanceSide) {
            // قريب جدًا من الجدار اليساري -> تعديل نحو اليمين
            adjustRight();
        } else if (rightDistance < safeDistanceSide) {
            // قريب جدًا من الجدار اليميني -> تعديل نحو اليسار
            adjustLeft();
        } else {
            // المسار مفتوح -> حركة للأمام
            forward(forwardSpeed);
        }
    } else {
        // عائق أمامي -> التوقف والانعطاف
        stopMotors();
        delay(500); // وقفة قصيرة قبل اتخاذ القرار
        if (leftDistance > rightDistance) {
            turnLeft();
        } else {
            turnRight();
        }
    }

    delay(100); // تأخير بسيط لاستقرار القراءة
}

// وظائف الحركة
void forward(int speed) {
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    digitalWrite(in3, HIGH);
    digitalWrite(in4, LOW);
    analogWrite(ENA, speed);
    analogWrite(ENB, speed);
}

void turnLeft() {
    digitalWrite(in1, LOW);
    digitalWrite(in2, HIGH);
    digitalWrite(in3, HIGH);
    digitalWrite(in4, LOW);
    analogWrite(ENA, turnSpeed);
    analogWrite(ENB, turnSpeed);
    delay(600); // وقت الانعطاف
    stopMotors();
}

void turnRight() {
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    digitalWrite(in3, LOW);
    digitalWrite(in4, HIGH);
    analogWrite(ENA, turnSpeed);
    analogWrite(ENB, turnSpeed);
    delay(600); // وقت الانعطاف
    stopMotors();
}

void adjustLeft() {
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    digitalWrite(in3, HIGH);
    digitalWrite(in4, LOW);
    analogWrite(ENA, turnSpeed);
    analogWrite(ENB, forwardSpeed); // تقليل سرعة المحرك اليميني للتوجه يسارًا
}

void adjustRight() {
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    digitalWrite(in3, HIGH);
    digitalWrite(in4, LOW);
    analogWrite(ENA, forwardSpeed);
    analogWrite(ENB, turnSpeed); // تقليل سرعة المحرك اليساري للتوجه يمينًا
}

void stopMotors() {
    digitalWrite(in1, LOW);
    digitalWrite(in2, LOW);
    digitalWrite(in3, LOW);
    digitalWrite(in4, LOW);
    analogWrite(ENA, 0);
    analogWrite(ENB, 0);
}

// وظيفة قياس المسافة
long measureDistance(int trigPin, int echoPin) {
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    long duration = pulseIn(echoPin, HIGH);
    long distance = duration * 0.034 / 2; // تحويل إلى السنتيمترات
    return distance;
}

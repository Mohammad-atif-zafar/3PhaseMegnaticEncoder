#define SELECT_PIN 4
#define CLOCK_PIN 5
#define DATA_PIN 6

void setup()
{
  // Setup our pins
  pinMode(DATA_PIN, INPUT);
  pinMode(CLOCK_PIN, OUTPUT);
  pinMode(SELECT_PIN, OUTPUT);

  // Give some default values
  digitalWrite(CLOCK_PIN, HIGH);
  digitalWrite(SELECT_PIN, HIGH);

  Serial.begin(19200);
}

// Variables to keep track of position
int reading = 0;
float angle = 0;

void loop()
{
   reading = readPosition();
   
   if (reading >= 0)
   {
      angle = ((float)reading / 1024.0) * 360.0;

      Serial.print("Reading: ");
      Serial.print(reading, DEC);
      Serial.print(" Angle: ");
      Serial.println((int)angle, DEC);
   }
   else
   {
      Serial.print("Error: ");
      Serial.println(reading);
   }
   
   delay(1000);
}

// Read the current angular position
int readPosition()
{
  unsigned int position = 0;

  // Shift in our data  
  digitalWrite(SELECT_PIN, LOW);
  delayMicroseconds(1);
  byte d1 = shiftIn(DATA_PIN, CLOCK_PIN);
  byte d2 = shiftIn(DATA_PIN, CLOCK_PIN);
  digitalWrite(SELECT_PIN, HIGH);

  // Get our position variable
  position = d1;
  position = position << 8;
  position |= d2;

  position = position >> 6;

  // Check the offset compensation flag: 1 == started up
  if (!(d2 & B00100000))
    position = -1;

  // Check the cordic overflow flag: 1 = error
  if (d2 & B00010000)
    position = -2;

  // Check the linearity alarm: 1 = error
  if (d2 & B00001000)
    position = -3;

  // Check the magnet range: 11 = error
  if ((d2 & B00000110) == B00000110)
    position = -4;

  return position;
}

// Read in a byte of data from the digital input of the board.
byte shiftIn(byte data_pin, byte clock_pin)
{
  byte data = 0;

  for (int i=7; i>=0; i--)
  {
    digitalWrite(clock_pin, LOW);
    delayMicroseconds(1);
    digitalWrite(clock_pin, HIGH);
    delayMicroseconds(1);

    byte bit = digitalRead(data_pin);
    data |= (bit << i);
  }

  Serial.print("byte: ");
  Serial.println(data, BIN);

  return data;
}

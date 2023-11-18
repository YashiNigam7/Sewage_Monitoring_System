# Sewage_Monitoring_System
#include <SoftwareSerial.h> // Library for using serial communication
SoftwareSerial SIM900(7, 8); // Pins 7, 8 are used as used as software serial pins
String incomingData; // for storing incoming serial data
String message = ""; // A String for storing the message
const int half_level_sensor=A2; // to check half filled sewer
const int full_level_sensor=A1; // to check fully filled sewer
int v=1; // for calibration
const int indicator =13; // to check putput
void setup()
{
 Serial.begin(9600); // baudrate for serial monitor
 SIM900.begin(9600); // baudrate for GSM shield
 pinMode(half_level_sensor,INPUT);
 pinMode(full_level_sensor,INPUT);
 // set SMS mode to text mode'
 SIM900.print("AT+CMGF=1\r"); 
 delay(100);
 
 // set gsm module to tp show the output on serial out
 SIM900.print("AT+CNMI=2,2,0,0,0\r"); 
 delay(100);
pinMode(indicator,OUTPUT);
digitalWrite(indicator,HIGH);
delay(1000);
digitalWrite(indicator,LOW);
//SIM900.println("AT+CMGD =4");
//delay(1000);
Serial.println("system is ready");
delay(500);
}
void loop()
{
 //Function for receiving sms
 receive_message();
 // if received command is to check tank
 if(incomingData.indexOf("Check")>=0)
 {
 int x1=analogRead(half_level_sensor);
int x2=analogRead(full_level_sensor);
Serial.println(x1);
Serial.println(x2);
if(x2>v && x1>v)
{
 Serial.println("water overflow");
 digitalWrite(indicator,HIGH);
 message= "water overflow";
}
if(x2<v && x1>v)
{
 Serial.println("water level is half filled");
 digitalWrite(indicator,LOW);
 message= "water level is half filled";
}
if(x2<v && x1<v)
{
 Serial.println("water level is OK");
 digitalWrite(indicator,LOW);
 message= "water level is low";
}
 // Send a sms back to confirm that the relay is turned on
 send_message(message);
 }
 // sleepGSM();
}
void receive_message()
{
 if (SIM900.available() > 0)
 {
 incomingData = SIM900.readString(); // Get the data from the serial port.
 Serial.print(incomingData); 
 delay(10); 
 }
}
void send_message(String message)
{
 SIM900.println("AT+CMGF=1"); //Set the GSM Module in Text Mode
 delay(500); 
 SIM900.println("AT+CMGR=1"); // Configuring TEXT mode
 delay(500);
 SIM900.println("AT+CMGS=\"+919968359003\""); // Replace it with your mobile number
 delay(500);
 SIM900.println(message); // The SMS text you want to send
 delay(500);
 SIM900.println((char)26); // ASCII code of CTRL+Z
 delay(100);
 SIM900.println();
 delay(2000); 
// SIM900.println("AT+CMGD =1,4");
// delay(1000);
}

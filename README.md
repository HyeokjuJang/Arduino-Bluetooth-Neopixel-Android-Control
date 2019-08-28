# Arduino-Bluetooth-Neopixel-Android-Control

## Upload code to arduino

upload bluetooth neopixel arduino control with android.ino to arduino(default is arduino mega 2560).  
If you use another arduino then you should change code to use bluetooth(ex. softwareserial).  
Anyway, change code. (ex. your custom number of neopixel)  

## On your android phone
Prepare android phone. not that old one.  
Download the app. [Link](https://play.google.com/store/apps/details?id=appinventor.ai_kms32123.royhelen_light_ver1&hl=ko)  
Turn on the bluetooth.  
Init the app.  
Connect to your bluetooth module.  
Have fun!  

## By the way
iPhone can not connect HC 06 module. remember!  

## Korean.Version  

# 💡 블루투스로 네오픽셀 제어
**필요한 것**

- 아두이노 나노
- 작은 브레드보드
- 네오픽셀
- 블루투스 모듈(HC-06)
- 전원(5V)
- 안드로이드 스마트폰

**회로도**

![다음과 같이 연결해주세요.](https://paper-attachments.dropbox.com/s_07906144E02CD80243C2791D64682BFD408FD4B2DD93968A6300FED1B96AA940_1566981964083_+2019-08-28++5.44.12.png)



- LED의 왼쪽은 VCC 중간은 DIN 오른쪽은 GND입니다.
- 블루투스는 HC-06입니다.
- 전원은 usb로 나노에 주면 됩니다.

**코드**

    #include <SoftwareSerial.h> //시리얼통신 라이브러리 호출
    #include <Adafruit_NeoPixel.h>
    #define PIN 6 // LED 제어선의 핀
    
    int blueTx=3;   //Tx (보내는핀 설정)
    int blueRx=2;   //Rx (받는핀 설정)
    SoftwareSerial mySerial(blueTx, blueRx);  //시리얼 통신을 위한 객체선언
    
    //네오픽셀을 사용하기 위해 객체 하나를 생성한다. 
    //첫번째 인자값은 네오픽셀의 LED의 개수
    Adafruit_NeoPixel strip = Adafruit_NeoPixel(20, PIN, NEO_GRB + NEO_KHZ800);
    
    String r = "255";
    String g = "255";
    String b = "255";
    int a = 0;
    
    void setup() {
      strip.begin(); //네오픽셀을 초기화하기 위해 모든LED를 off시킨다
      strip.show(); 
      Serial.begin(9600);   //시리얼모니터
      mySerial.begin(9600);
      /*
      주석을 해제하면 전원이 들어왔을 경우 아래 신호로 깜빡입니다.
      colorWipe(strip.Color(255, 0, 0), 50); //빨간색 출력
      colorWipe(strip.Color(0, 255, 0), 50); //녹색 출력
      colorWipe(strip.Color(0, 0, 255), 50); //파란색 출력
    
      theaterChase(strip.Color(127, 127, 127), 50); //흰색 출력
      theaterChase(strip.Color(127,   0,   0), 50); //빨간색 출력
      theaterChase(strip.Color(  0,   0, 127), 50); //파란색 출력
    
      //화려하게 다양한 색 출력
      rainbow(20);
      rainbowCycle(20);
      theaterChaseRainbow(50);
      */
    }
    
    void loop() {
      //아래의 순서대로 NeoPixel을 반복한다.
      if (mySerial.available()) {       
        if(a == 0){
          r = mySerial.read();
        }else if(a == 1){
          g = mySerial.read();
        }else if(a == 2){
          b = mySerial.read();
          a = -1;
        }
        a++;
        Serial.print(r+"\n"+g+"\n"+b);
        color(r.toInt(),g.toInt(),b.toInt());
        //colorWipe(strip.Color(r.toInt(),g.toInt(),b.toInt()), 50);
      }
      if (Serial.available()) {         
        mySerial.write(Serial.read());  //시리얼 모니터 내용을 블루추스 측에 WRITE
      }
    }
    
    void color(int r, int g, int b){
      for (int k=0; k < strip.numPixels(); k++) {
          strip.setPixelColor(k,strip.Color(r, g, b));
       }
       strip.show();
    }
    
    //NeoPixel에 달린 LED를 각각 주어진 인자값 색으로 채워나가는 함수
    void colorWipe(uint32_t c, uint8_t wait) {
      for(uint16_t i=0; i<strip.numPixels(); i++) {
          strip.setPixelColor(i, c);
          strip.show();
          delay(wait);
      }
    }
    
    //모든 LED를 출력가능한 모든색으로 한번씩 보여주는 동작을 한번하는 함수
    void rainbow(uint8_t wait) {
      uint16_t i, j;
    
      for(j=0; j<256; j++) {
        for(i=0; i<strip.numPixels(); i++) {
          strip.setPixelColor(i, Wheel((i+j) & 255));
        }
        strip.show();
        delay(wait);
      }
    }
    
    //NeoPixel에 달린 LED를 각각 다른색으로 시작하여 다양한색으로 5번 반복한다
    void rainbowCycle(uint8_t wait) {
      uint16_t i, j;
    
      for(j=0; j<256*5; j++) { 
        for(i=0; i< strip.numPixels(); i++) {
          strip.setPixelColor(i, Wheel(((i * 256 / strip.numPixels()) + j) & 255));
        }
        strip.show();
        delay(wait);
      }
    }
    
    //입력한 색으로 LED를 깜빡거리며 표현한다
    void theaterChase(uint32_t c, uint8_t wait) {
      for (int j=0; j<10; j++) {  //do 10 cycles of chasing
        for (int q=0; q < 3; q++) {
          for (int i=0; i < strip.numPixels(); i=i+3) {
            strip.setPixelColor(i+q, c);    //turn every third pixel on
          }
          strip.show();
         
          delay(wait);
         
          for (int i=0; i < strip.numPixels(); i=i+3) {
            strip.setPixelColor(i+q, 0);        //turn every third pixel off
          }
        }
      }
    }
    
    //LED를 다양한색으로 표현하며 깜빡거린다
    void theaterChaseRainbow(uint8_t wait) {
      for (int j=0; j < 256; j++) {     //256가지의 색을 표현
        for (int q=0; q < 3; q++) {
            for (int i=0; i < strip.numPixels(); i=i+3) {
              strip.setPixelColor(i+q, Wheel( (i+j) % 255));
            }
            strip.show();
           
            delay(wait);
           
            for (int i=0; i < strip.numPixels(); i=i+3) {
              strip.setPixelColor(i+q, 0); 
            }
        }
      }
    }
    
    //255가지의 색을 나타내는 함수
    uint32_t Wheel(byte WheelPos) {
      if(WheelPos < 85) {
       return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
      } else if(WheelPos < 170) {
       WheelPos -= 85;
       return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
      } else {
       WheelPos -= 170;
       return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
      }
    }

아두이노 라이브러리 매니저에서 SoftwareSerial과 Adafruit_NeoPixel를 설치해야합니다.

**업로드**
툴에서 usb포트와 보드를 nano로 선택하고 업로드합니다.

**안드로이드**

1. 안드로이드 스마트폰에서 [링크](https://play.google.com/store/apps/details?id=appinventor.ai_kms32123.royhelen_light_ver1&hl=ko)에 있는 앱을 받습니다.
2. 앱을 열고 하단의 블루투스 연결하기에서 HC-06을 선택합니다.
3. 연결이 된 후 중앙을 터치하면 on/off가 가능하고 중앙을 터치하고 있으면 색을 바꿀 수 있는 창이 나타납니다.
4. 터치 후 드래그로 색을 정밀하게 선택 후 터치를 떼면 색이 선택됩니다.


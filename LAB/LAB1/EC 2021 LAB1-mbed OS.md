## EC 2021 LAB1



    // [Include] //
    #include "mbed.h"
    // [Define] //
    #define     RESTART     1
    #define     PAUSE       0
    #define     OFF         0
    #define     MID         1
    #define     HIGH        2
    #define    VHIGH       3
    int    VELOFF     = 0;
    int    VELMID     = 5;
    int     VELHIGH    = 7;
    int     VELVHIGH   = 10;
    int     delv       = 1;
    Ticker      LED_tick;
    DigitalOut  led(LED1);
    PwmOut      pwm1(D11);
    InterruptIn button(USER_BUTTON);
    Serial      pc(USBTX, USBRX, 9600);
    PwmOut      trig(D10); // Trigger 핀
    InterruptIn echo(D7);  // Echo 핀
    Timer       tim;
    Serial      uart(USBTX, USBRX, 9600);
    Ticker      Printtick;
    // [Variables] //
    int     v           = 0;
    int     toggle      = 1;
    int     led_flag    = 1;
    int     curvel      =0;
    int     US_begin    = 0;
    int     US_end      = 0;
    float   Dist        = 0.0;
    bool printflag   =1;
    // [Functions] //
    
    void US_Rising(){
        US_begin = tim.read_us();
    }
    void US_Falling(){
        US_end = tim.read_us();
    }
    void ButtonPress(){
        v += delv;
        if (v > VHIGH)  
        {v = 0;
        }
        else
        {    v = v;
        }
    }
    void LED(){
        if (led_flag == 1) led = !led;
        else               led = 0;
    }
    void LED_Blink(){
        LED_tick.attach(&LED, 1);
    }
    void PrintFlagSet(){
        printflag = 1;
    }
    
    void Init(){    
        trig.period_ms(60);     // period      = 60ms
        trig.pulsewidth_us(10); // pulse-width = 10us
        echo.rise(&US_Rising);
        echo.fall(&US_Falling);
        tim.start();
        pwm1.period_ms(10);
        LED_Blink();
        button.fall(&ButtonPress);
        Printtick.attach(&PrintFlagSet,2);
    }
    
    void DistanceMeasure(){
        Dist =  (float)(US_end - US_begin) / 58; // [cm]
        
    
        if (Dist > 5) toggle = 0;
        else          toggle = 1;
    
    }
    
    void MiniFanON(){
        led_flag=1;
        switch(v){
            case(OFF)  : pwm1.pulsewidth_ms(VELOFF);curvel=0;led_flag=0; break;
            case(MID)  : pwm1.pulsewidth_ms(VELMID);curvel=5; break;
            case(HIGH) : pwm1.pulsewidth_ms(VELHIGH);curvel=7; break;
            case(VHIGH): pwm1.pulsewidth_ms(VELVHIGH);curvel=10; break;
        }
    }
    void print()
    {
       pc.printf("Distance : %.2f \r\n",Dist);
         pc.printf("Duty ratio : %d\r\n",curvel);
       printflag=0;
            }   
    void MiniFanOFF() {
        pwm1.pulsewidth_ms(VELOFF);
        curvel=0;
    }
    // [Main] //
    int main() {
        Init();
        while(1){
            DistanceMeasure();
            
    
            switch(toggle){
                
                case(RESTART): 
                    MiniFanON();
                    break;
                
                case(PAUSE)  :
                    MiniFanOFF();
                    break;
            }
            if(printflag)
            {
               print();
               
                }
            
        }     
    
    }




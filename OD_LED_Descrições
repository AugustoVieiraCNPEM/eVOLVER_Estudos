// Recurring Command: 'od_90r,1000,_!'
// Immediate Command: 'od_90i,1000,_!'
// Acknowledgement to Run: 'od_90a,1000,_!'


// Recurring Command: 'od_ledr,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,_!"
// Immediate Command: 'od_ledi,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,_!"
// Acknowledgement to Run: "od_leda,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,_!"

#include <evolver_si.h>      //10-16) inclui bibliotecas > Define variáveis para ver se há informação serial, se a string está completa e para segurar dados inseridos.
#include <Tlc5940.h>

// String Input
String inputString = "";         // a string to hold incoming data
boolean stringComplete = false;  // whether the string is complete
boolean serialAvailable = true;  // if serial port is ok to write on

// Mux Shield Components and Control Pins     // 18-23) Define valores pde vials, entrada e saída de dados, etc
int s0 = 7, s1 = 8, s2 = 9, s3 = 10, SIG_pin = A0;
int num_vials = 16;
int mux_readings[16]; // The size Assumes number of vials
int active_vial = 0;
int PDtimes_averaged = 1000;           // 24) define array output com valores 6000
int output[] = {60000,60000,60000,60000,60000,60000,60000,60000,60000,60000,60000,60000,60000,60000,60000,60000};
int Input[] = {4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095};
                                        // 25) define array output com valores 4095
//General Serial Communication
String comma = ",";             // Define string 'comma', para indicar separação
String end_mark = "end";        // define String 'endmark', para indicar a hora de para um processo.

// Photodiode Serial Communication
int expected_PDinputs = 2;              //define o número de inputs esperados
String photodiode_address = "od_90";    //define o endereço do photodiodo (ainda não sei o que isso é. Talvez uma sinalização? Se este endereço estiver presente, o sistema entende que deve-se tratar do fotodiodo?)
evolver_si in("od_90", "_!", expected_PDinputs); //2 CSV Inputs from RPI
boolean new_PDinput = false;   //define como falsa a informação inicial se há input do fotodiodo
int saved_PDaveraged = 1000; // saved input from Serial Comm.


// LED Settings
String led_address = "od_led"; //define o endereeço do LED
evolver_si led("od_led", "_!", num_vials+1); // 17 CSV-inputs from RPI
boolean new_LEDinput = false;
int saved_LEDinputs[] = {4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095,4095}; // valores iniciais dos LEDs. Em 4095, a intensidae é mínima, e em 0, é máxima.



void setup() {
  pinMode(12, OUTPUT);
  digitalWrite(12, LOW);


  // Set up Mux
  pinMode(s0, OUTPUT);
  pinMode(s1, OUTPUT);
  pinMode(s2, OUTPUT);
  pinMode(s3, OUTPUT);
  pinMode(SIG_pin, INPUT);

  digitalWrite(s0, LOW);
  digitalWrite(s1, LOW);
  digitalWrite(s2, LOW);
  digitalWrite(s3, LOW);

  analogReadResolution(16);
  Tlc.init(LEFT_PWM,4095);
  Serial1.begin(9600);
  SerialUSB.begin(9600);
  // reserve 1000 bytes for the inputString:
  inputString.reserve(1000);
  while (!Serial1);

  for (int i = 0; i < num_vials; i++) {
    Tlc.set(LEFT_PWM, i, 4095 - Input[i]);
  }
  while(Tlc.update());

}

void loop() {
  SerialUSB.print("Reading Vial:");
  SerialUSB.println(active_vial);
  read_MuxShield();   //Aciona a subrrotina para ler os valores do MUX de Input
  if (stringComplete) {  //caso a string esteja completa:
    SerialUSB.println(inputString);    //Escreve o input no monitor serial
    in.analyzeAndCheck(inputString);
    led.analyzeAndCheck(inputString);

    // Clear input string, avoid accumulation of previous messages
    inputString = "";
    
    // Photodiode Logic
    if (in.addressFound) {    //caso encontre o endereço do fotodiodo
      if (in.input_array[0] == "i" || in.input_array[0] == "r") { //se o comando for do tipo recorrente ou imediato
        
        SerialUSB.println("Saving PD Setting");  //escreve o que está entre aspas
        saved_PDaveraged = in.input_array[1].toInt();  //salva o valor lido pelo fotodiodo como número inteiro 
        
        SerialUSB.println("Echoing New PD Command");
        new_PDinput = true;  //indica que houve novo input do fotodiodo
        dataResponse();  //invoca a subrrotina dataResponse

        SerialUSB.println("Waiting for OK to execute...");
      }
      if (in.input_array[0] == "a" && new_PDinput) { //se recebe um reconhecimento para executar ('a') e há novo input
        PDtimes_averaged = saved_PDaveraged;    //salva os inputs em nova array
        SerialUSB.println("PD Command Executed!");  //escreve que o comando foi executado com sucesso
        new_PDinput = false;   //reseta o booleano para indicar quye não há novo input
      }        
      
      in.addressFound = false;
      inputString = "";
    }
    
    // LED Logic
    if (led.addressFound) {    /se encontra endereço do LED
      if (led.input_array[0] == "i" || led.input_array[0] == "r") {  //se o input for do tipo imediato (i)ou recorrente (r)
        SerialUSB.println("Saving LED Setpoints");
        for (int n = 1; n < num_vials+1; n++) {   //realiza uma ação para cada um dos vials
          saved_LEDinputs[n-1] = led.input_array[n].toInt();  //converte os inputs dos LEDs e os salva em uma lista
        }
        
        SerialUSB.println("Echoing New LED Command");
        new_LEDinput = true;  //indica que houve input
        echoLED();   //invoca a subrrotina echoLED
        
        SerialUSB.println("Waiting for OK to execute...");
      }
      if (led.input_array[0] == "a" && new_LEDinput) {  //se o comando for do tipo acknowledgement to run (a)
        update_LEDvalues(); // invoca a subrrotina update_LEDvalues
        SerialUSB.println("Command Executed!"); //escreve que o comando foi executado
        new_LEDinput = false;       //retorna o estado do input para o LED como falso.
        
      }


      led.addressFound = false;
      inputString = "";
    }

    // Clears strings if too long
    // Should be checked server-side to avoid malfunctioning
    if (inputString.length() > 2000){
      SerialUSB.println("Cleared Input String");
      inputString = "";
    }
  }

  // clear the string:
  stringComplete = false;
}

void serialEvent() {
  while (Serial1.available()) {   //enquanto há input vindo do RPi
    char inChar = (char)Serial1.read();   //cria caractere correspondente à leitura do RPi
    inputString += inChar;    // adiciona o caractere ao inputString
    if (inChar == '!') {       // se o input for '!'
      stringComplete = true;   // indica que a string foi completa
      break;                   // interrompe a subrrotina
    }
  }
}

void echoLED() {
  digitalWrite(12, HIGH);
  
  String outputString = led_address + "e,";  //adiciona 'e' ao led_address (od_lede)
  for (int n = 1; n < num_vials+1 ; n++) {   //para cada um dos vials:
    outputString += led.input_array[n];      //adiciona o valor da n-ésima posição ao outputString
    outputString += comma;     //separa cada elemento por uma vírgula
  }
  outputString += end_mark;    //adiciona o endmark ao final para indicar o fim 
  delay(100);
  if (serialAvailable) {  //caso haja dados vindo do serial
    SerialUSB.println(outputString);   // escreve o outputString no monitor Serial
    Serial1.print(outputString);       //escreve o outputString no RPi
  }  
  delay(100);
  digitalWrite(12, LOW);
}

void update_LEDvalues() {
  for (int i = 0; i < num_vials; i++) {  //para cada um dos vials:
    Tlc.set(LEFT_PWM, i, 4095 - saved_LEDinputs[i]);  //define os novos valores dos leds de acordo com o input do usuário 
  }
  while(Tlc.update());
}


int dataResponse (){
  digitalWrite(12, HIGH);
  String outputString = photodiode_address + "b,"; // b is a broadcast tag
  for (int n = 0; n < num_vials; n++) {   //para cada um dos vials:
    outputString += output[n];            // adiciona o n-ésimo elemento do output ao outputString
    outputString += comma;                //separa os elementos por vírgulas
  }
  outputString += end_mark;               //adiciona o sinalizador de fim no final da outputString

  delay(100); // important to make sure pin 12 flips appropriately
  
  SerialUSB.println(outputString);        //escreve o OutputString no monitor serial
  Serial1.print(outputString); // issues w/ println on Serial 1 being read into Raspberry Pi

  delay(100); // important to make sure pin 12 flips appropriately

  digitalWrite(12, LOW);
}

void read_MuxShield() {    //subrrotina para a leitura dos canais do MUX
  unsigned long mux_total=0;
  
  for (int h=0; h<(PDtimes_averaged); h++){
    mux_total = mux_total + readMux(active_vial);
    serialEvent();
    if (stringComplete){
      SerialUSB.println("String Completed, stop averaging");
      SerialUSB.println(h);
      break;
    }
  }
  if (!stringComplete){
    output[active_vial] = mux_total / PDtimes_averaged;
    SerialUSB.println(output[active_vial]);
    if (active_vial == 15){
      active_vial = 0;
    } else {
      active_vial++;
    }
  }
}

int readMux(int channel){
  int controlPin[] = {s0, s1, s2, s3};

  int muxChannel[16][4]={
    {0, 0, 0, 0}, //channel 0; Vial 0
    {1, 1, 0, 0}, //channel 3; Vial 1
    {1, 0, 0, 0}, //channel 1; Vial 2
    {0, 1, 0, 0}, //channel 2; Vial 3
    {0, 0, 1, 0}, //channel 4; Vial 4
    {1, 1, 1, 0}, //channel 7; Vial 5
    {1, 0, 1, 0}, //channel 5; Vial 6
    {0, 1, 1, 0}, //channel 6; Vial 7
    {0, 0, 0, 1}, //channel 8; Vial 8
    {1, 1, 0, 1}, //channel 11; Vial 9
    {1, 0, 0, 1}, //channel 9; Vial 10
    {0, 1, 0, 1}, //channel 10; Vial 11
    {0, 0, 1, 1}, //channel 12; Vial 12
    {1, 1, 1, 1}, //channel 15; Vial 13
    {1, 0, 1, 1}, //channel 13; Vial 14
    {0, 1, 1, 1}, //channel 14; Vial 15
  };

  //loop through the 4 sig
  for(int i = 0; i < 4; i ++){
    digitalWrite(controlPin[i], muxChannel[channel][i]);
  }

  //read the value at the SIG pin
  int val = analogRead(SIG_pin);

  //return the value
  return val;
}

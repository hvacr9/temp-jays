temp-jays
=========

jays cooler temp
/*
 Temperature Web Server
 
 A simple web server that shows the current temperatures using an Arduino 
 Wiznet Ethernet shield and TMP36 Sensor. 
 
 Circuit:
 * Ethernet shield attached to pins 10, 11, 12, 13
 * TMP36 attached to pin A0 
 */
 
#include <SPI.h>
#include <Ethernet.h>
 
// Enter a MAC address and IP address for your controller below.
// The IP address will be dependent on your local network:
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
byte ip[] = { 192,168,1,200 };
 
// TMP36 Pin Variables
int temperaturePin = 0; //the analog pin the TMP36's Vout (sense) pin is connected to
                        //the resolution is 10 mV / degree centigrade 
                        //(500 mV offset) to make negative temperatures an option
 
// Initialize the Ethernet server library
// with the IP address and port you want to use 
// (port 80 is default for HTTP):
Server server(80);
 
void setup()
{
  // start the Ethernet connection and the server:
  Ethernet.begin(mac, ip);
  server.begin();
}
 
//  url buffer size
#define BUFSIZE 255
 
void loop()
{
  char clientline[BUFSIZE];
  int index = 0;
  // listen for incoming clients
  Client client = server.available();
  if (client) {
    
    //  reset input buffer
    index = 0;
 
    // an http request ends with a blank line
    boolean currentLineIsBlank = true;
    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        
        //  fill url the buffer
        if(c != '\n' && c != '\r'){
          clientline[index] = c;
          index++;
 
          //  if we run out of buffer, overwrite the end
          if(index >= BUFSIZE)
            index = BUFSIZE -1;
 
          continue;
        }
        //  convert clientline into a proper
        //  string for further processing
        String urlString = String(clientline);
 
        //  extract the operation
        String op = urlString.substring(0,urlString.indexOf(' '));
 
        //  we're only interested in the first part...
        urlString = urlString.substring(urlString.indexOf('/'), urlString.indexOf(' ', urlString.indexOf('/')));
 
        //  put what's left of the URL back in client line
        urlString.toCharArray(clientline, BUFSIZE);
 
        //  get the first two parameters
        char *pin = strtok(clientline,"/");
        char *value = strtok(NULL,"/");
 
 
        // if you've gotten to the end of the line (received a newline
        // character) and the line is blank, the http request has ended,
        // so you can send a reply
        //if (c == '\n' && currentLineIsBlank) {
          // send a standard http response header
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/html");
          client.println();
          if( urlString.substring(1,5) == "temp" )
          {
            client.print("Current Temperature in fahrenheit: ");
            client.print(getTemperature());
            client.println("<br />");
          }
          else if ( urlString.substring(1,6) == "reset" )
          {
            client.print("Resetting <br />"); 
            String resetStringNumber = urlString.substring(urlString.indexOf('?')+1, urlString.indexOf('?')+5);
            char this_char[resetStringNumber.length() + 1];
            resetStringNumber.toCharArray(this_char, sizeof(this_char));
            int resetInteger = atoi(this_char);
            client.print(resetInteger);
          }
          else
          {
            client.print("Command Not Found <br />");
          }
          break;
      }
    }
    // give the web browser time to receive the data
    delay(1);
    // close the connection:
    client.stop();
  }
}
 
float getTemperature(){
 float temperature = getVoltage(temperaturePin);  //getting the voltage reading from the temperature sensor
 temperature = (((temperature - .5) * 100)*1.8) + 32;
 return (temperature);
}
 
// getVoltage() - returns the voltage on the analog input defined by pin
float getVoltage(int pin){
 return (analogRead(pin) * .004882814); // converting from a 0 to 1024 digital range
                                        // to 0 to 5 volts (each 1 reading equals ~ 5 millivolts

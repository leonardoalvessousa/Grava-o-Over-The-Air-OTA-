![Banner](https://raw.githubusercontent.com/leonardoalvessousa/Grava-o-Over-The-Air-OTA-/refs/heads/main/bannerOTA.jpg)

O processo de upload Over-the-Air (OTA) permite atualizar o firmware de um dispositivo sem a necessidade de conexão física. Para que o upload OTA funcione, a memória do dispositivo precisa ser organizada de forma a permitir a atualização do firmware sem corromper o código em execução. Neste artigo iremos explorar esse recurso voltado para os chips: Esp32 e Esp8266.

### Diagrama do processo de upload

> Etapa 1: O New Sketch será gravado na área de memória flash disponível entre o Current Sketch e a partição SPIFF.

`[Current Sketch] <------> [New Sketch]  <------> [SPIFF]`

 > [!NOTE]
> SPIFF (**S**ingle **P**age **I**n **F**ile **F**ile **S**ystem) é um sistema de arquivos de página única, projetado para ser usado em dispositivos com memória flash limitada, como microcontroladores.

> Etapa 2: Após a próxima reinicialização, o bootloader "eboot" processa os comandos recebidos. Em seguida, o novo sketch é gravado sobre o antigo e o dispositivo é iniciado com o novo firmware.

`[New Sketch]  <----------------------> [SPIFF]` 

### Code Pré-uploads

Antes de fazer o upload de novos sketches para o seu dispositivo com a funcionalidade OTA (Over-the-Air), é fundamental que você configure o sketch principal com as funcionalidades OTA propriamente ditas.

```IDE_Arduino
// C++

/* Inclusão de biblioteca e constantes */
#ifdef ESP8266
#include <ESP8266WiFi.h>
#include <ESP8266mDNS.h>
#else
#include <WiFi.h>
#include <ESPmDNS.h>
#endif

#include <WiFiUdp.h>
#include <ArduinoOTA.h>

const char* ssid = "..........";
const char* password = "..........";

/* Configurações iniciais */
void setup() 
{
	Serial.begin(115200);
	Serial.println("Booting");
	WiFi.mode(WIFI_STA);
	WiFi.begin(ssid, password);
	while (WiFi.waitForConnectResult() != WL_CONNECTED) 
	{
	    Serial.println("Connection Failed! Rebooting...");
	    delay(5000);
	    ESP.restart();
	}

 /* Funções de Callback */

  ArduinoOTA.onStart([]() 
 {
    String type;
    if (ArduinoOTA.getCommand() == U_FLASH) 
    {
      type = "sketch";
    } else 
    { 
      type = "filesystem";
    }
    Serial.println("Start updating " + type);
 });

  ArduinoOTA.onEnd([]() 
 {
    Serial.println("\nEnd");
 });

  ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) 
 {
    Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
 });

  ArduinoOTA.onError([](ota_error_t error) 
 {
    Serial.printf("Error[%u]: ", error);
    if (error == OTA_AUTH_ERROR) {
      Serial.println("Auth Failed");
    } else if (error == OTA_BEGIN_ERROR) {
      Serial.println("Begin Failed");
    } else if (error == OTA_CONNECT_ERROR) {
      Serial.println("Connect Failed");
    } else if (error == OTA_RECEIVE_ERROR) {
      Serial.println("Receive Failed");
    } else if (error == OTA_END_ERROR) {
      Serial.println("End Failed");
    }
 });
  ArduinoOTA.begin();
  Serial.println("Ready");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

/* Tarefa principal */
void loop() 
{
  ArduinoOTA.handle();
}
```


#### Descodificando: Inclusão de bibliotecas e constantes🚀

> Inclusão de bibliotecas

```cpp
#ifdef ESP8266 
#include <ESP8266WiFi.h> 
#include <ESP8266mDNS.h> 
#else 
#include <WiFi.h> 
#include <ESPmDNS.h> #endif
```


* **`#ifdef ESP8266`:** Esta diretiva verifica se a constante `ESP8266` está definida. Essa constante é geralmente definida pelo compilador quando você está compilando o código para um dispositivo ESP8266. 
* **`#include <ESP8266WiFi.h>`:** Se a constante `ESP8266` estiver definida, essa linha inclui a biblioteca `ESP8266WiFi.h`, que fornece as funções necessárias para conectar o dispositivo ESP8266 a uma rede Wi-Fi. 
* **`#include <ESP8266mDNS.h>`:** Se a constante `ESP8266` estiver definida, essa linha inclui a biblioteca `ESP8266mDNS.h`, que fornece as funções necessárias para usar o mDNS (Multicast DNS) no ESP8266. 
* **`#else`:** Esta diretiva indica que o código a seguir será executado caso a constante `ESP8266` **não** esteja definida. 
* **`#include <WiFi.h>`:** Se a constante `ESP8266` não estiver definida, essa linha inclui a biblioteca `WiFi.h`, que fornece as funções necessárias para conectar o dispositivo a uma rede Wi-Fi. * 
* **`#include <ESPmDNS.h>`:** Se a constante `ESP8266` não estiver definida, essa linha inclui a biblioteca `ESPmDNS.h`, que fornece as funções necessárias para usar o mDNS (Multicast DNS).

> Constantes

```cpp
const char* ssid = ".........."; 
const char* password = "..........";
```

#### Descodificando: Configurações iniciais🚀

```cpp
void setup() 
{
	Serial.begin(115200);
	Serial.println("Booting");
	WiFi.mode(WIFI_STA);
	WiFi.begin(ssid, password);
	while (WiFi.waitForConnectResult() != WL_CONNECTED) 
	{
	    Serial.println("Connection Failed! Rebooting...");
	    delay(5000);
	    ESP.restart();
	}   
//Continua...
```

- **`void setup()`:** Esta função é executada uma vez no início do programa, antes do loop principal (`loop()`). Ela configura as configurações iniciais do dispositivo.

- **`Serial.begin(115200);`:** Inicializa a comunicação serial com a taxa de baud de 115200 bps. Isso permite que o dispositivo se comunique com um computador ou outro dispositivo serial, enviando e recebendo dados.
- **`Serial.println("Booting");`:** Imprime a mensagem "Booting" na serial para indicar que o dispositivo está iniciando. Isso ajuda a monitorar o processo de inicialização.
- **`WiFi.mode(WIFI_STA);`:** Define o modo de operação da interface Wi-Fi como estação (cliente). Isso significa que o dispositivo irá se conectar a uma rede Wi-Fi existente, em vez de criar sua própria rede.
- **`WiFi.begin(ssid, password);`:** Inicia a conexão com a rede Wi-Fi usando o SSID e a senha definidos anteriormente nas constantes `ssid` e `password`. O dispositivo tenta se conectar à rede Wi-Fi especificada.
- **`while (WiFi.waitForConnectResult() != WL_CONNECTED) { ... }`:** Este loop aguarda até que o dispositivo seja conectado à rede Wi-Fi. A função `WiFi.waitForConnectResult()` retorna o estado da conexão Wi-Fi:
    - **`WL_CONNECTED`:** Indica que o dispositivo está conectado à rede Wi-Fi.
    - **Outros valores:** Indicam que a conexão falhou.
- **`Serial.println("Connection Failed! Rebooting...");`:** Se a conexão com a rede Wi-Fi falhar, essa linha imprime uma mensagem de erro na serial.
- **`delay(5000);`:** Espera 5 segundos antes de reiniciar o dispositivo. Isso dá tempo para visualizar a mensagem de erro na serial.
- **`ESP.restart();`:** Reinicia o dispositivo ESP. Isso permite que o dispositivo tente se conectar à rede Wi-Fi novamente.

#### Descodificando: Funções de Callback 🚀

Uma função de `callback` é uma função fornecida como argumento a outra função. Essas funções, que são chamadas quando a eventos específicos, ocorrem durante o processo de atualização de firmware.

```cpp
//Continua...
 /* Funções de Callback */

  ArduinoOTA.onStart([]() 
 {
    String type;
    if (ArduinoOTA.getCommand() == U_FLASH) 
    {
      type = "sketch";
    } else 
    { 
      type = "filesystem";
    }
    Serial.println("Start updating " + type);
 });

  ArduinoOTA.onEnd([]() 
 {
    Serial.println("\nEnd");
 });

  ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) 
 {
    Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
 });

  ArduinoOTA.onError([](ota_error_t error) 
 {
    Serial.printf("Error[%u]: ", error);
    if (error == OTA_AUTH_ERROR) {
      Serial.println("Auth Failed");
    } else if (error == OTA_BEGIN_ERROR) {
      Serial.println("Begin Failed");
    } else if (error == OTA_CONNECT_ERROR) {
      Serial.println("Connect Failed");
    } else if (error == OTA_RECEIVE_ERROR) {
      Serial.println("Receive Failed");
    } else if (error == OTA_END_ERROR) {
      Serial.println("End Failed");
    }
 });
  ArduinoOTA.begin();
  Serial.println("Ready");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}
```

- **`ArduinoOTA.onStart([]() { ... });`:** Define um callback que é chamado quando o processo de atualização começa. A função anônima dentro do callback define o que acontece quando a atualização é iniciada.
- **`ArduinoOTA.onEnd([]() { ... });`:** Define um callback que é chamado quando o processo de atualização termina. A função anônima dentro do callback define o que acontece quando a atualização termina.
- **`ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) { ... });`:** Define um callback que é chamado durante o processo de atualização para mostrar o progresso. A função anônima dentro do callback recebe dois parâmetros: `progress` e `total`, que representam o progresso atual e o tamanho total da atualização, respectivamente.
- **`ArduinoOTA.onError([](ota_error_t error) { ... });`:** Define um callback que é chamado caso ocorra algum erro durante o processo de atualização. A função anônima dentro do callback recebe um parâmetro `error`, que representa o tipo de erro ocorrido.
- **`ArduinoOTA.begin();`:** Inicia o servidor ArduinoOTA para receber atualizações de firmware.
- **`Serial.println("Ready");`:** Imprime uma mensagem na serial informando que o dispositivo está pronto para receber atualizações.
- **`Serial.print("IP address: ");`:** Imprime uma mensagem na serial informando que o endereço IP do dispositivo será mostrado a seguir.
- **`Serial.println(WiFi.localIP());`:** Imprime o endereço IP do dispositivo na serial.

#### Descodificando: Tarefa principal 🚀

```cpp
void loop() 
{
  ArduinoOTA.handle();
}
```

- **`void loop()`:** Esta função é executada repetidamente após a função `setup()` ser concluída. Ela contém o código principal do programa que é executado em loop.
- **`ArduinoOTA.handle();`:** Esta linha de código é a chave para o funcionamento do ArduinoOTA. Ela processa todas as solicitações de atualização de firmware que podem estar pendentes.

 > [!NOTE]
> Sketch carregado, agora você vai notar que ao visitar a aba `Port` na IDE Arduino estará um endereço IP. Este corresponde ao seu ESP que aguarda para receber novos uploads!
### Code upload 

Note que o sketch não muda muito. Isso se deve ao fato de que desejamos manter a funcionalidade OTA sempre disponível, não é mesmo?!

> LED de prova: Tente fazer uploads de algoritmos simples via OTA. E com o tempo, avance para algoritmos mais complexos. 

```cpp
#ifdef ESP8266
#include <ESP8266WiFi.h>
#include <ESP8266mDNS.h>
#else
#include <WiFi.h>
#include <ESPmDNS.h>
#endif

#include <WiFiUdp.h>
#include <ArduinoOTA.h>

#define LED_PROVA 2

const char* ssid = ".........."; 
const char* password = "..........";

void setup() 
{
  pinMode(LED_PROVA, OUTPUT);
  Serial.begin(115200);
  Serial.println("Booting");
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.waitForConnectResult() != WL_CONNECTED) {
    Serial.println("Connection Failed! Rebooting...");
    delay(5000);
    ESP.restart();
  }

  ArduinoOTA.onStart([]() 
  {
    String type;
    if (ArduinoOTA.getCommand() == U_FLASH) 
    {
      type = "sketch";
    } else 
    {
      type = "filesystem";
    }
    Serial.println("Start updating " + type);
  });
  ArduinoOTA.onEnd([]() {
    Serial.println("\nEnd");
  });
  ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) 
  {
    Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
  });
  ArduinoOTA.onError([](ota_error_t error) 
  {
    Serial.printf("Error[%u]: ", error);
    if (error == OTA_AUTH_ERROR) 
    {
      Serial.println("Auth Failed");
    } else if (error == OTA_BEGIN_ERROR) 
    {
      Serial.println("Begin Failed");
    } else if (error == OTA_CONNECT_ERROR) 
    {
      Serial.println("Connect Failed");
    } else if (error == OTA_RECEIVE_ERROR) 
    {
      Serial.println("Receive Failed");
    } else if (error == OTA_END_ERROR) 
    {
      Serial.println("End Failed");
    }
  });
  ArduinoOTA.begin();
  Serial.println("Ready");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() 
{
  digitalWrite(LED_PROVA, HIGH);
  delay(500);
  digitalWrite(LED_PROVA, LOW);
  delay(500);
  ArduinoOTA.handle(); //Não esqueça de mim!
}
```

> [!CAUTION]
> Não esqueça de incluir a função `ArduinoOTA.handle()` no fim de seus projetos. 

## ## 😼 Autor

🐈‍⬛ @leonardoalvessousa

## 🎁 Expressões de gratidão

- Conte a outras pessoas sobre este projeto 📢;
- Pague uma cerva para o autor **[🍺](https://docs.arduino.cc/software/ide-v1/tutorials/Linux/)**;


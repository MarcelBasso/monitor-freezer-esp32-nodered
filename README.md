# Monitor de Freezer com ESP32, MQTT e Node-RED

## Visão Geral do Projeto

Este projeto implementa um sistema de monitorização do estado de energia de um freezer utilizando um microcontrolador ESP32. O ESP32 deteta se o freezer está ligado ou desligado através de um sensor de toque (simulando um sensor de presença de energia) e publica essas informações via protocolo MQTT para um broker. O Node-RED subscreve a esses tópicos MQTT para exibir um dashboard com o estado atual e histórico do freezer, além de enviar notificações por e-mail em caso de o freezer ser desligado ou perder a comunicação por um período prolongado (configurado para 1 hora).

Este repositório contém:
* O código Arduino (`.ino` ou `.cpp`) para o ESP32.
* O fluxo de configuração (`.json`) para o Node-RED.

## Funcionalidades

* **Monitorização em Tempo Real:** O ESP32 envia o estado (Ligado/Desligado) do freezer para o broker MQTT.
* **Lógica de Sensor Invertida:** Configurado para que "sem toque" no sensor signifique freezer LIGADO e "com toque" signifique freezer DESLIGADO.
* **Dashboard no Node-RED:** Visualização do estado atual e histórico do freezer.
* **Notificações por E-mail:**
    * Envia um e-mail quando o estado do freezer muda para DESLIGADO.
    * Envia um e-mail se nenhuma comunicação for recebida do ESP32 por mais de 1 hora, indicando uma possível falha ou desconexão.
* **Sinalização Visual no ESP32:** Um LED na placa ESP32 pisca para indicar que está conectado ao broker MQTT.
* **Last Will and Testament (LWT):** O ESP32 está configurado para que o broker MQTT publique uma mensagem de status "-1" (desconectado) caso a conexão do ESP32 caia inesperadamente.

## Componentes Utilizados

### Hardware
* ESP32 (qualquer variante com pino de toque, por exemplo, ESP32 DevKitC)
* Sensor de toque (no projeto atual, o pino TOUCH0/GPIO4 do ESP32 é usado para simular isso)
* Conexão Wi-Fi
* (Opcional) Fonte de alimentação para o ESP32

### Software
* Arduino IDE ou PlatformIO para programar o ESP32
* Node-RED (instalado localmente ou em um servidor)
    * Nó `node-red-dashboard` (para o painel de visualização)
    * Nó `node-red-node-email` (para notificações por e-mail)
    * Broker MQTT (pode ser o Aedes integrado ao Node-RED ou um broker externo como Mosquitto)
* Conta de e-mail para enviar notificações (ex: Gmail com "Senha de App")

## Estrutura do Repositório

* `esp32_freezer_monitor.ino` (ou `esp32_freezer_txt.txt`): Código fonte para o microcontrolador ESP32.
* `flows_node_red_freezer.json`: Arquivo JSON contendo os fluxos do Node-RED para o dashboard e as notificações.

## Configuração e Uso

### 1. Configuração do ESP32

   a. **Abra o Código:** Carregue o arquivo `esp32_freezer_monitor.ino` na sua Arduino IDE ou ambiente PlatformIO.
   b. **Bibliotecas Necessárias:** Certifique-se de que tem as seguintes bibliotecas instaladas:
      * `WiFi.h` (geralmente incluída com o core do ESP32)
      * `PubSubClient` (por Nick O'Leary)
      * `ArduinoJson` (por Benoît Blanchon)
   c. **Ajustes no Código:**
      * **Credenciais Wi-Fi:** Modifique `ssid` e `password` com os dados da sua rede Wi-Fi.
          ```cpp
          const char* ssid = "SUA_REDE_WIFI";
          const char* password = "SUA_SENHA_WIFI";
          ```
      * **Endereço do Broker MQTT:** Atualize `mqtt_server` com o endereço IP da máquina onde o Node-RED (e o broker MQTT) está a ser executado.
          ```cpp
          const char* mqtt_server = "IP_DO_SEU_BROKER_MQTT"; // Ex: "192.168.1.100"
          ```
      * **Pino do Sensor de Toque e Threshold:** O código usa `TOUCH0` (GPIO4). O `TOUCH_THRESHOLD` (atualmente `30`) pode precisar de calibração dependendo do seu sensor ou da forma como está a simular o toque. Siga as instruções de calibração no Serial Monitor ao iniciar o ESP32.
   d. **Carregue o Código:** Faça o upload do código para o seu ESP32.
   e. **Monitor Serial:** Abra o Monitor Serial para ver os logs de conexão, calibração e publicação de status.

### 2. Configuração do Node-RED

   a. **Importar o Fluxo:**
      1. Copie o conteúdo do arquivo `flows_node_red_freezer.json`.
      2. No seu Node-RED, vá em Menu (☰) > Import.
      3. Cole o JSON na caixa de texto e clique em "Import to new flow" (ou similar).
   b. **Configurar o Broker MQTT:**
      * No fluxo importado, localize os nós `mqtt in` ("Status Freezer (MQTT)").
      * Dê um duplo clique em cada um e edite a configuração do "Broker".
      * Certifique-se de que o endereço do servidor e a porta correspondem ao seu broker MQTT (o mesmo configurado no ESP32). Se estiver a usar o broker Aedes local no mesmo Node-RED, `localhost` e porta `1883` geralmente funcionam.
   c. **Configurar o Nó de E-mail:**
      * Localize o nó `e-mail` (provavelmente chamado "Notificação Freezer").
      * Dê um duplo clique para editar.
      * **To:** Insira o endereço de e-mail do destinatário das notificações (ex: `exemplo1@gmail.com`).
      * **Userid:** Insira o endereço de e-mail da conta que enviará as notificações (ex: `exemplo2@gmail.com`).
      * **Password:** Insira a senha da conta de e-mail. **Importante:** Se estiver a usar Gmail com autenticação de dois fatores, você precisará gerar uma "Senha de app" específica para o Node-RED.
      * Verifique as configurações do servidor SMTP (para Gmail: `smtp.gmail.com`, porta `465`, com SSL/TLS).
   d. **Deploy:** Clique em "Deploy" no Node-RED para ativar os fluxos.

### 3. Testando o Sistema

   a. Ligue o ESP32. Verifique no Monitor Serial se ele se conecta ao Wi-Fi e ao broker MQTT. O LED de status no ESP32 deve começar a piscar.
   b. No Node-RED, aceda ao Dashboard (geralmente em `http://SEU_IP_NODERED:1880/ui`) para ver o status do freezer.
   c. Simule a alteração do estado do freezer (ativando/desativando o sensor de toque no ESP32).
      * Verifique se o dashboard reflete a mudança.
      * Se o freezer for para o estado "DESLIGADO" (sensor de toque ativado), você deverá receber um e-mail.
   d. Para testar a notificação de "SEM COMUNICAÇÃO":
      * Desligue o ESP32.
      * Aguarde o tempo configurado no nó "Watchdog 1h Sem Dados" (atualmente 1 hora).
      * Você deverá receber um e-mail informando sobre a ausência de comunicação. (Para testes rápidos, pode reduzir temporariamente este tempo no nó `trigger` do Node-RED).

## Como Referenciar este Projeto

* **Repositório Principal:** `https://github.com/MarcelBasso/monitor-freezer-esp32-nodered`
* **Código ESP32:** `https://github.com/MarcelBasso/monitor-freezer-esp32-nodered/blob/main/esp32_freezer_monitor.txt`
* **Fluxo Node-RED:** `https://github.com/MarcelBasso/monitor-freezer-esp32-nodered/blob/main/flows_node_red_freezer.json`

## Possíveis Melhorias e Próximos Passos

* Adicionar um sensor de temperatura real ao freezer e incluir esses dados no MQTT e no dashboard.
* Implementar um sistema de log mais robusto.
* Criar alertas mais complexos (ex: se o freezer permanecer desligado por mais de X minutos).
* Usar um sensor de corrente não invasivo para uma deteção mais precisa do estado de energia do freezer.
* Integrar com outros sistemas de automação residencial.


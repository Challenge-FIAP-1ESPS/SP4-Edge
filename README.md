# Sprint 4 - Edge

## Integrantes
- Davi Daparé RM: 560721
- Erick Cardoso RM: 560440
- Gabrielly Candido RM: 560916
- João Victor Ferreira RM: 560439
- Luiza Ribeiro RM: 560200

# Portal Digital Automatizado - Monitoramento de Estoque Hospitalar

Este projeto faz parte da proposta de modernização do Hospital Infantil Sabará, com foco na automação do controle de estoque de medicamentos. A solução envolve microcontroladores ESP32, sensores e uma integração completa com a plataforma TagoIO e um portal web.

---

## Objetivo

Desenvolver um sistema inteligente de monitoramento de estoque que identifique automaticamente a retirada ou entrada de medicamentos, meça a quantidade restante por meio de um sensor ultrassônico e envie essas informações para uma plataforma em nuvem (TagoIO), onde podem ser visualizadas por meio de um dashboard integrado ao portal do hospital.

---

## Arquitetura em Camadas

> O sistema está organizado em três camadas principais: IoT, Back-End e Aplicação.

### Camada IoT (Dispositivos e Sensores)

- **Microcontrolador:** ESP32
- **Sensores:**
  - Ultrassônico HC-SR04 (mede a distância para estimar o nível de estoque)
  - RFID MFRC522 (usado para identificar usuários)
  - Display LCD 16x2 I2C (mostra dados localmente)

Esses componentes operam embarcados no ESP32, que coleta os dados, classifica o nível de estoque (Cheio, Médio ou Vazio) e envia via HTTPS para a nuvem.

### Camada Back-End (Plataforma de IoT)

- **TagoIO**
  - Recebe os dados via protocolo HTTPS com autenticação via token.
  - Armazena os dados em buckets.
  - Permite criar dashboards interativos.
  - Envia notificações automáticas via Actions (ex: e-mail ao detectar “Estoque Vazio”).

### Camada Aplicação (Interface)

- **Portal Web:** [https://portalsabara.vercel.app/](https://portalsabara.vercel.app/)
- O portal possui uma seção de cadastro da farmácia que contém um item de menu chamado `Dashboard`, no qual está incorporado o painel gerado no TagoIO via `link externo`.

## Código do Microcontrolador

O código completo do ESP32 está no arquivo CODIGO e é possível acessar pelo link do Wokwi: https://wokwi.com/projects/429667778703380481 
Vale informar que o código no Wokwi não é o oficial pois ele não suporta a biblioteca de RFID, logo, aquele é o que funciona com a simulação de leitura do cartão. Contudo, no vídeo explicativo e no arquivo "CODIGO" dentro do read.me é possível visualizar a versão oficial testada e validada.

![image](https://github.com/user-attachments/assets/e6a53b07-fd96-4a7c-ae4d-1f4a229d7356)

## Estrutura do projeto de Arduino -  Especificações Técnicas

### Como Funciona?
1. O ESP32 lê o ID do cartão RFID e a distância do sensor ultrassônico.
2. O ESP32 conecta-se à rede Wi-Fi definida.
3. A cada intervalo de tempo, os dados coletados (RFID, distância e status) são enviados via requisição HTTPS (POST) para a plataforma TagoIO.
4. O TagoIO recebe, armazena e exibe esses dados em um dashboard interativo e, quando necessário, envia notificações automáticas (ex: e-mail para estoque "Vazio").
5. As informações também estão exibidas localmente em um display LCD 16x2 apenas para testes internos.

### Bibliotecas Utilizadas
- #include <Wire.h>: Comunicação I2C.
- #include <LiquidCrystal_I2C.h>: Comunicação com o display LCD.
- #include <WiFi.h>: Conexão do ESP32 com rede Wi-Fi.
- #include <HTTPClient.h>: Realiza requisições HTTP.
- #include <WiFiClientSecure.h>: Permite enviar dados com HTTPS.

### Componentes Utilizados
- ESP32: Controlador principal responsável pela leitura dos sensores, controle do LCD e comunicação Wi-Fi.
- Sensor RFID MFRC522: Utilizado para ler o ID de um cartão RFID, simualando a identificação de um produto no estoque. 
- Sensor Ultrassônico HC-SR04: Usado para medir a distância entre o sensor e um objeto, com o intuito de simular a quantidade de estoque restante.
- LCD 16x2 I2C: Display utilizado para mostrar informações ao usuário: ID do cartão e a distância medida pelo sensor ultrassônico. (Apenas para teste interno)
- Cabos e Protoboard: Para realizar as conexões físicas entre os componentes.

### Configurações e Definições
- LiquidCrystal_I2C lcd(0x27, 16, 2): Configuração do LCD (16 colunas, 2 linhas). (Apenas para teste interno)
- #define TRIG_PIN 5 & #define ECHO_PIN 18: Pinos para o sensor ultrassônico.
- const int SEND_INTERVAL = 5000;: Intervalo entre cada envio de dados.
- const char* TAGO_DEVICE_TOKEN = "...";: Token de autenticação do dispositivo no TagoIO.
- const char* URL = "https://api.tago.io/data";: URL da API TagoIO para envio de dados.

### Funções principais
- simulateRFID(): Retorna um valor fixo "12345678" para simular um cartão RFID.//mudar depois
- readDistance(): Mede a distância via sensor ultrassônico.
- sendToTago(...): Envia os dados via HTTP POST com estrutura JSON compatível com TagoIO.
- setup(): Inicializa Wi-Fi, display LCD e sensores.
- loop(): A cada 5 segundos, lê os dados, classifica o status (Cheio, Médio, Vazio), envia à nuvem e atualiza o LCD.

### Configuração Inicial setup()
1. Inicialização do LCD (para teste local).
2. Configuração dos pinos do sensor ultrassônico.
3. Inicialização da comunicação Serial para debug.
4. Conexão com a rede Wi-Fi.
5. Mensagens iniciais no monitor serial e display.

### Configuração no loop()
1. A cada 5 segundos (`SEND_INTERVAL`):
   
   a. Leitura do valor simulado de RFID.  
   b. Leitura da distância via sensor ultrassônico.  
   c. Classificação do status do estoque com base na distância.  
   d. Montagem de objeto JSON com `rfid`, `distancia` e `status`.  
   e. Envio do JSON via HTTPS (POST) para o endpoint do TagoIO.  
   f. Exibição das informações no display LCD (apenas para teste interno).

3. Os dados são armazenados na nuvem e podem ser acessados no dashboard do TagoIO.

### Leitura do cartão RFID
- Na SPRINT 3, utilizamos apenas a função simulateRFID(), que retornava um conjunto de números ("123456789") sempre que o ID fosse solicitado. Isso porque era apeans uma simulação, nessa segunda etapa, estamos utilizando o sensor na versão física pois o simulador online não suporta as bibliotecas necessárias. Logo, aquela função não existe mais e passamos a usar a String getRFID() que retorna o valor do UID (Unique Identifier - Identificador Único) em hexadecimal.
- Durante o desenvolvimento, descobrimos que existem alguns motivos para o UID ser impresso em hexadecimal:
  - O cartão RFID armazena o UID como bytes binários.
  - O formato mais usado internacionalmente para exibir esses bytes é hexadecimal (base 16).
  - Muitas ferramentas, sistemas e leitores profissionais exibem os UIDs assim.

## Diagrama da arquitetura e fluxo do projeto
![image](https://github.com/user-attachments/assets/ff2779e7-3242-47d6-ae60-843fe879fb09)

### Explicação do Fluxograma

- Início: O sistema é iniciado.
- Importar Bibliotecas: As bibliotecas necessárias são carregadas.
- Conectar no Wi-Fi: Ao utilizar a rede de Wi-Fi do Wowki ("Wokwi-GUEST"), ele conecta automaticamente.
- Conectar no MQTT: Existe um broker público (test.mosquitto.org), o tópico ("test_topic_challenge") que é publicado pelos dispositivos e uma porta (1883).
- Entrar no loop contínuo: O programa entra no loop(), onde executa de forma contínua a checagem de sensores e o envio de dados.
- Ler sensor ultrassônico: O ESP32 mede a distância usando o sensor HC-SR04 para estimar a quantidade de estoque disponível.
- Ler cartão RFID: Um valor fixo é utilizado para simular a leitura de um cartão RFID. Em futuras versões, essa leitura será feita por hardware real.
- Configurar LCD: O display LCD é inicializado. (Apenas para teste interno)
- Exibir Dados no LCD: O ID RFID e a distância são mostrados na tela. (Apenas para teste interno)
- Nova leitura de estoque?: O sistema verifica se deve continuar a leitura.
  - Se "Não": O sistema volta no loop para ler novamente os sensores.
  - Se "Sim": 
    - Classificar status do estoque: Com base na distância medida, o sistema classifica o nível de estoque como "Cheio", "Médio" ou "Vazio".
    - Montar JSON com RFID, Distância e Status
    - Publicar JSON via MQTT no tópico "test_topic_challenge", podendo ser recebido por ferramentas como o Node-RED ou a própria plataforma de nuvem TagoIO, que o exibe em um dashboard ou executa ações com base nos dados.
   
OBS: Se estivesse em uma aplicação real conectando á plataforma em nuvem (TagoIO), antes da etapa "Nova leitura de estoque" existiriam outras duas:
- Requisição HTTP (POST) - Irá postar os dados na plataforma de nuvem
- Enviar dados para a plataforma em nuvem (TagoIO): A plataforma escolhida, TagoIO, irá receber os dados de um sensor. Nesse caso, o da distância.
Logo depois, o fluxo seria o mesmo, com exceção de envio para exibição do dashboard no Node-RED, uma vez que o próprio TagoIO poderia fazer isso.

## Anexos
![image](https://github.com/user-attachments/assets/67861e1e-a66f-47c0-a78f-ca608c5f4d27)
![image](https://github.com/user-attachments/assets/59809989-f2d6-4b99-9921-e734d0652657)
![image](https://github.com/user-attachments/assets/c67e517b-be6a-41cb-8352-4b1e012628f6)

## Links Externos
- Link Vídeo Explicação: https://youtu.be/m3dcbxTbark 

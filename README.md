# Gatekeeper (Módulo Dispositivo IoT)


![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![ESP32](https://img.shields.io/badge/Espressif_ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![MQTT](https://img.shields.io/badge/MQTT-660066?style=for-the-badge&logo=mqtt&logoColor=white)
![Mosquitto](https://img.shields.io/badge/Mosquitto-3C5280?style=for-the-badge&logo=eclipsemosquitto&logoColor=white)

---

## Integrantes
- **Caio Cesar Silva Pena**
- **Leonardo Euripedes da Silva**

## Tema
Dispositivo de Borda para Validação de Acesso RFID via MQTT

## Descrição do Problema Resolvido
Sistema de controle de acesso físico automatizado, seguro e resiliente a falhas de conectividade. Utiliza ESP32 como dispositivo de borda integrado a leitor RFID MFRC522, display LCD I2C, LED RGB e buzzer para interface com usuário.

### Fluxo de Operação
1. Modo Online: ao ler uma tag RFID, o ESP32 captura a UID e publica um JSON no broker MQTT (tópico de requisição). O back-end responde autorizando ou negando o acesso.  
2. Modo Offline (resiliência): se não houver conectividade Wi‑Fi ou ocorrer timeout de 5 s, o ESP32 consulta a lista de tags pré-autorizadas armazenada na memória não-volátil (Preferences) e decide localmente.  
3. Sincronização e Logs: eventos offline são registrados localmente; ao restabelecer conexão, o dispositivo envia (flush) os logs ao servidor e consome mensagens de sincronização para atualizar a lista interna de tags.

## Entidades / Mensagens (JSON)
- AccessRequest (envio): evento com tag lida e deviceId.  
- AccessResponse (recebimento): decisão do back-end (conceder/negado).
  ```json
  {
    "isGranted": true
  }
  ```
- AccessSync (sincronização): lista atualizada de tags autorizadas.
  ```json
  {
    "allowedTags": ["A1B2C3D4", "E5F6G7H8", "99AA88BB"]
  }
  ```
- OfflineLogs (contingência): string estruturada com histórico de acessos enquanto offline (ex.: `TAG_ID:STATUS;`).  
- LWT (Last Will and Testament): status ("online"/"offline") no broker para monitoramento do dispositivo.

## Instruções para Execução

### 1. Preparação do Ambiente (IDE)
- Instalar Arduino IDE (2.x ou superior).
- Em File > Preferences adicionar:
  `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
- Tools > Board > Boards Manager → pesquisar `esp32` (Espressif Systems) e instalar.

### 2. Bibliotecas Necessárias
Instalar via Library Manager:
- MFRC522 (GithubCommunity)
- LiquidCrystal_I2C (Frank de Brabander)
- ESP32Servo (Kevin Harrington)
- WiFiManager (tzapu)
- PubSubClient (Nick O'Leary)
- ArduinoJson (Benoit Blanchon)

Observação: SPI.h, Wire.h, time.h e Preferences.h fazem parte do core ESP32.

### 3. Compilação e Gravação
- Abrir `gatekeeper_doitv1.ino`.
- Conectar ESP32 (DevKitV1 ou equivalente) via cabo USB.
- Tools > Board: selecionar modelo (ex.: DOIT ESP32 DEVKIT V1).
- Tools > Port: selecionar porta COM / dev.
- Serial Monitor: 115200 baud.
- Upload para gravar.

### 4. Provisionamento Inicial (Portal Wi‑Fi)
- Se não houver redes conhecidas, o dispositivo cria AP `Gatekeeper_AP`.  
- Conectar ao AP, acessar 192.168.4.1, preencher: rede Wi‑Fi, senha, ID do dispositivo, servidor MQTT, porta e tipo de atuador. Salvar; o dispositivo reinicia.

## Variáveis de Ambiente (salvas em Preferences)

| Variável / Parâmetro | Descrição | Valor padrão |
| --- | --- | --- |
| deviceId | Identificador do dispositivo no broker MQTT | "GATE_01" |
| actuatorType | Tipo de atuador (1 = Servo, 0 = Relé) | "1" |
| mqttServer | Endereço IP/hostname do broker Mosquitto | "192.168.0.100" |
| mqttPort | Porta TCP do broker MQTT | "1883" |
| ntpServer | Servidor NTP | "pool.ntp.org" |

### Tópicos MQTT usados
- Requisição de Acesso: `gatekeeper/access/request`  
- Resposta do Servidor: `gatekeeper/access/response/{deviceId}`  
- Sincronização de Cache: `gatekeeper/access/sync/{deviceId}`  
- Pedido de Sincronização: `gatekeeper/access/sync/request`  
- Envio de Logs Offline: `gatekeeper/access/offline_logs`  
- Status de Conexão (LWT): `gatekeeper/status/%s` (onde %s = deviceId)

## Testes / Credenciais
Testes baseados em tags RFID físicas e payloads do back‑end. Autenticação e mapeamento de UIDs são gerenciados pelo back-end central.

O firmware grava logs locais e gera os payloads JSON conforme especificado acima; executar provisionamento e testes conforme instruções para validar integração.

## Divisão de Responsabilidades
- Leonardo Euripedes da Silva: implementação do firmware, integração de periféricos, lógica offline e testes MQTT.  
- Caio Cesar Silva Pena: requisitos arquiteturais, modelagem de payloads e validação do fluxo ponta a ponta.


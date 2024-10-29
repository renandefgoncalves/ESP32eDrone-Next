<div align="center">

# Projeto SALVA  
Sistema de Auxílio e Localização de Vítimas em Áreas Alagadas
</div>

<br>

### Detector de Objetos Usando ESP32 e Yolov5
Identificação de pessoas em situação de risco e/ou pedindo ajuda em cima de telhados em regiões alagadas, com base no desastre no Rio Grande do Sul.

- O ESP32 é um microcontrolador desenvolvido pela Espressif Systems, conhecido por sua versatilidade e recursos avançados. Ele combina um processador dual-core com conectividade Wi-Fi e Bluetooth, tornando-o ideal para aplicações em Internet das Coisas (IoT).</br></br>
- O YOLOv5 é um modelo de detecção de objetos que faz parte da série de modelos YOLO (You Only Look Once). Ele é projetado para identificar e localizar objetos em imagens e vídeos em tempo real. A principal característica do YOLOv5 é sua capacidade de combinar alta precisão com velocidade, tornando-o ideal para aplicações que exigem resposta rápida, como vigilância, robótica e veículos autônomos.

#### Passo a Passo (Periféricos)
- Ter um ESP32-CAM
- Ter uma Câmera OV2640
- Ter um Módulo FTDI
- Ter um cabo usb-c ou micro usb (ver saída compatível da FTDI) *Preferir usb-c, pois, é o padrão mais atual* 
- Ter 4 cabos conectores com saídas femea Dupont em ambas extremidades e 1 jumper (ou 5 cabos conectores, sendo 1 utilizado como jumper)

*Em posse dos itens acima, efetuar as conexões*<br>
![conexao](https://github.com/user-attachments/assets/a855df23-4f20-4def-83d9-3a9f259e4fd4) <br>
O local apontado pela seta vermelha indica o jumper, que pode ser feito com 1 cabo conector com as saídas fêmeas dupont.

*Após essas conexões, vamos a parte da programação*

#### Passo a Passo (Programação)
- Efetue o download do Software | Arduino IDE https://www.arduino.cc/en/software e Efetue a instalação
- Com o Arduino aberto, se preferir, troque a linguagem para português-BR, acessando File -> Preferences -> Language e trocando English por português-BRASIL, clique em OK e reinicie o Arduino IDE para que altera o idioma.
- Já em português-BRASIL, Acesse novamente Arquivos -> Preferências), clique em URL do Gerenciador de Placas Adicionais e insira o link: https://dl.espressif.com/dl/package_esp32_index.json, dê ok.
- Abra o Gerenciador de Placas. Clique no segundo item do canto esquerdo, que fica entre o símbolo da pasta e dos livros.
- Após abrir o gerenciador de placas, clique em "filtrar sua pesquisa..." e escreva *esp32*
- Instale esp32 por espressif (Esse passo pode demorar bastante dependendo da sua conexão e do seu computador)
- Clique no retângulo -> Selecione outra placa e porta -> busque por "AI Thinker ESP32-CAM"
- Se aparecer nenhuma porta conectada, conecte o USB ao computador e verifique a porta que será habilitada. (Caso não habilite nenhuma porta, tente outro cabo usb), caso apareça mais de uma porta, remova o cabo e verifique o número da porta que será removido, reinsira o cabo e selecione novamente a porta que será habilitada.
- Após confirmar que as conexões dos cabos estão de acordo com a imagem https://github.com/user-attachments/assets/a855df23-4f20-4def-83d9-3a9f259e4fd4, a placa selecionada é a "AI Thinker ESP32-CAM" e a porta certa está selecionada, insira o código no seu esboço, substituindo
``` 
void setup() {
  // put your setup code here, to run once:

}

void loop() {
  // put your main code here, to run repeatedly:

}
```
pelo código:

```
#include <WebServer.h> //biblioteca responsável em instanciar um servidor web. Sobe um servidor local de imagens na porta 80 para que o seu navegador possa compilar e te mostrar ao usuário.
#include <WiFi.h> //biblioteca que gerencia o hardware da placa para acessar o wifi
#include <esp32cam.h> //gerencia o hadware para a câmera funciona
 
const char* WIFI_SSID = "ssid wifi celular"; //celular Apple não funciona, porque ele não tem 2.4GHz
const char* WIFI_PASS = "senha";
//a variável const é uma boa prática para alocar e economizar espaço.
//char é um tipo de variável que mexe com caracteres.
//* indica que é um ponteiro, e um ponteiro pega uma faixa de endereços livres na memória


int8_t nivelBrilho = 100; //esse número indica o brilho do flash. Varia de 0 (desligado) a 255 (brilho máximo). Não use níveis acima de 200 porque vai aquecer o circuito do flash e posteriormente, queimá-lo
#define FLASH_GPIO_NUM 4  //definindo o pino 4 onde está o flash
 
WebServer server(80); //WebServer é obrigatório, server é um nome dado pelo programador. 80 é a porta do HTML
 
 
static auto loRes = esp32cam::Resolution::find(320, 240); //preparo da martriz de pixels com a imagem
static auto midRes = esp32cam::Resolution::find(350, 530); //em diferentes resoluções
static auto hiRes = esp32cam::Resolution::find(800, 600);

void serveJpg() // void não retorna variável, nome serveJpg é definido pelo programador
{
  auto frame = esp32cam::capture(); //está capturando imagens e guardando em frame de forma automatica
  if (frame == nullptr) { //se frame for nulo, deu erro na câmera
    Serial.println("CAPTURE FAIL"); //captura deu errado
    server.send(503, "", ""); //indica o código 503 de serviço indisponível no HTML
    return; //não retorna nada, mas libera a função serveJpg()
  } //fecha o if
  Serial.printf("CAPTURE OK %dx%d %db\n", frame->getWidth(), frame->getHeight(),
                static_cast<int>(frame->size())); //frase impressa no monitor serial, onde %d reserva um espaço para ser preenchido com o respectivo conteúdo de frame->getWidth(), frame->getHeight() e   static_cast<int>(frame->size())
 
  server.setContentLength(frame->size()); //manda pra frente o tamanho da tela para o navegador saber
  server.send(200, "image/jpeg"); //manda o padrão de imagem para o seu navegador
  WiFiClient client = server.client(); //o hardware conversa com o software para enviar isso via wifi
  frame->writeTo(client); //gerencia a porta 80 no seu navegador
} //fecha a função do serveJpg
 
void handleJpgLo() //é uma função, porém gerencia a resolução baixa 320x240
{
  if (!esp32cam::Camera.changeResolution(loRes)) { //está testando se a resolução está com problema
//o ! indica a negação. Então estamos perguntando se “esp32cam::Camera.changeResolution” == falso
    Serial.println("SET-LO-RES FAIL"); //msg está com problema
  }
  serveJpg(); //aqui ele carrega a função acima
}
 
void handleJpgHi() //igual ao low, porém gerencia a resolução mais alta 800x600
{
  if (!esp32cam::Camera.changeResolution(hiRes)) {
    Serial.println("SET-HI-RES FAIL");
  }
  serveJpg();
}
 
void handleJpgMid() //gerencia a resolução média 350x530
{
  if (!esp32cam::Camera.changeResolution(midRes)) {
    Serial.println("SET-MID-RES FAIL");
  }
  serveJpg(); //carrega a função explicada acima
}
 
 
void  setup(){
  Serial.begin(115200); //define o baud rate que é a taxa de bits que vai para o monitor serial
  Serial.println(); //imprime uma linha vazia
  {
    using namespace esp32cam; //estou criando um espaço de variáveis
    Config cfg; //carregando o arquivo de configuração da pinagem do ESP32-CAM
    cfg.setPins(pins::AiThinker); //use o modelo do ESP32 da empresa AiThinker
    cfg.setResolution(hiRes); //seta a resolução para a mais alta
    cfg.setBufferCount(2); //carrega um buffer. Buffer é o exemplo da caixa d’água residencial
    cfg.setJpeg(80); //seta a porta 80 na câmera
 
    bool ok = Camera.begin(cfg); //arquivo de configuração chamado cfg é carregado na camera por meio do begin
    Serial.println(ok ? "CAMERA OK" : "CAMERA FAIL"); //teste + Serial.println
//se ok == true, "CAMERA OK", senão "CAMERA FAIL"
  }
  WiFi.persistent(false); //tá dizendo que nao é para persistir o WiFi
  WiFi.mode(WIFI_STA); //tá dizendo que o seu ESP32 precisa de um roteador para funcionar
  WiFi.begin(WIFI_SSID, WIFI_PASS); //carrega a credenciais para o seu WiFi
  while (WiFi.status() != WL_CONNECTED) { //enquanto o WiFi tenta se conectar, imprime um .
    Serial.print("."); //imprime pontos
    delay(500);
  }
  Serial.print("http://"); //imprime http://
  Serial.println(WiFi.localIP()); //ele busca o IP local e imprime junto com a frase anterior
  Serial.println("  /cam-lo.jpg"); //imprime as opções para vc escolher
  Serial.println("  /cam-hi.jpg");
  Serial.println("  /cam-mid.jpg");
 
  server.on("/cam-lo.jpg", handleJpgLo); //aqui é aquela parte que você escolhe a versão de 
  server.on("/cam-hi.jpg", handleJpgHi); //resolução na URL
  server.on("/cam-mid.jpg", handleJpgMid);
 
  server.begin(); //ele ativa o server
  pinMode(FLASH_GPIO_NUM, OUTPUT); //define o pino do flash como saíde
}
 
void loop()
{
  server.handleClient(); //gerencia a atualização das imagens e manda pra frente
  analogWrite(FLASH_GPIO_NUM,nivelBrilho); //se o esp se conectar no wifi, o flash acende fraquinho para não queimar. 
}
```
- após copiar e colar este código, clique na seta verde, localizada no centro dos ícones.
- O programa começará compilando o código
- após isso, ele tentará transferir o programa para a placa. Quando aparecer ..., pressione o botão preto localizado na parte traseira da placa por 3 segundos e solte.
- A leitura continuará até 100% e retornará a seguinte mensagem: RTS pin ...
- Já pode remover o jumper da placa.

*** terminar

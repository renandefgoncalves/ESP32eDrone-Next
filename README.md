<div align="center">

# Projeto SALVA  
Sistema de Auxílio e Localização de Vítimas em Áreas Alagadas
</div>

<br>

### Detector de Objetos Usando ESP32 e Yolov5
Identificação de pessoas em situação de risco e/ou pedindo ajuda em cima de telhados em regiões alagadas, com base nos desastres ocorridos no Rio Grande do Sul.

- O ESP32 é um microcontrolador desenvolvido pela Espressif Systems, conhecido por sua versatilidade e recursos avançados. Ele combina um processador dual-core com conectividade Wi-Fi e Bluetooth, tornando-o ideal para aplicações em Internet das Coisas (IoT).</br></br>
- O YOLOv5 é um modelo de detecção de objetos que faz parte da série de modelos YOLO (You Only Look Once). Ele é projetado para identificar e localizar objetos em imagens e vídeos em tempo real. A principal característica do YOLOv5 é sua capacidade de combinar alta precisão com velocidade, tornando-o ideal para aplicações que exigem resposta rápida, como vigilância, robótica e veículos autônomos.

### Linguagem utilizada
- Python
- C++

## Passo 01: Periféricos (Possuir os seguintes equipamentos)
- ESP32-CAM
- Câmera OV2640
- Módulo FTDI
- Cabo usb-c ou Micro Usb (ver saída compatível da FTDI) *Preferir usb-c, pois, é o padrão mais atual* 
- 4 cabos conectores com saídas fêmea Dupont em ambas extremidades e 1 jumper (ou acrescente um cabo conector, para ser utilizado como jumper)

*Em posse dos itens acima, efetuar as conexões:*<br>
<p align="center">
  <img src="https://github.com/user-attachments/assets/a855df23-4f20-4def-83d9-3a9f259e4fd4" width="50%" />
</p>
<br>
O local apontado pela seta vermelha indica o jumper, que pode ser feito com 1 cabo conector com as saídas fêmeas dupont, ligados em U.

## Passo 02: Programação (Softwares) Video Passo a Passo (https:youtube...)
- Efetue o download do Software | Arduino IDE https://www.arduino.cc/en/software e do software | Visual Studio Code https://code.visualstudio.com/. *No lugar do Visual Studio Code, pode ser usada a IDE de sua preferência, compatível com Python.*
- Efetue a instalação de ambos softwares
- Com o Arduino aberto, se preferir, troque a linguagem para português-BR, acessando File -> Preferences -> Language e trocando English por português-BRASIL, clique em OK e reinicie o Arduino IDE para que altera o idioma.
- Já em português-BRASIL, Acesse novamente Arquivos -> Preferências), clique em URL do Gerenciador de Placas Adicionais e insira o link: https://dl.espressif.com/dl/package_esp32_index.json, dê ok.
- Abra o Gerenciador de Placas. Clique no segundo item do canto esquerdo, que fica entre o símbolo da pasta e dos livros.
- Após abrir o gerenciador de placas, clique em "filtrar sua pesquisa..." e escreva *esp32*
- Instale esp32 por espressif (Esse passo pode demorar bastante dependendo da sua conexão e do seu computador)
- Clique no retângulo -> Selecione outra placa e porta -> busque por "AI Thinker ESP32-CAM"
- Se aparecer 'nenhuma porta conectada', conecte o USB ao computador e verifique a porta que será habilitada. (Caso não habilite nenhuma porta, tente outro cabo usb), caso apareça mais de uma porta, remova o cabo e verifique o número da porta que será removido, reinsira o cabo e selecione novamente a porta que será habilitada.
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










## Passo 03: Organizando seu projeto
Por praticidade e organização, hospedaremos nossos arquivos no Google Drive, isso permite que os arquivos sejam acessados de qualquer lugar e ainda teremos uma acessibilidade melhor para manipular os arquivos no Google Collab 

### 1.1) No seu PC, crie 4 pastas/subpastas da seguinte forma:
- **images**
  - **/train**
    - nessa pasta você colocará 48 imagens de treino
  - **/val**
    - nessa pasta você colocará 12 imagens de validação do seu modelo.<br> 

- **labels**
  - **/train**
    - labels das imagens de treino
  - **/val**
    - labels das imagens de validação

---

## Passo 04: Busca de Imagens
#### 4.1) Procure imagens contendo o objeto e salve tudo em uma pasta local no seu PC de sua preferência.
#### 4.2) Renomeie cada arquivo nesse formato: [NOME DO OBJETO]_[NUMERO DO EXEMPLO] (Opcional) - **Exemplo**: telhado_01. Isso vai facilitar na visualização e organização dos arquivos.
#### 4.3) Após isso, separe 80% em uma nova pasta chamada **train** e o resto em uma pasta chamada **val**
#### 4.4) Mova as pastas train e val para uma nova pasta chamada **images**.
#### 4.5) O resultado será esse:
```images/train```
```images/val```

## Passo 05: MakeSense.AI
#### 5.1) Entre no site makesense.ai | https://www.makesense.ai/ e clique em "Get started";

#### 5.2) Clique no quadrado "Drop imagens or Click here to select them", aponte para a sua pasta local ```images/train```, dê um "Ctrl + A" para selecionar todas as imagens ao mesmo tempo, e confirme. (Você precisará repetir esse passo para ```images/val```);

#### 5.3) Clique em Object Detection;

#### 5.4) Clique em Start project;

#### 5.5) Na tela seguinte, com o título Create labels, clique no sinal de + que fica no canto esquerdo superior. Esse botão serve para você criar os labels telhado e pessoa. Então, após clicar no sinal de +, você aciona quantas labels que você desejar.
(crie um rótulo que faça sentido para o objeto que você está reconhecendo. Nesse caso, telhado.)

#### 5.6) Crie uma caixa delimitadora retangular e não polígono (POLÍGONO NÃO FUNCIONA!) para cada objeto em cada imagem e à direita da sua tela, escolha o label telhado. Na sua seleção da moldura, dê ênfase ao que você está pretendendo reconhecer, que nesse caso, será telhado inundado.

Importante! Se houver mais de um objeto na foto, crie uma caixa para cada objeto, mesmo que eles sejam do mesmo tipo. Exemplo: Se houver dois telhados separados na foto, crie uma caixa para o primeiro telhado e outra caixa para o outro telhado, e não uma caixa só para os dois juntos. Vide imagem abaixo:
https://blog.totalcad.com.br/wp-content/uploads/2017/06/m-d987916938a7aebcac0fcde7311a5b9ce52f8dc7-740x360.jpg

## Passo 06: Exportando as Imagens
6.1) Após criar as caixas para todas as imagens, vá em "Actions" e "Export annotations".

6.2) Selecione A.zip package containing files in YOLO format. Dentro desse zip estão todos os rótulos para cada imagem.

6.3) Salve esse zip na subpasta labes/train que você criou na ETAPA-01.

6.4) Descompacte o conteúdo do .zip nessa subpasta e os seus TXT estarão no labes/train.

6.5) Repita o processo com as images/val.

6.6) No final, você terá que ter os labels de treino e validação dentro da sua respectiva pasta da seguinte forma:

- **images**
  - **/train**
    - nessa pasta você colocará 48 imagens de treino
  - **/val**
    - nessa pasta você colocará 12 imagens de validação do seu modelo.<br> 

- **labels**
  - **/train**
    - labels das imagens de treino
  - **/val**
    - labels das imagens de validação

6.7) Copie toda essa estrutura de pastas e subpastas para o seu Google Drive, em pasta ou subpasta sem espaços ou caracteres especiais, de preferência, o mais próximo da raiz. Exemplo:
```/content/drive/MyDrive/reconhecedor-obj```

## Passo 07: Criando o código no Google Collab | Video Passo a Passo (https:youtube...)
7.1) Acesse o site do Google Collab | https://colab.research.google.com/ **Certifique-se de estar logado com a mesma conta que fez o upload dos arquivos no Google Drive**

7.2) Clique em arquivo -> Novo Notebook no Drive

7.3) Troque a instância para uma mais potente. Acesse **Alterar Tipo de Ambiente de Execução**, selecione **T4 GPU** e clique em **Salvar**

7.2) Agora podemos começar a Codar. Faremos isso em blocos:

> **Anexando arquivos do Google Drive:** O Código abaixo faz com que o Google Collab identifique as imagens que estão no seu Google Drive.

```
from google.colab import drive
drive.mount('/content/drive')
```

> **Clonando Repositório Yolov5** Faz um clone do repositório Yolov5 para dentro do seu projeto.
```
! git clone https://github.com/ultralytics/yolov5.git
```

> **Instale as bibliotecas necessárias** Faz o download automático de todas bibliotecas necessárias para rodar o repositório do Yolov5 no seu ambiente. Essas bibliotecas são indicadas pelo próprio dono do repositório.
```
! pip install -r yolov5/requirements.txt
```

> **Crie um arquivo chamado telhado.yaml usando o bloco de notas** Coloque o seguinte conteúdo dentro desse arquivo:
Observação: mantenha essa raiz /content/drive/MyDrive/ e mantenha o final /images/train/ e atualize o meio da raiz apontando para as pastas e subpastas do seu drive. Exemplo: /content/drive/MyDrive/reconhecedor-obj/ manter o /images/train/ e depois o /images/val/
```
train: /content/drive/reconhecedor-obj/images/train/
val: /content/drive/MyDrive/reconhecedor-obj/images/val/

names: # Defina aqui as labels do seu dataset
  0: "Telhado"
  1: "Pessoa"
  # ...
```
copie o arquivo ```telhado.yaml``` para dentro da pasta** reconhecedor-obj que está no Google Drive ```/content/drive/MyDrive/reconhecedor-obj/teclado.yaml```

! cp /content/drive/MyDrive/FIAP/FIAP_NEXT/TESTE/telhado.yaml yolov5/data/

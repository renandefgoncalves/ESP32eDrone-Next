<div align="center">

<img src="https://github.com/user-attachments/assets/18cf580f-5a08-40a0-8cb3-e113e3c687a5" width="20%"/>

# ⛑️ Projeto SALVA ⛑️
Sistema de Auxílio e Localização de Vítimas em Áreas Alagadas
</div>
</br>

## Descrição do Projeto: Detector de Pessoas e Objetos usando ESP32 e Yolov5 operado por Drone
Esse projeto visa Identificar pessoas em situação de risco e/ou pedindo ajuda em cima de telhados ou locais de difícil acesso em regiões alagadas, com base nos desastres ocorridos pelas enchentes no Rio Grande do Sul.

- O ESP32 é um microcontrolador desenvolvido pela Espressif Systems, conhecido por sua versatilidade e recursos avançados. Ele combina um processador dual-core com conectividade Wi-Fi e Bluetooth, tornando-o ideal para aplicações em Internet das Coisas (IoT).</br></br>
- O YOLOv5 é um modelo de detecção de objetos que faz parte da série de modelos YOLO (You Only Look Once). Ele é projetado para identificar e localizar objetos em imagens e vídeos em tempo real. A principal característica do YOLOv5 é sua capacidade de combinar alta precisão com velocidade, tornando-o ideal para aplicações que exigem resposta rápida, como vigilância, robótica e veículos autônomos.</br></br>
- O Drone não será abordado nesse passo a passo, devido diversidade de modelos e conexões existentes.

### Linguagens utilizadas
```python
Python
```

``` c#
C++
```
<br></br>

## Passo 01: Periféricos (Possuir os seguintes equipamentos)
- ESP32-CAM
- Câmera OV2640
- Módulo FTDI
- Cabo usb-c ou Micro Usb (ver saída compatível da FTDI) *Preferir usb-c, pois, é o padrão mais atual* 
- 4 cabos conectores com saídas fêmea Dupont em ambas extremidades e 1 jumper (ou acrescente um cabo conector, para ser utilizado como jumper)

*Em posse dos itens acima, efetuar as conexões:*<br> O local apontado pela seta vermelha indica o jumper, que pode ser feito com 1 cabo conector com as saídas fêmeas dupont, ligados em U.
<div align="center">
  <img src="https://github.com/renandefgoncalves/Reconhecedor-de-Objetos/blob/main/img/Ligacao-FTDI.png" width="70%" /> <br><br>

  <table>
    <tr>
      <th>Pinos do ESP32-CAM</th>
      <th>Pinos Placa FTDI</th>
      <th>Descrição</th>
      <th>Cores</th>
    </tr>
    <tr>
      <td>5V</td>
      <td>VCC</td>
      <td>Alimentação Positiva</td>
      <td>Vermelho</td>
    </tr>
    <tr>
      <td>GND</td>
      <td>GND</td>
      <td>Alimentação Negativa</td>
      <td>Preto</td>
    </tr>
    <tr>
      <td>UOT</td>
      <td>RX</td>
      <td>Pinos de Comunicação cruzados. TX ligado no RX</td>
      <td>Azul</td>
    </tr>
    <tr>
      <td>UOR</td>
      <td>TX</td>
      <td>Pinos de Comunicação cruzados. RX ligado no TX</td>
      <td>Verde</td>
    </tr>
    <tr>
      <td>IO0 com GND</td>
      <td>--------</td>
      <td>Habilita o modo programação</td>
      <td>Cinza</td>
    </tr>
  </table>
</div>

<br>

## Passo 02: Organizando seu projeto
Por praticidade e organização, hospedaremos nossos arquivos no Google Drive, isso permite que os arquivos sejam acessados de qualquer lugar e ainda teremos uma acessibilidade melhor para manipular os arquivos no Google Collab 

2.1) No seu PC, crie 4 pastas/subpastas da seguinte forma:
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
<br></br>

## Passo 03: Busca de Imagens
3.1) Procure imagens contendo o objeto e salve tudo em uma pasta local de sua preferência no seu PC, neste caso, usaremos *telhados* e *pessoas*

3.2) Renomeie cada arquivo nesse formato: **telhado_01**. Isso vai facilitar na visualização e organização dos arquivos

3.3) Após isso, separe 80% em uma nova pasta chamada **train** e o resto em uma pasta chamada **val**

3.4) Mova as pastas train e val para uma nova pasta chamada **images**.

3.5) O resultado será esse:
```images/train```
```images/val```
<br><br>

## Passo 04: MakeSense.AI
4.1) Entre no site makesense.ai | https://www.makesense.ai/ e clique em "Get started";

4.2) Clique no quadrado "Drop imagens or Click here to select them", aponte para a sua pasta local ```images/train```, dê um "Ctrl + A" para selecionar todas as imagens ao mesmo tempo, e confirme. (Você precisará repetir esse passo para ```images/val```);

4.3) Clique em Object Detection;

4.4) Clique em Start project;

4.5) Na tela seguinte, com o título Create labels, clique no sinal de + que fica no canto esquerdo superior. Esse botão serve para você criar os labels telhado e pessoa. Então, após clicar no sinal de +, você aciona quantas labels que você desejar.
(crie um rótulo que faça sentido para o objeto que você está reconhecendo. Nesse caso, telhado.)

4.6) Crie uma caixa delimitadora retangular e não polígono **POLÍGONO NÃO FUNCIONA!** para cada objeto em cada imagem e à direita da sua tela, escolha o label telhado. Na sua seleção da moldura, dê ênfase ao que você está pretendendo reconhecer, que nesse caso, será telhado inundado.

**Importante! Se houver mais de um objeto na foto, crie uma caixa para cada objeto, mesmo que eles sejam do mesmo tipo. Exemplo: Se houver dois telhados separados na foto, crie uma caixa para o primeiro telhado e outra caixa para o outro telhado, e não uma caixa só para os dois juntos. Vide imagem abaixo:**
https://blog.totalcad.com.br/wp-content/uploads/2017/06/m-d987916938a7aebcac0fcde7311a5b9ce52f8dc7-740x360.jpg
<br><br>

## Passo 05: Exportando as Imagens
5.1) Após criar as caixas para todas as imagens, vá em "Actions" e "Export annotations".

5.2) Selecione A.zip package containing files in YOLO format. Dentro desse zip estão todos os rótulos para cada imagem.

5.3) Salve esse zip na subpasta labes/train que você criou na ETAPA-01.

5.4) Descompacte o conteúdo do .zip nessa subpasta e os seus TXT estarão no labes/train.

5.5) Repita o processo com as images/val.

5.6) No final, você terá que ter os labels de treino e validação dentro da sua respectiva pasta da seguinte forma:

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

5.7) Copie toda essa estrutura de pastas e subpastas para o seu Google Drive, em pasta ou subpasta sem espaços ou caracteres especiais. Exemplo:
```/content/drive/MyDrive/reconhecedor-obj```
<br></br>

## Passo 06: Criando o código no Google Collab
6.1) Acesse o site do Google Collab | https://colab.research.google.com/ **Certifique-se de estar logado com a mesma conta que fez o upload dos arquivos no Google Drive**

6.2) Clique em arquivo -> Novo Notebook no Drive

6.3) Troque a instância para uma mais potente. Acesse **Alterar Tipo de Ambiente de Execução**, selecione **T4 GPU** e clique em **Salvar**

6.4) Agora podemos começar a Codar. Faremos isso em blocos:

> **Anexando arquivos do Google Drive:** O Código abaixo faz com que o Google Collab identifique as imagens que estão no seu Google Drive.

``` python
from google.colab import drive
drive.mount('/content/drive')
```

> **Clonando Repositório Yolov5** Faz um clone do repositório Yolov5 para dentro do seu projeto.
``` python
! git clone https://github.com/ultralytics/yolov5.git
```

> **Instale as bibliotecas necessárias** Faz o download automático de todas bibliotecas necessárias para rodar o repositório do Yolov5 no seu ambiente. Essas bibliotecas são indicadas pelo próprio dono do repositório.
``` python
! pip install -r yolov5/requirements.txt
```

> **Crie um arquivo chamado telhado.yaml usando o bloco de notas** Coloque o seguinte conteúdo dentro desse arquivo:
Observação: mantenha essa raiz /content/drive/MyDrive/ e mantenha o final /images/train/ e atualize o meio da raiz apontando para as pastas e subpastas do seu drive. Exemplo: /content/drive/MyDrive/reconhecedor-obj/ manter o /images/train/ e depois o /images/val/
``` python
train: /content/drive/reconhecedor-obj/images/train/
val: /content/drive/MyDrive/reconhecedor-obj/images/val/

names: # Defina aqui as labels do seu dataset
  0: "Telhado"
  1: "Pessoa"
  # ...
```
copie o arquivo ```telhado.yaml``` para dentro da pasta "reconhecedor-obj" que está no Google Drive ```/content/drive/MyDrive/reconhecedor-obj/telhado.yaml```


> **Insere o arquivo .yaml para treinamento** Este é o primeiro passo para começar o treinamento
``` python
! cp /content/drive/MyDrive/reconhecedor-obj/telhado.yaml yolov5/data/
```

> **Treinamento em si** Você pode mexer na variável --epochs mudando o número na frente dele. Exemplo, você pode subir o 40 para 60 e analisar os resultados de acurácia do seu modelo.
``` python
! python yolov5/train.py --data telhado.yaml --weights yolov5s.pt --img 640 --epochs 40
```

6.5) Nessa etapa, você precisa criar uma subpasta chamada test, ficando assim:

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

- test

6.6) Escolha uma imagem da sua pasta ```/images/train``` que tenha uma vista superior e de perto de um telhado com água em volta e copie para ```/test```. Busque outras imagens que não tenham sido utilizados no ```train``` e ```val```.

6.7) Atualize o path na frente de --source com o caminho do seu test. Exemplo: ```--source drive/MyDrive/reconhecedor-obg/test/```
``` python
import os
import subprocess

def get_latest_train_run_folder():
    subfolders = [f.path for f in os.scandir('yolov5/runs/train') if f.is_dir()]
    latest_folder = max(subfolders, key=os.path.getctime, default=None)
    return latest_folder

latest_run = get_latest_train_run_folder()
result = subprocess.run(f'python yolov5/detect.py --weights {latest_run}/weights/best.pt --img 640 --source drive/MyDrive/PROJETOS/DRONE/2024/test/ --data yolov5/data/telhado.yaml', shell=True, capture_output=True, text=True)
if latest_run:
    # COMANDO
    result = subprocess.run(f'python yolov5/detect.py --weights {latest_run}/weights/best.pt --img 640 --source drive/MyDrive/PROJETOS/DRONE/2024/test/ --data yolov5/data/telhado.yaml', shell=True, capture_output=True, text=True)
    print(result.stdout)
    print(result.stderr)
else:
    print("Não foi possível encontrar a pasta de treinamento mais recente.")
```
6.8) Faça o download do arquivo ```best.pt``` que está no seguinte path: ```yolov5/runs/train/exp/weights/best.pt```, **esse arquivo será utilizado posteriormente**.

<br></br>

## Passo 07: Programação (Softwares)
7.1) Efetue o download dos seguintes Softwares:

- Arduino IDE https://www.arduino.cc/en/software <br>
[![](https://markdown-videos-api.jorgenkh.no/youtube/_s8Dx61ECJA)](https://youtu.be/_s8Dx61ECJA)


- Visual Studio Code https://code.visualstudio.com/ <br>
[![](https://markdown-videos-api.jorgenkh.no/youtube/1s_cB4q-ZKI)](https://youtu.be/1s_cB4q-ZKI)


**No lugar do Visual Studio Code, pode ser usada a IDE de sua preferência, compatível com Python.**

7.2) Efetue a instalação de ambos softwares

7.3) Com o Arduino aberto, se preferir, troque a linguagem para português-BR, acessando File -> Preferences -> Language e trocando English por português-BRASIL, clique em OK e reinicie o Arduino IDE para que altera o idioma. 

[![](https://markdown-videos-api.jorgenkh.no/youtube/Im_17-sjf0w)](https://youtu.be/Im_17-sjf0w)


7.4) Acesse novamente Arquivos -> Preferências, clique em URL do Gerenciador de Placas Adicionais e insira o link: https://dl.espressif.com/dl/package_esp32_index.json, dê ok.

[![](https://markdown-videos-api.jorgenkh.no/youtube/poVgoe4Pon8)](https://youtu.be/poVgoe4Pon8)


7.5) Abra o Gerenciador de Placas. Clique no segundo item do canto esquerdo, que fica entre o símbolo da pasta e dos livros, clique em "filtrar sua pesquisa..." e escreva *esp32*

7.6) Instale esp32 por "Espressif System"

7.7) Faça o download das bibliotecas 
- [ESP Async WebServer](https://github.com/renandefgoncalves/Reconhecedor-de-Objetos/blob/main/Bibliotecas/ESPAsyncWebServer-master.zip)
- [ESP32CAM Main](https://github.com/renandefgoncalves/Reconhecedor-de-Objetos/blob/main/Bibliotecas/esp32cam-main.zip)
- [ESP32 SDK Master](https://github.com/renandefgoncalves/Reconhecedor-de-Objetos/blob/main/Bibliotecas/esp8266-esp32-sdk-master.zip)

[![](https://markdown-videos-api.jorgenkh.no/youtube/df5JZYzU0JU)](https://youtu.be/df5JZYzU0JU)


clique em Rascunho -> Incluir Biblioteca -> Adicionar Biblioteca.ZIP... , e selecione as 3 bibliotecas que fez o download uma de cada vez, repetindo o processo.

7.8) Clique no retângulo -> Selecione outra placa e porta -> busque por "AI Thinker ESP32-CAM" 
[![](https://markdown-videos-api.jorgenkh.no/youtube/_gqzeojdfGuw)](https://youtu.be/gqzeojdfGuw)

7.9) Se aparecer 'nenhuma porta conectada', conecte o USB ao computador e verifique a porta que será habilitada. (Caso não habilite nenhuma porta, tente outro cabo usb), caso apareça mais de uma porta, remova o cabo e verifique o número da porta que será removido, reinsira o cabo e selecione novamente a porta que será habilitada.

7.10) Após confirmar que as conexões dos cabos estão de acordo com a imagem https://github.com/user-attachments/assets/a855df23-4f20-4def-83d9-3a9f259e4fd4, a placa selecionada é a "AI Thinker ESP32-CAM" e a porta certa está selecionada, insira o código no seu esboço.
Apague esse código:
``` C++
void setup() {
  // put your setup code here, to run once:

}

void loop() {
  // put your main code here, to run repeatedly:

}
``` 
Copie e Cole esse código:

``` C++
#include <WebServer.h> //biblioteca responsável em instanciar um servidor web. Sobe um servidor local de imagens na porta 80 para que o seu navegador possa compilar e te mostrar ao usuário.
#include <WiFi.h> //biblioteca que gerencia o hardware da placa para acessar o wifi
#include <esp32cam.h> //gerencia o hadware para a câmera funciona
 
const char* WIFI_SSID = "ssid wifi celular";
const char* WIFI_PASS = "password";
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
7.11) Após colar este código, altere nas linhas 5 e 6 sua rede wi-fi e senha pelas que você utilizará (Essa rede deve ser 2.4 GHz, não é possível conexão com redes 5 GHz. **Não funciona em iPhones, pois não possuem rede 2.4 GHz**

[![](https://markdown-videos-api.jorgenkh.no/youtube/5Tq-6lDQ9BQ)](https://youtu.be/5Tq-6lDQ9BQ)

  
``` c++ 
const char* WIFI_SSID = "ssid wifi celular"; Troque o "ssid wifi celular" pelo nome certo da sua rede
const char* WIFI_PASS = "password"; Troque a senha "password" pela que configurou ao criar a rede
```


7.12) Clique na seta verde, localizada no centro dos ícones. O programa começará compilando o código
7.13) Após isso, ele tentará transferir o programa para a placa. Quando aparecer ..., pressione o botão preto localizado na parte traseira da placa por 3 segundos e solte.
7.14) A leitura continuará até 100% e retornará a seguinte mensagem: RTS pin ...
7.15) Já pode remover o jumper da placa.

*** terminar

## Passo 08: Rodando o Modelo no Visual Code
7.1) Abra um Visual Code;

7.2) Crie um arquivo Python, cole esse código e salve-o na mesma pasta em que está o .ino e o best.pt:

``` python
import cv2
import torch
import numpy as np
import urllib
import pathlib
from pathlib import Path

pathlib.PosixPath = pathlib.WindowsPath

path = 'best.pt' # TROQUE PELO CAMINHO DO SEU PESO CASO QUEIRA (best.pt que foi gerado no treinamento) ex: yolov5/runs/train/exp9/weights/best.pt
image_url = 'http://192.168.10.250/cam-hi.jpg' # TROQUE PELO LINK GERADO NO MONITOR SERIAL

model = torch.hub.load('ultralytics/yolov5', 'custom', path, force_reload=True)
model.conf = 0.6 #essa variável determina a acurácia do seu modelo. Faça testes para encontrar o melhor resultado.

print(path)

while True:
    img_resp=urllib.request.urlopen(url=image_url)
    imgnp=np.array(bytearray(img_resp.read()),dtype=np.uint8)
    im = cv2.imdecode(imgnp,-1)
    results = model(im)
    print(results)
    frame = np.squeeze(results.render())
    cv2.imshow('Deteccao', frame)
    key=cv2.waitKey(5)
    if key==ord('q'):
        break

cv2.destroyAllWindows()
```
7.3) Só lembrando que o seu arquivo best.pt fica num path parecido como esse: yolov5/runs/train/exp9/weights/best.pt do seu Google Drive. Esse exp9 é o número de vezes que você treinou. Portanto, ele vai incrementar a cada treinamento.

7.4) Dê o play no seu Visual Studio e uma tela pop-up do Python vai se abrir com imagens capturadas da sua câmera do ESP32-CAM.


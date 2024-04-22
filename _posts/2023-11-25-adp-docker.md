---
layout: post
title: ADP 시험을 위한 Jupyter Notebook 환경 구축
featured-img: sleek
category: [Certificate]
summary: Docker로 Jupyter notebook 환경 커스터마이징
---

# Introduction
- 한국데이터산업진흥원 주관 ADP(Advanced Data analytics Professional) 실기 수험생 중, Python을 사용하는 분들이 시험장에서 마주할 Jupyter notebook 환경을 Docker image로 작성하였습니다.
- 이 문서와 관련된 모든 파일들은 제 개인 github에 있습니다. 혹시 Dockerfile을 다루는 데 능숙하시다면, Dockerfile을 직접 수정하여 환경을 고치는 것도 가능합니다.
- Github link: [https://github.com/hjben/adp/tree/master/docker](https://github.com/hjben/adp/tree/master/docker)   

# Prerequisites
- 저희 집은 Mac을 사랑하는 관계로... Windows PC를 전혀 가지고 있지 않아서 Windows 환경 테스트를 못해봤습니다. 그래서, Linux shell 기반의 OS(MacOS, Ubuntu Linux 등)에서 작업하시는 것을 추천드립니다.
- Docker image는 Macbook Pro M2, Docker desktop for Mac 4.22.0 버전에서 작성되었습니다. Docker image 실행 환경도 동일합니다.

# Docker container construction
### 1. Run docker desktop
먼저, Docker desktop을 설치하고, 실행합니다. <br>
Docker 관련 명령어를 사용하려면 Docker desktop이 실행 중이어야 합니다.

### 2. Download docker image
Docker image를 다운로드 받습니다. 다운로드 명령어는 아래와 같습니다.
```
docker pull hjben/adp-python:latest
```

### 3-1. Generate docker container (Using Automated shell)
(1) 위 Github 링크에서 _docker-script_ 경로에 있는 shell 파일 2개 (container-init.sh, container-remove.sh)를 다운로드 받아서, Docker 명령어가 사용 가능한 경로로 파일을 복사합니다.

(2) CLI(=Command Line Interface, 터미널)을 열고, _./container-init.sh_ 를 실행시켜서 Docker image를 실행시킵니다. Shell script의 Parameter는 정해진 순서대로 사이에 공백을 하나씩 넣어서 입력합니다. 파라미터 순서와 종류는 아래와 같습니다.
- container_name: Docker container 이름 (사용자가 지정)
- image_version: 사용할 Docker image 버전. 가장 최신 버전인 "latest"를 지정합니다.
- port: Jupyter notebook을 사용할 포트 번호 (사용자가 지정). 사용 중인 다른 Jupyter가 없는 경우 8888을 추천합니다.
- workspace_path: 환경에 연결할 작업 path로, Jupyter notebook 환경에서 작업하는 파일이 저장되는 로컬 PC 경로입니다.
- resource_limit: Docker container에 자원 제한을 걸 지 여부로, Optional입니다. 입력하지 않으면 시험장 환경인 2C CPU에 4G Memory로 제한이 걸리고, unlimited를 지정하면 자원 제한을 걸지 않습니다.

e.g.
```
./container-init.sh test-adp latest 8889 /Users/hyunjoong/Documents/workspace/notebook_base/adp unlimited
```

(3) 실행이 완료되면 아래와 같이 로그가 발생하면서 Jupyter notebook 실행되는 것을 볼 수 있습니다. 로그 아래쪽에 있는 http://({container ID} or 127.0.0.1):8888/?token= 뒤에 있는 값이 로그인에 필요한 토큰입니다.

<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/adp-docker/notebook-log.png" alt="notebook-log">
<br><br> Jupyter notebook 접속은 웹 브라우저에 접속한 후 localhost:{사용자 지정 포트}로 할 수 있습니다. <br><br>
<img src ="https://raw.githubusercontent.com/hjben/hjben.github.io/master/_img/adp-docker/notebook-main.png" alt="notebook-main">

<br> 로그인 화면에서 노트북 비밀번호 설정도 가능하지만, 컴퓨터 재부팅 등으로 Container가 중지되면 원상복구되어 다시 토큰 로그인을 해야 합니다.

(4) 다른 CLI를 열고 _./container-remove.sh_ 명령어를 수행하여 실행 중인 Jupyter notebook을 중지시킬 수 있습니다. 명령어 수행 시 삭제할 Container 이름인 container_name 파라미터를 지정해야 합니다.

### 3-2. Generate docker container (Manual)
(1) CLI를 열고, 아래의 두 Docker 명령어를 직접 수행하여 Container를 생성할 수 있습니다. 중괄호로 표시된 Parameter들은 2-1에서 container-init.sh 파일의 Parameter와 동일합니다.

```
docker run --name {container_name} -d -t -p {port}:8888 -v {workspace_path}:/workspace/Jupyter hjben/adp-python:{image_version}
docker exec -it {container_name} bash -c "jupyter-notebook --ip=0.0.0.0"
```

(2) 실행이 완료되면 로그가 발생하면서 Jupyter notebook 실행되는 것을 볼 수 있습니다. 로그 아래쪽에 있는 http://({container ID} or 127.0.0.1):8888/?token= 뒤에 있는 값이 로그인에 필요한 토큰입니다.

(3) 다른 CLI를 열고, 아래 Docker 명령어를 직접 수행하여 실행 중인 Jupyter notebook을 중지시킬 수 있습니다.

```
docker rm -f {container_name}
```

# Jupyter notebook information
### 1. OS & Python Version
- 각각 Ubuntu 16.04, Python 3.7.4로 시험장 버전과 동일합니다.

### 2. Font
- 한글 폰트로 맑은 고딕(malgun.ttf)이 설치되어 있으며, 설치 경로는 /usr/share/fonts/truetype/ 입니다.

### 3. Data Path
- Container 내부의 기본 경로는 시험장의 dataset 경로에 맞추어 /workspace/Jupyter 입니다.
- 로컬의 {workspace_path} 경로가 Container 내부의 /workspace/Jupyter 경로와 동기화됩니다. 로컬의 {workspace_path} 경로에 파일을 복사하면 Jupyter notebook 내 /workspace/Jupyter 경로 하위에 들어가며, 반대 케이스도 가능합니다.

### 4. Python Packages
- Python 패키지 버전은 31회 ADP실기 시험안내를 참고하였으며, 이후 패키지가 추가/변경되는 경우 업데이트 예정입니다.
- 패키지는 가능한 비슷한 버전으로 구성했지만 실제 시험장의 환경과는 차이가 있으며 일부 기능이 동작하지 않을 수도 있습니다.
- 다음은 시험장에는 있지만, 현재 시점에서 설치가 불가능하여 Docker image에 없는 패키지 목록입니다.
```
mkl-fft==1.0.11
mkl-random==1.0.2
PyQt5==5.12
spyder==3.3.3
```

- 또한, 다음은 시험장 환경과 버전 차이가 있는 패키지 목록입니다. 딱 맞는 버전 설치가 불가능했거나, Dependency에 따라 버전이 변경된 패키지들입니다.
```
absl-py==1.0.0
attrs==19.3.0
blis==0.7.4
catboost==1.2
graphviz==0.8.4
grpcio==1.24.3
importlib-metadata==0.23
Jinja2==2.11.1
joblib==1.0.0
jupyter-client==6.1.5
Keras-Preprocessing==1.1.1
lief==0.11.0
llvmlite==0.30.0
matplotlib==3.2.0
mxnet==1.9.0
nbformat==5.0.2
networkx==2.4
pandas-profiling==3.0.0
phik==0.11.1
preshed==3.0.2
protobuf==3.9.2
pycodestyle==2.6.0
pyflakes==2.2.0
pygame==2.0.2
Pygments==2.4.1
requests==2.24.0
seaborn==0.10.1
smart-open==5.2.1
spacy==3.1.0
srsly==2.4.3
tensorboard==2.10.0
tensorflow==2.10.0
tensorflow-estimator==2.10.0
thinc==8.0.7
torch==1.8.0
tqdm==4.48.2
wasabi==0.8.1
Werkzeug==1.0.1
zope.interface==4.7.2
```

- 아래 패키지들은 시험 공부 중 필요에 의해 추가 설치한 패키지로, Docker image에 포함되어 있습니다. 2023.11.17 기준 최신 버전이 설치되어 있습니다.
```
stemgraphic
kendall-w
gower
hdbscan
scikit-learn-extra
sklearn-som
minisom
factor_analyzer
nltk
wordcloud
pydotplus
bioinfokit
fisher-test-python
```
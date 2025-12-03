# 주피터 사용하기

### pipenv를 이용한 주피터 환경 구축

### 초기 환경 구축

1. 신규 디렉토리 생성

   ```bash
   mkdir jupyter ; cd jupyter
   ```

2. pipenv로 python 세팅

   ```bash
   pipenv --python 3.9
   ```

3. pipenv로 패키지 설치

   ```bash
   pipenv install jupyter pandas seaborn torch sklearn metakernel zmq pysqlite3
   ```


### 터널링 환경 설정

* 인터넷 검색 요청 : Sock5 dynamic tunneling 추천

### 주피터 실행

1. 주피터 노트북 실행

   ```bash
   pipenv run jupyter notebook --no-browser --ip 0.0.0.0 
   ```

   >Launching subshell in virtual environment...\
   >. /share/geonmo/.local/share/virtualenvs/pipenv_test01-rTDE4siu/bin/activate\
   >[geonmo@bio-ui7 pipenv_test01]$  . /share/geonmo/.local/share/virtualenvs/pipenv_test01-rTDE4siu/bin/activate\
   >(pipenv_test01) [geonmo@bio-ui7 pipenv_test01]$ pip freeze\
   >numpy==1.21.6\
   >(pipenv_test01) [geonmo@bio-ui7 pipenv_test01]$ python --version
   >Python 3.6.8

   


# 분석 환경 구축
## ROOT 및 Geant4 사용자 환경 설정 안내

### geant4env

* ROOT 및 Geant4를 이용하시기 위해서는 ```geant4env``` 명령을 사용하시면 됩니다. 해당 설정을 입력하면 ROOT 및 Geant4 등 기초적인 LCG 프로그램들에 대한 환경변수 설정이 완료됩니다.
  * 위 명령은 LCG_107 Views를 사용하기 때문에 사용자 본인이 원하는 view 가 있다면 해당 명령을 $HOME/.bashrc 파일에서 재정의하여 사용하시면 됩니다.
  * 자세한 동작 원리는 ```/etc/profile.d/ngnu.sh``` 파일을 참고해 주시기 바랍니다.

## DUNE 사용자들을 위한 환경 설정 방법 안내

### dune_el7과 dune_setup7

* 차세대중성미자 클러스터는 Almalinux9(AL9)으로 구축되어 있습니다. DUNE 실험의 경우, 주요 시스템 패키지들이 Scientic Linux 7(SL7)을 기준으로 하여 작동하고 아직 AL9을 지원하지 않는 경우가 있어서 OS 버전을 변경하면서 사용할 필요가 있습니다.
* ```dune_el7``` 명령은 쉘을 SL7 환경으로 전환하는 명령입니다. 해당 명령 사용 이후로는 SL7 환경처럼 이용이 가능합니다.
  * HTCondor 배치 시스템에서는 container universe를 이용하여 SL7 환경에서의 프로그램 실행이 가능합니다. 해당 내용은 ```HTCondor 이용``` 부분에서 자세하게 다루겠습니다.
* ```dune_setup7``` 명령은 SL7 환경에서 사용해야 하며, ```setup_dune.sh``` 환경변수 설정파일을 로드합니다. 해당 설정이 되어야 ```larsoft``` 등을 사용하실 수 있습니다.
  * 이후 ```setup dunesw $DUNELAR_VERSION -q $DUNELAR_QUALIFIER``` 등으로 전용 소프트웨어 갱신이 가능합니다.
* 자세한 동작 원리는 ```/etc/profile.d/ngnu.sh``` 파일을 참고해 주시기 바랍니다.

### dune_setup9와 dune_spacksetup

* DUNE 실험의 경우, AL9에서 완전한 패키지 구성이 되어 있지 않으나 ```spack``` 을 통한 일부 SW들을 사용하실 수 있습니다. ```dune_setup9``` 명령으로 ```spack``` 환경을 구성하실 수 있으며, 기본 presetup 설정은 ```dune_spacksetup``` 설정으로 불러오실 수 있습니다. 
* 자세한 동작 원리는 ```/etc/profile.d/ngnu.sh``` 파일을 참고해 주시기 바랍니다.



## Python 환경 구축 안내

### pipenv를 이용한 환경 구축

* pipenv는 파이썬 버전 및 virtualenv 환경 그리고 패키지 관리까지 포함된 공식 python 환경 시스템입니다.
* 디렉토리 별로 프로젝트를 구분하기 때문에 새로운 프로젝트는 새로운 디렉토리에서 환경을 구축해야 합니다.

### 초기 환경 구축
1. 신규 디렉토리 생성
    ```bash
    mkdir pipenv_test01
    ```

2. pipenv 설치

    ```bash
    pip3 install --user pipenv
    ```

3. pyenv 설치

    ```bash
    curl https://pyenv.run | bash
    
    echo "export PYENV_ROOT=\"$HOME/.pyenv\"" >> $HOME/.bashrc
    eval "$(pyenv init -)" >> $HOME/.bashrc
    
    source $HOME/.bashrc
    ```

4. pyenv 에서 원하는 python 버전 설치

    ```bash
    pyenv install 3.7.13
    ```

    

5. pipenv로 python 세팅

    ```bash
    pipenv --python 3.7.13
    ```
    >Creating a virtualenv for this project...\
    Pipfile: /share/geonmo/01management/condor_check/pipenv_test01/Pipfile\
    Using /share/geonmo/.pyenv/versions/3.7.13/bin/python3.7m (3.7.13) to create virtualenv...                                                                    
    ⠏ Creating virtual environment...created virtual environment CPython3.7.13.final.0-64 in 15781ms                                                              
    creator CPython3Posix(dest=/share/geonmo/.local/share/virtualenvs/pipenv_test01-rTDE4siu, clear=False, no_vcs_ignore=False, global=False)              
    seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/share/geonmo/.local/share/virtualenv)               
    added seed packages: pip==22.0.4, setuptools==62.1.0, wheel==0.37.1
    activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator\
    ✔ Successfully created virtual environment!
    Virtualenv location: /share/geonmo/.local/share/virtualenvs/pipenv_test01-rTDE4siu                                                                          
    Creating a Pipfile for this project...

6. py

7. pipenv로 패키지 설치

    ```bash
    #pipenv install <Package>
    pipenv install numpy
    ```
    >Installing numpy...\
    Adding numpy to Pipfile's [packages]...\
    ✔ Installation Succeeded\
    Pipfile.lock not found, creating...\
    Locking [dev-packages] dependencies...\
    Locking [packages] dependencies...\
    Building requirements...\
    Resolving dependencies...\
    ✔ Success!\
    Updated Pipfile.lock (2cfc5e)!\
    Installing dependencies from Pipfile.lock (2cfc5e)...\
    🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0  00:00:00\
    To activate this project's virtualenv, run pipenv shell.\
    Alternatively, run a command inside the virtualenv with pipenv run.

8. pipenv shell로 해당 환경 접속
    ```bash
    pipenv shell 
    ```
    >Launching subshell in virtual environment...\
    . /share/geonmo/.local/share/virtualenvs/pipenv_test01-rTDE4siu/bin/activate\
    [geonmo@bio-ui7 pipenv_test01]$  . /share/geonmo/.local/share/virtualenvs/pipenv_test01-rTDE4siu/bin/activate\
    (pipenv_test01) [geonmo@bio-ui7 pipenv_test01]$ pip freeze\
    numpy==1.21.6\
    (pipenv_test01) [geonmo@bio-ui7 pipenv_test01]$ python --version
    Python 3.6.8

9. pipenv run으로 외부에서 명령어 실행 가능
    ```bash
    pipenv run pip freeze
    ```


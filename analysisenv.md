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

### Conda를 이용한 환경 구축

* anaconda 설정에 대한 내용은 추후 추가될 예정입니다.
* 다만, conda 명령 그대로보다는 mamba 등을 사용해주시기를 요청드립니다.

 ### Poetry + Apptainer를 이용한 환경 구축

#### 1. 개요

본 문서는 Python 프로젝트에서 **Poetry**를 이용해 의존성을 관리하고, 실행 환경은 **Apptainer(Singularity)** 컨테이너로 고정하여 HPC·클러스터 환경에서도 **완전한 재현성(reproducibility)** 을 보장하는 개발 및 배포 흐름(workflow)을 설명한다.

Poetry는 Python 패키지 의존성과 가상환경을 관리하며, Apptainer는 시스템 라이브러리, OS 의존성, Python 런타임 등을 캡슐화한다.
 두 도구를 조합하면:

- Python dependencies → Poetry (`pyproject.toml`, `poetry.lock`)
- OS dependencies → Apptainer (base OS 이미지)
- 실행 환경 전부가 컨테이너 내부에서 재현 가능
- HPC 환경에서도 모듈 충돌 없이 안전하게 실행 가능

이라는 장점을 얻을 수 있다.

#### 2. 전체 구조 개요

```
project/
 ├─ pyproject.toml       # Poetry 의존성 정의
 ├─ poetry.lock          # Poetry lock
 ├─ app/                 # Python 코드
 ├─ apptainer.def        # Apptainer 컨테이너 정의 파일
 └─ run.sh               # 실행 스크립트(옵션)
```

- **Poetry는 Python 라벨의 종속성만 관리**
- **시스템 종속성(libpq-dev, ffmpeg 등)은 Apptainer 정의 파일(apptainer.def)에서 설치**
- Poetry 환경은 컨테이너 *안에서* 생성하며, 외부 시스템에는 오염 없음

-----

#### 3. Apptainer 기본 사용법 (요약)

##### 3.1. 이미지 빌드

```
apptainer build myimage.sif apptainer.def
```

##### 3.2. 이미지 실행

```
apptainer exec myimage.sif python app/main.py
```

##### 3.3. 이미지 쉘 진입

```
apptainer shell myimage.sif
```

----

#### 4. 프로젝트 준비

##### 4.1. Poetry 설정

프로젝트 루트에서:

```
poetry init
# or 기존 프로젝트는 아래만 수행
poetry install
```

`pyproject.toml` 에 필요한 패키지를 추가:

```
poetry add numpy pandas requests
poetry add --group dev pytest black
```

---

#### 5. Apptainer 정의 파일 작성 (apptainer.def)

아래는 **Poetry + minimal Debian 기반** 예시.

```
Bootstrap: docker
From: python:3.11-slim

%labels
    Author YourName
    Description "Python+Poetry container for HPC"

%post
    # 기본 패키지
    apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        git \
        curl \
        libpq-dev \
        && rm -rf /var/lib/apt/lists/*

    # Poetry 설치
    curl -sSL https://install.python-poetry.org | python3 -
    ln -s /root/.local/bin/poetry /usr/local/bin/poetry

    # 프로젝트 복사
    mkdir -p /app
    cp -r $APPTAINER_BUILDDEF_DIR/* /app
    cd /app

    # Poetry 가상환경을 컨테이너 내부에 생성
    poetry config virtualenvs.in-project true
    poetry install --no-interaction --no-root

%environment
    export PATH="/app/.venv/bin:$PATH"
    export PYTHONPATH="/app:$PYTHONPATH"

%runscript
    exec poetry run python app/main.py "$@"
```

------

## 6. 컨테이너 빌드

HPC 환경에서는 싱귤래리티가 root-free로 빌드될 수 있음:

```
apptainer build myproj.sif apptainer.def
```

빌드 성공 후 생성된 `myproj.sif`는 해당 Python 환경을 완전히 포함한다.

------

## 7. 컨테이너 실행 방식

### 7.1 직접 실행 (프로그램 자동 실행)

```
apptainer run myproj.sif
```

### 7.2 특정 Python 파일 실행

```
apptainer exec myproj.sif poetry run python app/train.py
```

### 7.3 컨테이너 쉘 진입

```
apptainer shell myproj.sif
```

------

## 8. 외부 디렉토리 마운트 (HPC에서 매우 중요)

아래는 `/scratch` 같은 외부 스토리지를 컨테이너에 마운트하는 예:

```
apptainer exec --bind /scratch:/scratch myproj.sif python app/main.py
```

또는 여러 디렉토리:

```
apptainer exec --bind /data,/scratch myproj.sif python ...
```

------

## 9. HTCondor 배치 스크립트 예시

`python_apptainer_test.sub`:

```
Universe = container
container_image = /ngnu/container_images/ngnu_test01/myproj.sif
JobBatchName            = Python_Apptainer_Test
Log = htcondor.log
Output = job_log/$(JobBatchName)/$(Cluster)/$(Process).out
Error = job_log/$(JobBatchName)/$(Cluster)/$(Process).err

should_transfer_files  = YES
when_to_transfer_output = ON_EXIT

### sh> run.script $1 $2 $3
Executable = python_analysis_code.sh
Arguments = "$(PARAM1) $(PARAM2) $(PARAM3)"

### 전송할 파일 목록. 참고로 Executable은 input files 목록에 넣을 필요 없음.
transfer_input_files = analysis_code.py
transfer_output_files = output.root
transfer_output_remaps = "output.root=results/$(Cluster)/output_$(Process).root"

### 컨테이너 내부에서 연결할 호스트의 디렉토리를 설정. 반드시 string형태여야 함. " "로 묶인 문장
+SingularityBind="/cvmfs,/ngnu,/etc/profile.d/ngnu.sh"


### 필요한 자원 정보
Request_cpus = 1
Request_memory = 5GB
Request_disk = 1GB

### 작업 제출.(https://htcondor.readthedocs.io/en/latest/users-manual/submitting-a-job.html)
Queue 5 PARAM1, PARAM2, PARAM3 from analysis_path.txt
## Queue 1 A, B from a.txt
## Queue 1 in (A, B)
```

```python_analysis_code.sh```:

```bash
#!/bin/bash
cd /app
poetry run python analysis_code.py
```

제출:

```
condor_submit python_apptainer_test.sub
```

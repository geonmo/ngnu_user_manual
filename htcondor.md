# HTCondor를 이용한 배치 작업 실행 
UI 서버는 많은 사용자들이 같이 이용하는 서버입니다. 따라서, 컴퓨팅 자원을 많이 사용하는 분석 코드의 경우에는 UI보다는 WN에서 실행하는 것을 추천 드립니다.
### 작업 확인(condor_q)
* 제출된 작업의 상태 및 실행 여부를 확인할 수 있는 명령어입니다.
1. condor_q 명령어로 본인의 작업 정보를 확인 가능합니다.
   ```bash
   condor_q
   ````
   > -- Schedd: nu-ui01.gsdc.internal : <10.1.45.234:9618?... @ 12/02/25 04:36:26
   > OWNER BATCH_NAME      SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS
   > 
   > Total for query: 0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended 
   > Total for ngnu_test01: 0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended 
   > Total for all users: 0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
   * -allusers(-all) 옵션으로 다른 사용자의 작업을 확인할 수 있습니다.
   * -global 옵션으로 다른 그룹의 작업을 확인할 수 있습니다. 
   * -long(-l) 옵션으로 특정 작업의 자세한 ClassAds 정보를 확인할 수 있습니다.
   * -better-analyze 옵션으로 hold된 작업이나 매칭되는 장비들을 확인할 수 있습니다.
   * -r 옵션은 현재 실행되고 있는 장비를 알 수 있습니다.

### 서버 상태 확인(condor_status)
* 작동하는 서버들의 작업 할당 여부 등을 확인할 수 있는 명령어입니다.
1. condor_status 명령어로 서버들의 작업 할당 상황을 확인 가능합니다.
   ```bash
   condor_status
   ```
   >   Name                        OpSys      Arch   State     Activity LoadAv Mem     ActvtyTime
   >
   >   slot1@nu-wn01.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 596190  6+22:20:08
   >   slot1@nu-wn02.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 596190  6+22:20:18
   >   slot1@nu-wn03.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 644571  0+21:54:36
   >   slot1@nu-wn04.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 644571  0+21:54:31
   >   slot1@nu-wn05.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 644571  0+21:54:41
   >   slot1@nu-wn06.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 644571  0+18:44:32
   >   slot1@nu-wn07.gsdc.internal LINUX      X86_64 Unclaimed Idle      0.000 644571  0+21:54:35
   >
   >                  Total Owner Claimed Unclaimed Matched Preempting  Drain Backfill BkIdle
   >
   >     X86_64/LINUX     7     0       0         7       0          0      0        0      0
   >
   >            Total     7     0       0         7       0          0      0        0      0
   * State와 Activity에 따라 머신의 상태 확인 가능
     * Unclaimed - Idle : 작업이 할당 안된 슬롯(동적 슬롯이 아닌 경우에는 남은 자원)
     * Claimed - Busy : 작업이 사용 중인 슬롯
   * -server 옵션으로 각 슬롯에 할당된 자원의 크기를 알 수 있습니다.
   * -long(-l) 옵션으로 보다 자세한 ClassAds 정보를 확인할 수 있씁니다.
   
### 작업 우선권 확인(condor_userprio)
* 여러 사용자들의 작업이 제출된 경우 작업의 우선권대로 작업이 실행됩니다. 이에 따라, 같은 그룹 내에서 우선권이 높을수록 더 많은 슬롯이 배정됩니다. 이를 확인하기 위한 명령어입니다.
1. condor_userprio로 우선권 및 제출된 작업 수를 쉽게 파악할 수 있습니다.
   ```bash
   condor_userprio
   ```

### 작업 제출 방법(condor_submit)

#### (1) 일반 배치작업(vanilla universe)

* bash 스크립트나 분석 프로그램을 별도의 사전 환경 설정 없이 배치 작업으로 수행하는 경우 ```vanilla universe```를 사용합니다. 

* HTCondor 작업을 제출하기 위한 JDS(작업제출명세 파일/.sub 또는 .jds)을 작성합니다.
  ```test.sub``` 기본 양식 (vanilla 유니버스)

```
Universe = vanilla
JobBatchName            = Geant4_Test01
Log = htcondor.log
Output = job_log/$(Cluster)/$(Process).out
Error = job_log/$(Cluster)/$(Process).err

should_transfer_files  = YES
when_to_transfer_output = ON_EXIT

### sh> run.script $1 $2 $3
Executable = run_geant4.sh
#Arguments = "$1 $2 $3"

### 전송할 파일 목록
transfer_input_files =
transfer_output_files = ccal.root
transfer_output_remaps = "ccal.root=results/$(Cluster)/ccal_$(Process).root"

### 필요한 자원 정보
Request_cpus = 1
Request_memory = 5GB
Request_disk = 1GB

### 작업 제출.(https://htcondor.readthedocs.io/en/latest/users-manual/submitting-a-job.html)
Queue 1
## Queue 1 A, B from a.txt
## Queue 1 in (A, B)
```

* 위에서 작성한 작업명세파일을 **condor_submit**으로 제출합니다.

```bash
condor_submit test.sub
```

#### (2) 컨테이너 환경에서의 작업 수행 (container universe)

* Almalinux9 이 아닌 OS 환경에서는 컨테이너 유니버스를 이용하여 작업을 수행하여야 합니다.
* 컨테이너 유니버스의 기본 양식은 아래와 같습니다.

```
Universe = container
container_image = /cvmfs/singularity.opensciencegrid.org/fermilab/fnal-dev-sl7:latest
JobBatchName            = DUNE_SL7_Test
Log = htcondor.log
Output = job_log/$(JobBatchName)/$(Cluster)/$(Process).out
Error = job_log/$(JobBatchName)/$(Cluster)/$(Process).err

should_transfer_files  = YES
when_to_transfer_output = ON_EXIT


### sh> run.script $1 $2 $3
Executable = run_dune_sl7.sh
#Arguments = "$1 $2 $3"

### 전송할 파일 목록. 참고로 Executable은 input files 목록에 넣을 필요 없음.
transfer_input_files = 
transfer_output_files = file.png
transfer_output_remaps = "file.png=results/$(Cluster)/file_$(Process).png"

### 컨테이너 내부에서 연결할 호스트의 디렉토리를 설정. 반드시 string형태여야 함. " "로 묶인 문장
requirements = ( HasSingularity == true )
+SingularityBind="/cvmfs,/ngnu,/etc/profile.d/ngnu.sh"


### 필요한 자원 정보
Request_cpus = 1
Request_memory = 5GB
Request_disk = 1GB

### 작업 제출.(https://htcondor.readthedocs.io/en/latest/users-manual/submitting-a-job.html)
Queue 1
## Queue 1 A, B from a.txt
## Queue 1 in (A, B)
```

#### (3) Executable용 bash 스크립트 예제

* 분석 코드를 바로 실행할 수 있는 상태라면 좋겠으나 대개 Framework 등의 라이브러리  혹은 모듈 경로를 설정하는 경우가 많으므로 bash 스크립트를 이용하여 환경 설정 후 분석 코드를 돌리는 식으로 작성합니다.
* executable용 스크립트는 반드시 실행 권한을 부여해야 합니다. (```chmod +x run.sh```)
* Geant4 사용 예제

```bash
#!/bin/bash
### Geant4 환경설정 후 실행
source /cvmfs/sft.cern.ch/lcg/views/LCG_107/x86_64-el9-gcc11-opt/setup.sh

## 임시 디렉토리로 복사하는 경우는 아래와 같습니다. 일반적인 경우라면 $HOME으로 이동하여 분석 코드를 실행해도 무방합니다.
echo "HOME is $HOME"
cp -rf /ngnu/home/ngnu_test01/composite_calorimeter $_CONDOR_SCRATCH_DIR
whoami
cd composite_calorimeter
source envExample.sh
./build/CompositeCalorimeter test.g4mac
## 결과파일은 반드시 임시 디렉토리에서의 상대경로로 맞쳐야 합니다.
cp ccal.root $_CONDOR_SCRATCH_DIR
RESULT=$?
exit $RESULT
## 분석코드의 반환코드를 리턴해줘야 합니다.

```

* SL7용 예제 (SL7 전환용)

```bash
#!/bin/bash
### DUNE SL7용 설정. container universe라면 이미 OS는 SL7이니 SL7용 환경변수 설정만 하면 됩니다.
export LANG=en_US.UTF-8 && source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
setup root v6_28_12 -q e26:p3915:prof # sets up root for you  

## 분석 코드 실행
root -l -q $ROOTSYS/tutorials/io/file.C
exit $?
## 분석코드의 반환코드를 리턴해줘야 합니다.
```






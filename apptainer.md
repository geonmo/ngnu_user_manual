# Apptainer(구 Singularity)를 이용한 환경설정

Apptainer를 이용한 환경을 구축하는 방법에 대한 안내드립니다.

## Apptainer와 Docker 차이

* Apptainer는 관리자 권한 없이 사용가능 <-> Docker는 관리자 권한의 docker 데몬 필요
* Apptainer는 컨테이너 내부에서 root로 승급 불가 <-> Docker 가능
  * 관리자 권한이 필요한 패키지 설치는 불가능
  * pipenv 같은 python 환경설정은 사용자 설치 방법으로 설치 가능

## 이용방법

1.  아래 명령어를 참고하여 컨테이너 이미지를 다운로드 합니다.

    ```bash
    ### Container Image download from docker hub,
    apptainer pull deepvariant_1.4.0.sif docker://google/deepvariant:1.4.0
    ### Checking sif image file,
    ls -l deepvariant_1.4.0.sif
    ```

    > \[geonmo@bio-ui7 \~]$ ls -lh deepvariant\_1.4.0.sif
    >
    > \-rwxr-x---. 1 geonmo geonmo 2.1G Jun 15 14:43 deepvariant\_1.4.0.sif
    
2.  컨테이너에 접속하여 작업이 원하는대로 수행되는지 확인합니다.

    ```bash
    ### UI서버에서는 기본 설정이 되어 있어 간단하게 접속이 가능합니다.
    apptainer shell deepvariant_1.4.0.sif
    ### 컨테이너의 OS정보를 확인합니다.
    cat /etc/lsb-release
    ### 분석 코드를 실행해봅니다.
    #### tensorflow 코드 작성
    cat > tf_test.py
    import tensorflow as tf
    mnist = tf.keras.datasets.mnist
    
    (x_train, y_train), (x_test, y_test) = mnist.load_data()
    x_train, x_test = x_train / 255.0, x_test / 255.0
    model = tf.keras.models.Sequential([
      tf.keras.layers.Flatten(input_shape=(28, 28)),
      tf.keras.layers.Dense(128, activation='relu'),
      tf.keras.layers.Dropout(0.2),
      tf.keras.layers.Dense(10, activation='softmax')
    ])
    
    model.compile(optimizer='adam',
                  loss='sparse_categorical_crossentropy',
                  metrics=['accuracy'])
    model.fit(x_train, y_train, epochs=5)
    
    model.evaluate(x_test,  y_test, verbose=2)
    ## \^C
    ## 컨트롤 + c
    #### tensorflow 코드 실행
    python tf_test.py
    ```

    > Apptainer> cat /etc/lsb-release
    >
    > DISTRIB\_ID=Ubuntu
    >
    > DISTRIB\_RELEASE=20.04
    >
    > DISTRIB\_CODENAME=focal
    >
    > DISTRIB\_DESCRIPTION="Ubuntu 20.04.4 LTS"
    >
    > Apptainer> cat > tf\_test.py
    >
    > import tensorflow as tf
    >
    > mnist = tf.keras.datasets.mnist
    >
    > (x\_train, y\_train), (x\_test, y\_test) = mnist.load\_data()
    >
    > x\_train, x\_test = x\_train / 255.0, x\_test / 255.0
    >
    > model = tf.keras.models.Sequential(\[
    >
    > tf.keras.layers.Flatten(input\_shape=(28, 28)),
    >
    > tf.keras.layers.Dense(128, activation='relu'),
    >
    > tf.keras.layers.Dropout(0.2),
    >
    > tf.keras.layers.Dense(10, activation='softmax')
    >
    > ])
    >
    > model.compile(optimizer='adam',
    >
    >  loss='sparse\_categorical\_crossentropy',
    >
    >  metrics=\['accuracy'])
    >
    > model.fit(x\_train, y\_train, epochs=5)
    >
    > model.evaluate(x\_test, y\_test, verbose=2)
    >
    > ^C
    >
    > Apptainer> python tf\_test.py
    >
    > 2022-06-15 14:57:32.959817: I tensorflow/core/platform/cpu\_feature\_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations: AVX2 FMA
    >
    > To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    >
    > 2022-06-15 14:57:32.964689: I tensorflow/core/common\_runtime/process\_util.cc:146] Creating new thread pool with default inter op setting: 2. Tune using inter\_op\_parallelism\_threads for best performance.
    >
    > Epoch 1/5 1875/1875 \[==============================] - 24s 13ms/step - loss: 0.2973 - accuracy: 0.9125
    >
    > Epoch 2/5 1875/1875 \[==============================] - 18s 10ms/step - loss: 0.1389 - accuracy: 0.9578
    >
    > Epoch 3/5 1875/1875 \[==============================] - 20s 11ms/step - loss: 0.1064 - accuracy: 0.9674
    >
    > Epoch 4/5 1875/1875 \[==============================] - 22s 12ms/step - loss: 0.0874 - accuracy: 0.9727
    >
    > Epoch 5/5 1875/1875 \[==============================] - 21s 11ms/step - loss: 0.0746 - accuracy: 0.9772
    >
    > 
    >
    > 313/313 - 1s - loss: 0.0748 - accuracy: 0.9764 - 1s/epoch - 5ms/step
    
3.  작업 제출 명세파일을 작성합니다. on `submit_Apptainer.sub`,

    ```bash
    JobBatchName  = Apptainer_$(Cluster)
    executable = Apptainer_test.sh
    
    universe   = container
    container_image = /ngnu/container_images/ngnu_test01/deepvariant_1.4.0.sif
    
    requirements = ( HasSingularity == true )
    getenv     = True
    should_transfer_files = YES
    when_to_transfer_output = ON_EXIT
    output = job_$(Process).out
    error = job_$(Process).err
    log = job_$(Process).log
    
    transfer_input_files = tf_test.py
    #transfer_output_files =
    
    +SingularityrBind = "/ngnu, /var/lib/condor"
    
    notification = Error
    #notify_user = <YOUR EMAIL ADDRESS>
    
    queue 1
    ```
    
4.  실행 파일을 작성합니다. on `Apptainer_test.sh`,

    ```
    #!/bin/bash
    echo $(hostname)
    cat /etc/lsb-release
    
    python tf_test.py
    ```
    
5. condor 작업을 제출합니다. 
   
   ```bash
   condor_submit submit_Apptainer.sub
   ```






# 주피터 사용하기

### poetry를 이용한 주피터 환경 구축

### 초기 환경 구축

1. 신규 디렉토리 생성

   ```bash
   mkdir run_jupyter_nb ; cd run_jupyter_nb
   ```

2. poetry로 python 세팅

   ```bash
   poetry init --no-interaction
   ```

3. poetry로 패키지 설치

   ```bash
   ## Python 3.9 (AlmaLinux9 기본)
   ### torch, sklearn 등 설치 불가
   poetry add jupyter pandas seaborn metakernel zmq pysqlite3
   
   ## ML 권장버전인 Python 3.10으로 변경하려면 pyenv 사용
   pyenv install 3.10
   pyenv local 3.10.19
   poetry use 3.10.19
   ## pyproject.toml 파일에서 requires-python 버전을 >=3.10,<3.11 로 변경
   poetry add jupyter pandas seaborn torch scikit-learn metakernel zmq pysqlite3
   ```


### 터널링 환경 설정

* 인터넷 검색 요청 : Sock5 dynamic tunneling 추천

### 주피터 실행

1. 주피터 노트북 실행

   ```bash
   poetry run jupyter notebook --no-browser --ip 0.0.0.0
   ```

   >[I 2025-12-03 09:46:37.877 ServerApp] Extension package jupyter_lsp took 0.1707s to import
   >[I 2025-12-03 09:46:38.225 ServerApp] jupyter_lsp | extension was successfully linked.
   >[I 2025-12-03 09:46:38.231 ServerApp] jupyter_server_terminals | extension was successfully linked.
   >[I 2025-12-03 09:46:38.237 ServerApp] jupyterlab | extension was successfully linked.
   >[I 2025-12-03 09:46:38.242 ServerApp] notebook | extension was successfully linked.
   >[I 2025-12-03 09:46:40.718 ServerApp] notebook_shim | extension was successfully linked.
   >[I 2025-12-03 09:46:40.847 ServerApp] notebook_shim | extension was successfully loaded.
   >[I 2025-12-03 09:46:40.850 ServerApp] jupyter_lsp | extension was successfully loaded.
   >[I 2025-12-03 09:46:40.851 ServerApp] jupyter_server_terminals | extension was successfully loaded.
   >[I 2025-12-03 09:46:40.860 LabApp] JupyterLab extension loaded from /ngnu/home/ngnu_test01/.cache/pypoetry/virtualenvs/run-jupyter-nb-39-pN1PDl9U-py3.9/lib/python3.9/site-packages/jupyterlab
   >[I 2025-12-03 09:46:40.860 LabApp] JupyterLab application directory is /ngnu/home/ngnu_test01/.cache/pypoetry/virtualenvs/run-jupyter-nb-39-pN1PDl9U-py3.9/share/jupyter/lab
   >[I 2025-12-03 09:46:40.861 LabApp] Extension Manager is 'pypi'.
   >[I 2025-12-03 09:46:41.093 ServerApp] jupyterlab | extension was successfully loaded.
   >[I 2025-12-03 09:46:41.104 ServerApp] notebook | extension was successfully loaded.
   >[I 2025-12-03 09:46:41.105 ServerApp] Serving notebooks from local directory: /ngnu/home/ngnu_test01/python_projects/run_jupyter_nb_39
   >[I 2025-12-03 09:46:41.105 ServerApp] Jupyter Server 2.17.0 is running at:
   >[I 2025-12-03 09:46:41.105 ServerApp] http://nu-ui01.gsdc.internal:8888/tree?token=08545576122329a67a3577bdd5121def2fac8c673fc6165c
   >[I 2025-12-03 09:46:41.105 ServerApp]     http://127.0.0.1:8888/tree?token=08545576122329a67a3577bdd5121def2fac8c673fc6165c
   >[I 2025-12-03 09:46:41.105 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
   >[C 2025-12-03 09:46:41.113 ServerApp] 
   >    
   >    To access the server, open this file in a browser:
   >        file:///ngnu/home/ngnu_test01/.local/share/jupyter/runtime/jpserver-181664-open.html
   >    Or copy and paste one of these URLs:
   >        http://nu-ui01.gsdc.internal:8888/tree?token=08545576122329a67a3577bdd5121def2fac8c673fc6165c
   >        http://127.0.0.1:8888/tree?token=08545576122329a67a3577bdd5121def2fac8c673fc6165c
   >[I 2025-12-03 09:46:41.348 ServerApp] Skipped non-installed server(s): basedpyright, bash-language-server, dockerfile-language-server-nodejs, javascript-typescript-langserver, jedi-language-server, julia-language-server, pyrefly, pyright, python-language-server, python-lsp-server, r-languageserver, sql-language-server, texlab, typescript-language-server, unified-language-server, vscode-css-languageserver-bin, vscode-html-languageserver-bin, vscode-json-languageserver-bin, yaml-language-server
   


# datastream.io
파이썬, Elasticsearch, Kibina를 활용한 실시간 alnomaly detiction을 위한 오픈소스 기반 프레임워크

## Installation
추천 설치 방법은 virtualenv를 이용해서 가상환경을 구축하고 그 가상환경 안에 pip로 다음과 같이 설치하는 방법입니다.

    virtualenv --python=python3 dsio-env
    source dsio-env/bin/activate
    pip install -e git+https://github.com/MentatInnovations/datastream.io#egg=dsio

## Usage
dsio는 커맨드라인이나 파이썬 임포트를 통해서 사용할 수 있습니다.   
#### dsio의 기능
1. built-in Bokeh 서버를 이용해서 시각화 할 수 있다.
2. ElasticSearch에 데이터 스트림을 흐르게 하고 Kibina로 시각화 할 수 있습니다.  

위의 어떤 방식이든 dsio는 각각의 데이터 스트림에 적절한 대시보드를 만들어줍니다.
또한 주피터 노트북에서 dsio를 작동시키면 Bokeh 대시보드에 데이터가 스트리밍 되는 것을 주피터 노트북 안에서 볼 수 있습니다.

![Jupyter](screenshots/jupyter.png?raw=true "DSIO bokeh dashboard")

### Examples

이 섹션에서는 먼저 'examples' 폴더안에서 커맨드를 실행하세요. 만약 이전에 언급했던 방식으로(pip 사용 설치 방식) 설치하셨다면 다음 커맨드를 실행하면 example 폴더 위치로 바로 갈 수 있습니다.

    cd dsio-env/src/dsio/examples

가지고 있는 dataset이나 example용 csv dataset을 사용할 수 있습니다. 만약 데이터 셋이 시간축을 가지면 dsio는 자동으로 그 축을 찾으려고 합니다.
다른 방법으로는 `--timefiels` 파라미터를 통해 직접 사용자가 시간축을 지정해 줄 수 있습니다. 만약 시간축이 존재하지 않으면 dsio는 자동적으로 시계열데이터가 현재부터 1초간격으로 구성된것으로 파악합니다.

    dsio data/cardata_sample.csv

위 커맨드는 cardata라는 샘플 csv를 로드하고 기본 anomaly detection 모델(Gaussian1D)을 각각의 모든 열에 적용합니다. 그리고 나서 적절한 Bokeh 대쉬보드를 생성하고 데이터를 흐르게 할 것입니다. 생성된 대시보드를 가르키는 브라우저의 window는 열려있어야 합니다.

![Bokeh](screenshots/bokeh.png?raw=true "DSIO bokeh dashboard")

여러 데이터셋과 Anomaly 모델을 실험해볼 수 있습니다.

    dsio --detector percentile1d path_to_my_dataset/my_dataset.csv

`--senosrs` 파라메터를 이용하여 특정한 열을 선택할 수 있습니다.  
`--speed` 파라메터는 스트리밍 속도를 올리거나 내릴 수 있습니다.

    dsio --sensors accelerator_pedal_position engine_speed --detector gaussian1d --speed 5 data/cardata_sample.csv

### Elasticsearch & Kibana (optional)

로컬 Elasticsearch에 데이터를 흐르게 하고 Kibana 대쉬보드를 생성하기 위해서는 `--es-uri`와 `--kibina-uri` 파라메터를 이용하면 됩니다.

    dsio --es-uri http://localhost:9200/ --kibana-uri http://localhost:5601/app/kibana data/cardata_sample.csv

만약 localhost와 기본 Kibina와 ElasticSearch 포트를 쓰신다면 그냥 간단하게 다음과 같이 끝내도 됩니다.

    dsio --es data/cardata_sample.csv

![ElasticKibana](screenshots/ek.png?raw=true "DSIO bokeh dashboard")

만약 Elasticsearch와 Kibina 5.x에 접근할 수 없다면 `examples` 폴더의 docker-compose.yaml을 이용하여 시작하면 됩니다. 도커와 docker-compose는 이 작업을 위해서 설치되어야 합니다.

    docker-compose up -d

Elasticsearch와 Kibana가 실행됬는지 체크

    docker-compose ps

끝나면 다음과 같이 종료합니다.

    docker-compose down

docker-compose 커맨드들은 docker-compose.yaml 파일이 있는 곳에서 실행되어야합니다. (e.g. dsio-env/src/dsio/examples)
Keep in mind that docker-compose commands need to be run in the directory where the docker-compose.yaml file resides (e.g. dsio-env/src/dsio/examples)

### Defining your own anomaly detectors

코딩된 Anomaly 모델을 dasio에서 쓸 수 있습니다. AnomalyDetector abstract base 클래스를 상속받고 최소한 train, update & score 메소드를 만들면 쓸 수 있습니다. 99퍼센트의 정확도를 보이는 anomaly detector 모델이 examples 폴더에 있습니다. 코딩된 디텍터들을 포함하는 파이썬 모듈들을 `-modules` 파라메터를 통해 로드하고 모듈안에 있는 디텍터 클래스를 클래스 이름으로 `--detector` 파라메터를 통해서 넣어주면 됩니다.

    dsio  --modules detector.py --detector GreaterThanMaxRolling data/cardata_sample.csv

### Integration with scikit-learn

자연히 우리는 사람들에게 `dasio`와 `sklearn`을 함께 쓸 것을 권합니다.
`sklearn`은 현제 regression, classification, 클러스터링 도구를 제공합니다. 그러나 anomaly detection은 따로 떨어져 분류되어있습니다. 우리는 `AnomalyMinxin` 인터페이스를 통해서 `sklearn`을 쓸 수 있도록 했습니다. `sklearn`을 임포트 한 후 간단하게 인터페이스 안의 함수를 override해서 `dsio`와 함께 쓸 수 있도록 했습니다. 다음 위치에 example 코드를 만들어 놨습니다.

    ./datamstream.io/examples/lof_anomaly_detector.py

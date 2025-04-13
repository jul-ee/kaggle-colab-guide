# 🪧 Google Colab 환경에서 Kaggle 필사를 위한 환경 설정 가이드

이 repository는 Google Colab 환경에서 Kaggle 필사를 효율적으로 진행하기 위한 설정 방법과  
재현 가능한 분석 환경 템플릿을 제공합니다.

세션이 끊기거나 다음 날 다시 Colab에서 작업할 때도, 효율적으로 이어서 필사할 수 있도록 자동화된 코드 템플릿도 함께 포함되어 있습니다.

📌 Kaggle 환경이 낯설거나 Colab에서 Kaggle 데이터를 처음 다뤄보는 분들께 특히 유용합니다.

> [참고]
> 이 가이드는 [Kaggle Titanic](https://www.kaggle.com/c/titanic) 대회의 커널을 기반으로 작성되었습니다.

> 자세한 설명과 작성 배경은 [velog 포스팅](https://velog.io/@jul-ee/Kaggle-구글-코랩에서-캐글-필사-시작하기)에서 확인할 수 있습니다.

<br>

## 사용한 환경 및 구성 방식

- Google Colab 환경
- Kaggle API를 통한 데이터 다운로드
- Google Drive 연동을 통한 파일 관리

> 다양한 방법 중에서 위 방식을 선택한 이유는 재현성과 확장성, 효율성을 고려했기 때문입니다.

<br>

## Google Colab vs Kaggle Notebook 비교

| 항목 | Google Colab | Kaggle Notebook |
|------|---------------|-----------------|
| 자유도 높은 환경 설정 | 가능 (제한 없음) | 제한 있음 |
| Google Drive 연동 | 완전 자동화 가능 | 불가능 |
| GitHub 연동 | 직접 push/pull 가능 | 불편하고 제한적 |
| 오프라인 연동 | Drive로 가능 | 불가능 |
| 세션 시간 | Pro 기준 24시간 | 무료 기준 약 90분 |
| 인터페이스 | Jupyter 스타일, 한글 UI 가능 | 제한적 UI |

> Colab을 사용하면 코드 관리 및 GitHub와의 연동이 훨씬 용이합니다.

<br>

## 세션 종료 후 사라지는 것 vs 유지되는 것

| 항목 | 세션 종료 후 | 설명 |
|------|----------------|------|
| 변수 (train, test, model) | 사라짐 | RAM 초기화 |
| /content/ 내 파일 | 사라짐 | 임시 저장소 |
| Google Drive 파일 | 유지됨 | 클라우드 저장 |
| .ipynb 노트북 파일 | 유지됨 | Drive에 자동 저장됨 |

> 다음 날 이어서 작업할 경우, **초기 설정 셀만 다시 실행**하면 분석을 그대로 이어갈 수 있습니다.

<br>
<br>

## 🔧 Colab 분석 환경 구축 프로세스

1. Kaggle API로 Drive에 kaggle.json 저장  
2. Drive에 데이터 저장  
3. Colab 템플릿을 이용한 반복 사용

kaggle.json을 Drive에 보관하면 매번 새로 업로드할 필요가 없습니다.  
또한, 템플릿을 사용하면 GitHub에도 바로 올릴 수 있는 필사 환경이 구성됩니다.

<br>

## 📁 추천 폴더 구조

```
/drive/MyDrive/kaggle_replica/
└── titanic/
    ├── titanic_analysis.ipynb
    ├── train.csv
    └── test.csv
```

- 대회/프로젝트별 폴더로 정리 (`kaggle_replica/<대회명>/`)
- Google Drive 내에 정리된 상태로 저장되므로 재사용, 공유, 업로드할 때 폴더 단위로 관리가 가능합니다.
- 관련 파일을 같은 경로에 모아두면 코드 실행 시 경로 오류를 줄일 수 있습니다.

<br>

## 🔑 Kaggle API 토큰 발급

1. [Kaggle](https://www.kaggle.com/) 로그인 후, 우측 상단 프로필 > Settings
2. 하단의 API 섹션에서 Create New Token 클릭
3. kaggle.json 파일이 다운로드됨

> ⚠️ kaggle.json은 GitHub에 업로드하지 않도록 주의!
> 
> 
> .gitignore에 반드시 추가하거나 수동으로 제외해야 합니다.
> 

<br>
<br>

## 환경 설정 단계별 코드

환경 자동 설정용 템플릿 노트북인 `.ipynb` 파일은 아래 위치에서 확인하실 수 있습니다.
> 📁 template/kaggle_colab_guide.ipynb

<br>

### [1] Google Drive 마운트 (매번 필요)

```python
from google.colab import drive
drive.mount('/content/drive')
```

실행하면 Google 계정 인증 요청이 뜨고, 승인하면 /content/drive/MyDrive/ 아래에 내 드라이브가 연결됩니다.

![](https://velog.velcdn.com/images/jul-ee/post/83b9f133-c5f1-4018-b9d7-bd1fb9140808/image.png)

<br>

### [2] kaggle.json 설정 (매번 필요)

```python
# Kaggle API 설정
!mkdir -p ~/.kaggle
!cp /content/drive/MyDrive/kaggle_replica/kaggle.json ~/.kaggle/
!chmod 600 ~/.kaggle/kaggle.json
```


<br>

### [3] 데이터 다운로드 (처음 1회만)

```python
!kaggle competitions download -c titanic
!unzip titanic.zip

!mv train.csv /content/drive/MyDrive/kaggle_replica/titanic/
!mv test.csv /content/drive/MyDrive/kaggle_replica/titanic/
!mv gender_submission.csv /content/drive/MyDrive/kaggle_replica/titanic/
```

<br>

### [4] 데이터 로드 (매번 필요)

```python
# 필요한 라이브러리도 함께 불러오기
import pandas as pd

train_path = '/content/drive/MyDrive/kaggle_replica/titanic/train.csv'
test_path = '/content/drive/MyDrive/kaggle_replica/titanic/test.csv'

train_df = pd.read_csv(train_path)
test_df = pd.read_csv(test_path)
```

<br>
<br>

## 💡 사용 팁

- 템플릿 초기 셀에 위 설정 코드를 항상 포함시켜두면, 다음 날에도 쉽게 이어서 작업할 수 있습니다.
- Drive에 저장된 kaggle.json은 다른 대회에도 재사용 가능하므로 경로만 관리하면 됩니다.

<br>

## 캐글 필사를 시작하기 전에

> 분석은 자유롭게 진행하되,
> 
> 
> 각 셀에 주석을 꼼꼼히 달며 “왜 이 작업을 하는지” 설명하는 것이 필사의 핵심입니다.

<br>

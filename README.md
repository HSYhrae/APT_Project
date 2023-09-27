# APT_Project
Jeonse price prediction

## 목적: 부동산 전세가격 예측·전세가율 분석: 적정 전세가율을 활용한 전세사기 예방 웹사이트 구축
- 담당역할: 데이터 수집, 데이터 전처리, EDA

### 프로젝트 수행 방향
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/6a6a1cee-3bcb-4b1a-8623-2303eb2ac5fa)

### 데이터 수집(논문 참고)
- 수집 데이터: 서울 전세 실거래, 서울 매매 실거래, 기준금리, 실업률, 전세수급동향, 지하철역 위치, 초중고등학교 위치, 대학 위치, 인구수, 경기종합지수, 서울 5대 범죄
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/08ac498b-1ee7-4b92-812c-0182a8e8ff4f)

### 데이터 전처리
- 사용한 변수: 계약년도, 계약월, 년월, 자치구명, 층, 건물연수, 건물용도, 임대면적, 인구수, 기준금리, 실업률, 선행종합지수, 동행종합지수, 후행종합지수, 전세수급동향 지수, 공동주택 매매 실거래가격지수, 범죄율, 위도, 경도, 평균 매매가격
지하철역까지의 최소거리, 대학까지의 최소거리, 초중고등학교와의 최소거리, 전세가격
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/c33ea036-f7df-4a0e-a843-1bb61eb3c742)
#### 문제상황: 전세 데이터의 개수는 총 240만개가 넘어 모든 데이터의 좌표를 추출하기에는 비용적, 시간적으로 많은 손해가 발생
- 해결책:
  1. 좌표를 추출할 데이터 수를 줄이는 것이 좋겠다고 판단 -> drop_duplicates 함수를 활용하여 추출할 데이터의 수를 줄임
  2. 줄여진 데이터에 Google Map API의 Geocoding을 활용하여 좌표추출
  3. 기존 전세 데이터와 '주소' 컬럼을 기준으로 병합
  4. 모든 데이터의 좌표를 추출함
 
해결책 2번 소스코드
```
!pip install googlemaps
import googlemaps
from datetime import datetime

apikey = '' # 해당 API 입력

def getLoc(addr):
    gmaps = googlemaps.Client(key='') # 해당 API 입력
    geocode_result = gmaps.geocode(addr)
    n_lat = geocode_result[0]['geometry']['location']['lat']
    n_lng = geocode_result[0]['geometry']['location']['lng']
    loc = {'lat':n_lat, 'lng':n_lng}
    return loc

for i in range(len(unique_rows)):
    find = unique_rows.loc[i, "address"]
    sub_lat = getLoc(find)['lat']
    sub_long = getLoc(find)['lng']
    unique_rows.loc[i, '위도'] = sub_lat
    unique_rows.loc[i, '경도'] = sub_long

# 함수 설정을 통해 구글 지오코드 서비스로 위도 경도에 대한 데이터 가져온 후
# 실거래가 데이터 프레임의 "주소" 컬럼의 주소 값들을 기반으로 위도, 경도 위치 데이터 받은 후
# "위도", "경도"에 컬럼값 위치 데이터 넣기
```
자세한 내용은 '/APT_Project/Preprocessing/Coordinate extraction.ipynb' 참고  

#### 문제상황: 데이터에 결측치가 많음
- 해결책:
1. 기준금리: 23년 2월 이후 현시점인 23년 9월까지 '3.5'로 변동이 없어 '3.5'로 대체
2. 범죄율: 연도별, 자치구별 평균값으로 각 결측치 대체
3. 나머지(실업률, 선행종합지수, 동행종합지수, 후행종합지수, 전세수급동향, 매매 실거래가격지수, 각 최소거리): XGBoost를 활용하여 결측치 대체


### EDA
- 전세 데이터의 전반적인 정보를 확인
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/5971044d-d7b8-41b0-883c-de8666700800)
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/be4efff6-be05-4708-b345-c5da63e6dfd7)
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/587800aa-bc1b-44fa-933c-cf4707e3b05f)


### 통계분석
- 목적: 통계적 유의성 검정, 불필요한 변수 파악
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/f67cb3d4-9361-4cd2-850d-9b2ebe5868b6)
#### 문제상황: 선형 회귀분석을 실행하려 했지만 데이터의 정규성과 등분산성이 만족하지 않음
- 해결책: 다중 로지스틱 회귀분석을 실행
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/6c1738d4-7def-47df-91ac-84e8e84ecbbd)
- 가설 설정:
  - 귀무가설: 독립변수(매매가격, 임대면적,..기준금리)가 종속변수(전세가격)에 영향이 없다.
  - 대립가설: 독립변수(매매가격, 임대면적,..기준금리)가 종속변수(전세가격)에 하나라도 영향이 있다.
#### 문제상황: 범주형 데이터(년월, 자치구명)에 원핫 인코딩을 실시하니 변수가 지나치게 증가
- 해결책: 타겟 인코딩으로 방식을 변경하여 수행

- 교차검증과 학습곡선을 확인하여 모형을 진단함
 ![image](https://github.com/HSYhrae/APT_Project/assets/139428828/b0811491-04c3-4783-a6bd-58b18787414b)

- VIF와 p-value값을 기준으로 건물용도별 제거해야 할 변수를 파악함
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/6e2f41b3-0bfe-4592-8113-5b0885724bf7)



### ML/DL
- 다양한 모델을 사용해 보았고 그 중에서 성능이 좋은 것을 선택함(LightGBM Regressor)
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/6efeb53a-0e73-4157-953d-3d984f436673)
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/30349e79-222e-4335-b647-d4812ac2fab6)

- LGBM Regressor 성능 평가
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/8afca21a-d541-4278-9a33-2575be21ddbf)

- 추가로 미래 예측값을 도출하기 위해 Prophet 사

### Streamlit
- 현재 대시보드를 실행시키기 위해서 구글 지오코딩의 API키를 발급해서 실행해야함
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/4376397e-5f0c-425c-be6c-1ff31ac91c54)
![image](https://github.com/HSYhrae/APT_Project/assets/139428828/46cf3c3b-ad31-435d-bdce-ea4723a79cc6)


### 최종 결론
- 당초 목표했던 서울 지역의 전세가격을 예측하여 대시보드 형태로 표현하는 구현함 
- 전세 부동산을 알아보는 이용자들에게 유용할 것으로 판단
- 전세가율을 통해 위험 매물인지 판단 가능

- 한계점
  - 서울외지역의 부동산 가격을 예측하지 못함
  - 보다 세분화된 모델을 생성하지 못함
  - 대시보드에 구현한 인프라의 종류가 부족함

### 개인적인 소감
1. 작은 프로젝트지만 해당 경험을 통해 데이터 분석 프로젝트가 어떻게 진행되는지 전체적인 윤곽을 알수 있었음
2. 데이터 수집, 데이터 전처리, EDA, ppt 작성에만 집중하여 다른 파트의 내용에 대한 이해도가 낮음









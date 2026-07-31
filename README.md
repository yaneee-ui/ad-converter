# 쇼핑검색광고 실적 대시보드

네이버 쇼핑검색광고(+ EP채널 비교) 실적을 조회하고 전년 대비 비교하는 Streamlit 대시보드입니다.

## 구성

```
shopping-ad-dashboard/
├── app.py                      # 메인 대시보드 (사이드바: 실적 / 전년비교 / 카테고리별 실적)
├── utils.py                    # 데이터 로드 + 비율지표 재계산 + 기간/비교 로직
├── styles.py                   # 다크 사이드바 + KPI 카드 + 배지 스타일 컴포넌트
├── converter_app.py            # 원본 리포트 → 대시보드용 CSV 변환기 (별도 실행)
├── data/
│   ├── shopping_ad_daily.csv   # 일자별 쇼핑검색광고 실적 원천 데이터
│   └── category_daily.csv      # 카테고리별 쇼핑검색광고/EP채널 거래액 원천 데이터
├── requirements.txt
└── README.md
```

## 대시보드 페이지

1. **쇼핑검색광고 실적** — 기준일자(일/주/월 단위 선택) + 표시방식(누계/일평균), KPI 카드
   (전일·전주·전월비 + 전년비 배지), 실적요약 비교 테이블, 2026년 추이(전년비 비교선) 차트, 원본 데이터 다운로드
2. **전년비교** — 일자별 YoY(전년 동일 요일 매칭) / 월별 누적 YoY
3. **카테고리별 실적** — 카테고리(12개)별 쇼핑검색광고 vs EP채널 거래액 비교, 광고비중, 랭킹 차트/테이블,
   카테고리별 2026년 추이

## 비율 지표 처리 원칙

CTR, CR, 객단가, ROAS, 순결제비중 등 비율 지표는 절대 **일자별 값의 평균을 내지 않습니다.**
대신 `utils.py`의 `RATIO_DEFS`에 정의된 분자/분모 원천값(base metric)을 먼저 합산한 뒤
재계산하여, 어떤 기간으로 집계하든 정확한 값이 나오도록 했습니다.

## 로컬 실행

```bash
pip install -r requirements.txt
streamlit run app.py
```

## GitHub 업로드 & Streamlit Community Cloud 배포

```bash
git init
git add .
git commit -m "Initial commit: 쇼핑검색광고 실적 대시보드"
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>
git push -u origin main
```

이후 [share.streamlit.io](https://share.streamlit.io) 에서 New app → 방금 만든 repo 선택 →
Main file path를 `app.py`로 지정하면 배포됩니다.

## 데이터 갱신 (매번 반복되는 작업 → 변환기 웹앱 사용)

두 원본 리포트 모두 **매번 전체 누적 기간을 다시 추출하는 형태**이므로, 새 리포트를 받을 때마다
증분 병합할 필요 없이 그냥 전체를 다시 변환해서 기존 CSV를 통째로 교체하면 됩니다.

`converter_app.py`가 이 작업을 해주는 별도 웹앱입니다:

```bash
streamlit run converter_app.py
```

- **① 일별 실적 탭**: `일일리포트_쇼핑검색광고_RAW.xlsx` 업로드 → `shopping_ad_daily.csv` 다운로드
- **② 카테고리별 실적 탭**: `네이버_쇼검_ep거래액_상세_YYYYMMDD.csv` 업로드 → `category_daily.csv` 다운로드

각 탭에서 변환 후 행 수·날짜 범위·날짜 누락 여부를 바로 확인할 수 있고,
다운로드한 파일로 GitHub 리포의 `data/` 폴더 안 동일한 이름의 파일을 교체(덮어쓰기)하면 됩니다.

이 변환기도 `app.py`처럼 Streamlit Community Cloud에 별도 앱으로 배포해두면(Main file path를
`converter_app.py`로 지정), 매번 로컬 실행 없이 웹에서 바로 변환할 수 있습니다.

### 참고: 두 파일이 함께 움직여야 하는 경우

`shopping_ad_daily.csv`(쇼핑검색광고 실적)와 `category_daily.csv`(카테고리별 광고/EP 거래액)는
서로 다른 원본에서 나오는 별개 파일이라 갱신 주기가 어긋나도 대시보드가 깨지지는 않습니다.
다만 "카테고리별 실적" 페이지의 데이터 기간이 "쇼핑검색광고 실적" 페이지보다 하루이틀 짧게
보일 수 있는데, 이는 원본 리포트의 집계 마감 시점 차이일 뿐 오류가 아닙니다.

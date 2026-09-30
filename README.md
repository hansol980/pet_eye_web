# pet-eye web — 반려동물 안구질환 검사 화면

Colab에서 띄운 추론 API를 호출하는 정적 웹 페이지입니다.
사진을 올리면 증상별 이상 확률을 표시합니다.

```
[GitHub Pages]  index.html  ──fetch──▶  [Colab]  FastAPI /predict  ──▶  증상별 모델 6개
   프런트엔드 (이 저장소)                  백엔드 (pet_eye_serve.ipynb)
```

빌드 도구가 없습니다. `index.html` 한 파일이 전부입니다.

## 1. 저장소 만들기

GitHub에서 새 저장소(예: `pet-eye-web`)를 **Public**으로 만든 뒤, 이 폴더를 올립니다.

```bash
cd pet_eye_web
git init
git add index.html README.md
git commit -m "pet-eye 검사 화면 추가"
git branch -M main
git remote add origin https://github.com/<계정>/pet-eye-web.git
git push -u origin main
```

## 2. GitHub Pages 켜기

저장소 **Settings → Pages** 에서

- **Source**: `Deploy from a branch`
- **Branch**: `main` / `/ (root)`

를 선택하고 저장합니다. 1분쯤 뒤 아래 주소가 열립니다.

```
https://<계정>.github.io/pet-eye-web/
```

## 3. 백엔드 띄우기

백엔드는 둘 중 하나를 쓰시면 됩니다. 응답 형식이 같아서 이 화면은 그대로 동작합니다.

| 방법 | 주소 | 특징 |
| --- | --- | --- |
| Colab (`pet_eye_serve.ipynb`) | ngrok 예약 도메인 | GPU라 빠름. 노트북을 켜 둔 동안만 동작 |
| Hugging Face Space (`pet_eye_space/`) | `https://<계정>-pet-eye-api.hf.space` | 상시 가동. 무료는 CPU라 사진당 2~5초 |

### Colab으로 띄우기

`pet_eye_serve.ipynb`를 Colab에서 열고 1번부터 7번 셀까지 실행합니다.
7번 셀이 API 주소를 출력합니다.

```
API 주소: https://pet-eye-api.ngrok-free.app
```

ngrok 대시보드에서 **정적 도메인**을 하나 만들어 노트북의 `DOMAIN`에 적어 두면,
Colab을 껐다 켜도 주소가 바뀌지 않습니다. 이 화면의 API 주소를 매번 고칠 필요가 없습니다.

## 4. 연결

페이지를 열고 맨 위 **API 주소** 칸에 위 주소를 넣은 뒤 **연결 확인**을 누릅니다.
`연결됨 — 모델 6개 …`가 뜨면 사진을 올리시면 됩니다.
주소는 브라우저에 저장되므로 다음부터는 바로 사용하실 수 있습니다.

## 화면 설명

| 항목 | 의미 |
| --- | --- |
| 종 | 검사에 사용할 모델을 고릅니다. 학습한 종에 맞춰 주세요 |
| 초음파 모델 | `us_` 모델(초음파 영상 학습) 포함 여부입니다. 일반 사진이면 제외 |
| 의심 판정 기준 | 이상 확률이 이 값을 넘으면 '의심'으로 표시합니다 |
| 이상 확률 | `1 − P(정상)`. 그 모델이 소견이 있다고 본 정도입니다 |
| 소견 | 정상을 제외한 클래스 중 확률이 가장 높은 것입니다 |

## 문제가 생기면

| 증상 | 확인할 것 |
| --- | --- |
| `연결 실패: Failed to fetch` | Colab 런타임이 끊겼거나 7번 셀을 실행하지 않은 경우입니다 |
| 결과 대신 ngrok 경고 페이지 | 이 페이지는 `ngrok-skip-browser-warning` 헤더를 보내므로 정상적으로는 나오지 않습니다. 브라우저에서 API 주소를 직접 열면 보입니다 |
| CORS 오류 | 노트북 6번 셀의 `ALLOW_ORIGINS`에 `'*'` 또는 이 Pages 주소가 들어 있는지 확인해 주세요 |
| 응답이 느림 | Colab이 CPU 런타임이면 몇 초 걸립니다. GPU 런타임으로 바꾸면 1초 내외입니다 |

## API

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| GET | `/health` | 상태와 모델 목록 |
| GET | `/diseases` | 검사 가능한 증상 목록 |
| POST | `/predict` | `file`(사진)을 받아 증상별 결과 JSON 반환 |

```bash
curl -X POST "https://<도메인>/predict?species=cat&include_us=false" \
     -H "ngrok-skip-browser-warning: true" \
     -F "file=@eye.jpg"
```

```json
{
  "threshold": 0.5,
  "suspect_count": 1,
  "top": { "label": "고양이 결막염", "abnormal": 0.87, "finding": "유(있음)" },
  "results": [
    { "disease": "cat_Conjunctivitis", "label": "고양이 결막염",
      "abnormal": 0.87, "verdict": "의심", "finding": "유(있음)", "finding_prob": 0.87 }
  ]
}
```

## 참고

이 결과는 진단이 아닙니다. 실제 판단은 수의사의 몫입니다.
증상별 모델은 각자 자기 증상 데이터만 보고 학습했으므로, 학습에 없던 형태의 사진에 대한 출력은 보장되지 않습니다.

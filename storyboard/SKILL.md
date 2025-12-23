# Storyboard Image Generator

시나리오를 분석하여 장면별로 분리하고, AI 이미지 생성을 위한 프롬프트를 자동 생성합니다.

---

## 지원 API

| 모델 | 플랫폼 | 가격 | 추천 용도 |
|------|--------|------|----------|
| **Google Imagen 4** | Google AI Studio | **무료** (1,500장/일) | 테스트, 소규모 프로젝트 |
| **FLUX.2 [pro]** | fal.ai | $0.03/MP | 고품질, 상업용 |

### 비용 비교 (100장 기준)

| 모델 | 비용 | 품질 |
|------|------|------|
| Google Imagen 4 | **$0** | ★★★★☆ |
| FLUX.2 [pro] | $3.00 | ★★★★★ |

### 추천
- **무료로 시작** → Google Imagen 4
- **최고 품질 필요** → FLUX.2 [pro]

---

## 실행 조건

사용자가 다음과 같이 요청하면 이 스킬을 실행합니다:
- "스토리보드 만들어줘"
- "시나리오 분석해줘"
- "장면별로 나눠줘"
- "이미지 프롬프트 생성해줘"
- `/storyboard` 명령어 사용

---

## 실행 단계

### 1단계: 시나리오 입력 확인

사용자가 시나리오를 제공하지 않았다면 요청합니다:

```
시나리오 또는 스크립트를 입력해주세요.

예시:
- 영화/드라마 대본
- 광고 스크립트
- 유튜브 영상 기획안
- 스토리 텍스트
- 웹툰/만화 스토리
```

### 2단계: 장면(Scene) 분석

시나리오를 다음 기준으로 장면별로 분리합니다:

**분리 기준:**
- 장소 변경
- 시간 변경 (낮/밤, 다음날 등)
- 주요 액션 전환
- 감정/분위기 전환
- 등장인물 변화

**각 장면에서 추출할 요소:**

| 요소 | 설명 | 예시 |
|------|------|------|
| 장소 (Location) | 배경 환경 | 도시 거리, 카페 내부, 숲속 |
| 시간대 (Time) | 시간/조명 | 새벽, 황혼, 한밤중 |
| 인물 (Characters) | 등장인물 | 30대 남성, 노인, 어린이 |
| 액션 (Action) | 주요 동작 | 걷고 있다, 대화 중, 달리는 |
| 감정 (Mood) | 분위기 | 긴장감, 평화로움, 슬픔 |
| 카메라 (Camera) | 촬영 기법 | 클로즈업, 와이드샷, 로우앵글 |
| 소품 (Props) | 중요 오브젝트 | 빨간 우산, 오래된 편지 |

### 3단계: 장면별 분석 결과 출력

다음 형식으로 각 장면을 정리합니다:

```
## Scene 1: [장면 제목]

### 장면 분석
- 장소: [Location]
- 시간대: [Time of day]
- 등장인물: [Characters]
- 주요 액션: [Action]
- 분위기: [Mood]
- 카메라: [Camera angle]

### 원본 텍스트
> [해당 장면의 원본 시나리오 텍스트]
```

### 4단계: AI 이미지 프롬프트 생성

각 장면에 대해 이미지 프롬프트를 생성합니다:

**프롬프트 구조:**
```
[주제/액션], [인물 묘사], [배경/장소], [시간대/조명], [분위기/스타일], [카메라 앵글], [품질 키워드]
```

**예시:**
```
A young woman walking alone on a rainy city street at night,
wearing a red coat, neon lights reflecting on wet pavement,
cinematic lighting, melancholic mood, wide shot,
photorealistic, 8k, detailed
```

**스타일 옵션 (사용자 선택 가능):**
- `photorealistic` - 실사 스타일
- `cinematic` - 영화적 스타일
- `anime` - 애니메이션 스타일
- `illustration` - 일러스트 스타일
- `3d render` - 3D 렌더링 스타일
- `watercolor` - 수채화 스타일
- `oil painting` - 유화 스타일

### 5단계: 최종 출력 형식

```markdown
# Storyboard: [프로젝트 제목]

## 개요
- 총 장면 수: N개
- 스타일: [선택된 스타일]
- 종횡비: [16:9 / 9:16 / 1:1]

---

## Scene 1: [장면 제목]

### 장면 분석
| 요소 | 내용 |
|------|------|
| 장소 | ... |
| 시간대 | ... |
| 인물 | ... |
| 액션 | ... |
| 분위기 | ... |
| 카메라 | ... |

### 이미지 프롬프트 (영어)
```
[English prompt]
```

### 프롬프트 (한국어 참고용)
```
[한국어 설명]
```

---

## Scene 2: ...
[반복]
```

---

## 6단계: API 호출 코드 생성

사용자가 요청하면 실제 API 호출 코드를 생성합니다:

### Option 1: Google Imagen 4 (무료 1,500장/일)

```python
import google.generativeai as genai
import os

genai.configure(api_key=os.environ["GEMINI_API_KEY"])

def generate_image_imagen(prompt: str, output_path: str):
    """Google Imagen으로 이미지 생성 (무료)"""
    response = genai.ImageGenerationModel("imagen-3.0-generate-002").generate_images(
        prompt=prompt,
        number_of_images=1,
        aspect_ratio="16:9",
    )
    response.images[0].save(output_path)
    return output_path
```

### Option 2: FLUX.2 [pro] via fal.ai (최고 품질)

```python
import fal_client
import requests
import os

os.environ["FAL_KEY"] = "your_fal_api_key"

def generate_image_flux2(prompt: str, output_path: str):
    """FLUX.2 Pro로 이미지 생성 (최고 품질)"""
    result = fal_client.subscribe(
        "fal-ai/flux-2-pro",
        arguments={
            "prompt": prompt,
            "image_size": "landscape_16_9",
            "num_images": 1,
        }
    )

    img_url = result["images"][0]["url"]
    img_data = requests.get(img_url).content
    with open(output_path, "wb") as f:
        f.write(img_data)

    return output_path
```

### 스토리보드 일괄 생성 함수

```python
def generate_storyboard(scenes: list, api: str = "imagen"):
    """모든 장면 이미지 일괄 생성

    Args:
        scenes: [{"prompt": "..."}, ...]
        api: "imagen" (무료) | "flux2" (고품질)
    """
    generators = {
        "imagen": generate_image_imagen,
        "flux2": generate_image_flux2,
    }

    generate_fn = generators[api]
    results = []

    for i, scene in enumerate(scenes):
        print(f"Generating Scene {i+1}/{len(scenes)}...")
        img_path = f"scene_{i+1:02d}.png"
        generate_fn(scene["prompt"], img_path)
        results.append({"scene": i+1, "image": img_path})
        print(f"  ✓ Saved: {img_path}")

    return results

# 사용 예시
scenes = [
    {"prompt": "A woman with red umbrella walking in rainy alley at night, neon lights, cinematic, 8k"},
    {"prompt": "Close-up of woman turning around, surprised expression, rain drops, dramatic lighting"},
    {"prompt": "Two old friends facing each other, emotional reunion, warm smile, soft lighting"},
]

# 무료: Google Imagen
generate_storyboard(scenes, api="imagen")

# 고품질: FLUX.2 Pro
generate_storyboard(scenes, api="flux2")
```

---

## 프롬프트 작성 팁

### 좋은 프롬프트 구조

```
[Subject] + [Action] + [Setting] + [Lighting] + [Mood] + [Camera] + [Style]
```

### 품질 향상 키워드

**화질:**
- `8k`, `4k`, `high resolution`, `detailed`, `sharp focus`

**스타일:**
- `cinematic`, `film grain`, `anamorphic lens`, `shallow depth of field`

**조명:**
- `golden hour`, `blue hour`, `neon lighting`, `volumetric lighting`, `rim lighting`

**분위기:**
- `atmospheric`, `moody`, `dramatic`, `ethereal`, `nostalgic`

### 카메라 앵글 키워드

| 앵글 | 키워드 | 효과 |
|------|--------|------|
| 와이드 | `wide shot`, `establishing shot` | 전체 장면 |
| 미디엄 | `medium shot`, `waist-up` | 인물 중심 |
| 클로즈업 | `close-up`, `face detail` | 감정 표현 |
| 익스트림 클로즈업 | `extreme close-up`, `macro` | 디테일 강조 |
| 로우앵글 | `low angle`, `worm's eye view` | 위압감 |
| 하이앵글 | `high angle`, `bird's eye view` | 취약함 |
| 더치앵글 | `dutch angle`, `tilted` | 불안감 |

### 피해야 할 표현

- 부정적 표현 ("not", "without", "no")
- 모호한 표현 ("nice", "good", "beautiful")
- 너무 긴 문장 (핵심만 간결하게)

---

## 예시: 완성된 스토리보드

### 입력 시나리오
```
비 오는 밤, 도시의 골목길.
한 여성이 빨간 우산을 쓰고 천천히 걸어간다.
갑자기 뒤에서 누군가 이름을 부른다.
여성이 돌아보면 오래된 친구가 서 있다.
두 사람이 서로를 바라보며 미소 짓는다.
```

### 출력 예시

---

#### Scene 1: 비 오는 골목길

| 요소 | 내용 |
|------|------|
| 장소 | 도시 골목길, 젖은 아스팔트 |
| 시간대 | 밤, 네온 조명 |
| 인물 | 빨간 우산 쓴 여성 (뒷모습) |
| 액션 | 천천히 걸어감 |
| 분위기 | 고독함, 서정적 |
| 카메라 | 와이드샷, 뒤에서 |

**이미지 프롬프트:**
```
Back view of a woman holding a red umbrella, walking alone
through a narrow city alley on a rainy night,
neon lights reflecting on wet pavement,
cinematic lighting, melancholic mood, wide shot,
photorealistic, 8k, detailed
```

---

#### Scene 2: 돌아보는 순간

| 요소 | 내용 |
|------|------|
| 장소 | 동일 골목길 |
| 시간대 | 밤 |
| 인물 | 여성 (얼굴 보임, 놀란 표정) |
| 액션 | 뒤돌아봄 |
| 분위기 | 긴장, 기대 |
| 카메라 | 미디엄 클로즈업 |

**이미지 프롬프트:**
```
Medium close-up of a woman turning around with surprised expression,
holding red umbrella, rain drops on her face,
neon lights in background, dramatic lighting,
cinematic, shallow depth of field, 8k
```

---

#### Scene 3: 재회의 미소

| 요소 | 내용 |
|------|------|
| 장소 | 동일 골목길 |
| 시간대 | 밤 |
| 인물 | 여성, 오래된 친구 (남성) |
| 액션 | 서로 바라보며 미소 |
| 분위기 | 따뜻함, 감동 |
| 카메라 | 투샷, 미디엄 |

**이미지 프롬프트:**
```
Two old friends facing each other in a rainy alley at night,
woman with red umbrella and man, both smiling warmly,
emotional reunion, soft warm lighting mixed with neon,
cinematic, medium two-shot, 8k, detailed
```

---

## 사용자 옵션

스킬 실행 시 다음 옵션을 선택할 수 있습니다:

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| 스타일 | photorealistic, cinematic, anime, etc. | cinematic |
| 종횡비 | 16:9, 9:16, 1:1, 4:3 | 16:9 |
| 언어 | 영어, 한국어 | 영어 |
| API | imagen (무료), flux2 (고품질) | imagen |

---

## API 설정 가이드

### Google AI Studio (Imagen 4) - 무료
1. https://aistudio.google.com 접속
2. API 키 생성
3. `GEMINI_API_KEY` 환경변수 설정
4. `pip install google-generativeai`

```bash
export GEMINI_API_KEY="your_api_key"
```

### fal.ai (FLUX.2) - 고품질
1. https://fal.ai 가입
2. API 키 생성
3. `FAL_KEY` 환경변수 설정
4. `pip install fal-client requests`

```bash
export FAL_KEY="your_api_key"
```

---

## 주의사항

1. **저작권**: 생성된 이미지의 상업적 사용 시 각 API의 이용약관 확인
2. **비용**: FLUX.2는 이미지당 약 $0.03 (100장 = $3)
3. **일관성**: 동일 인물/장소의 일관성 유지가 어려울 수 있음
   - 해결책: 참조 이미지 사용, 상세한 인물 묘사 일관되게 유지
4. **API 한도**: Google Imagen 무료 티어는 1,500장/일 제한

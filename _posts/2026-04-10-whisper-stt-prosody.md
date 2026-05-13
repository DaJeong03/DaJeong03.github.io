---
title: "Whisper로 음성을 텍스트로 — STT 실습"
date: 2026-05-13 00:00:00 +0900
categories: [공부기록, 음성AI]
tags: [whisper, STT, 음성인식, prosody, emotional-tts]
math: true
---

오늘은 Whisper로 내 목소리를 텍스트로 변환하는 STT 실습을 했다.

---

## 1. Whisper란?

Whisper는 OpenAI가 만든 음성인식(STT) 모델이다.
현재 업계에서 STT의 기준점이 되는 모델로,
68만 시간의 다국어 음성 데이터로 학습되었다.

모델 크기는 5가지가 있다.

| 모델 | 크기 | 특징 |
|---|---|---|
| tiny | 가장 작음 | 빠르지만 정확도 낮음 |
| base | 작음 | 가볍고 빠름 |
| small | 중간 | 균형잡힌 성능 |
| medium | 큼 | 정확도 높음 |
| large | 가장 큼 | 가장 정확, 연산량 많음 |

무조건 큰 모델이 좋은 게 아니다.
실시간 서비스라면 `base`나 `tiny`를 쓰고,
정교한 자막 제작이나 데이터 라벨링에는 `large`를 사욯한다.

---

## 2. Colab에서 Whisper 돌려보기

```python
!pip install openai-whisper
!apt-get install -y ffmpeg

import whisper
model = whisper.load_model("base")
print("Whisper 모델 로드 완료!")
```

마이크로 직접 목소리를 녹음해서 STT를 실행했다.

```python
result = model.transcribe("recorded.wav", language="ko")
print("인식 결과:", result["text"])
```

결과는 꽤 정확했다.
한국어도 잘 인식했고, 짧은 문장은 거의 완벽하게 변환됐다.

---

## 3. Whisper의 내부 구조 — 어떻게 동작하는가?
오디오 입력
↓
Mel-spectrogram 변환
↓
Encoder — 오디오 특징 압축
↓
Decoder — 텍스트 하나씩 예측
↓
텍스트 출력

---

## 4. 환각(Hallucination)과 Prosody

실습 중 흥미로운 현상을 발견했다.

**"안~녕~하~세~요"** 라고 길게 늘여서 말했는데
결과는 정갈하게 **"안녕 하세요"** 로 나왔다.

### 왜 이런 현상이 생기나?

Whisper는 소리를 그대로 옮기는 게 아니라
**"사람이 읽기 좋은 텍스트"** 를 생성하도록 훈련됐다.

말을 더듬거나, 모음을 길게 늘여도
모델은 이를 노이즈로 간주하고
최종적인 의미인 "안녕하세요"로 수렴시킨다.
이를 **텍스트 정규화(Text Normalization)** 라고 한다.

일반적인 받아쓰기 서비스에서는 훌륭한 기능이지만,
감정이나 강조를 파악해야 하는 상황에서는 적절하지 않다.

### word_timestamps로 확인해보기

Whisper의 `word_timestamps=True` 옵션으로
각 단어의 시간 정보를 확인해 봤다.

```python
result = model.transcribe(
    "recorded.wav",
    language="ko",
    word_timestamps=True
)

for segment in result["segments"]:
    for word in segment["words"]:
        duration = word["end"] - word["start"]
        print(f"{word['word']} | {word['start']:.2f}s ~ {word['end']:.2f}s | 길이: {duration:.2f}s")
```

**"안녕"** 이라는 두 음절에 **1.82초** 가 걸렸다.
평소 대화 속도라면 0.3~0.5초면 충분한데,
무려 4~5배나 길게 측정된 것이다.

---

## 5. 이게 왜 중요한가? — Emotional TTS와의 연결

텍스트는 똑같이 **"안녕"** 이지만
- 0.4초로 발음하면 → 무뚝뚝한 인사
- 1.82초로 발음하면 → 반가움이나 장난기 섞인 인사

소리의 **길이(Duration)** 가 감정을 결정하는 것이다.

최신 TTS 모델(VITS, FastSpeech 2) 내부에는
**Duration Predictor** 라는 모듈이 있다.
이 모듈이 텍스트를 보고 각 음절을 얼마나 길게 발음할지 계산한다.

오늘 측정한 1.82초라는 데이터가 바로
이런 모델을 학습시키는 정답 데이터가 되는 것이다.

---

## 6. 환각(Hallucination)

Whisper를 쓰다 보면 가끔 이런 현상이 생긴다.

- 아무 말도 안 했는데 자막이 생긴다
- 같은 말을 무한 반복한다

Whisper가 너무 '똑똑해서' 소리가 없어도
문맥상 어울리는 말을 지어내려는 성질 때문이다.

이 현상을 차단하기 위해서
말소리가 없는 구간을 미리 잘라내고 Whisper에 전달한다.

---

## 마치며

오늘 실습에서 배운 것을 한 줄로 정리하면 이렇다.

> "Whisper는 텍스트의 내용은 잘 잡지만,
> 소리의 감정(Duration, Pitch)은 담지 않는다."

다음에는 오디오에서 Pitch(음높이)를 직접 추출해서
감정 정보를 시각화해볼 예정이다.

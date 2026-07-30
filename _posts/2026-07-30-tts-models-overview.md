---
title: "TTS 모델 탐구 — gTTS부터 VITS, StyleTTS2까지"
date: 2026-07-30 00:00:00 +0900
categories: [공부기록, 음성AI]
tags: [TTS, gTTS, VITS, StyleTTS2, Tacotron2, 음성합성]
---

지금까지 음성을 텍스트로 변환하는 STT(Whisper)를 공부했다.
이번에는 **텍스트를 음성으로 변환하는 TTS**를 공부했다.

---

## 1. TTS란?

TTS(Text-to-Speech)는 텍스트를 입력하면 음성을 출력하는 기술이다.
스마트폰 음성 안내, 오디오북, AI 스피커 등 일상에서 많이 쓰이고 있다.

---

## 2. TTS 모델의 발전 흐름
gTTS (Google TTS)
→ 가장 쉽게 쓸 수 있는 TTS
→ Google 서버에 API 요청하는 방식
→ 인터넷 필요, 커스터마이징 불가

Tacotron 2 (Google, 2018)
→ 딥러닝 기반 TTS의 시작점
→ 텍스트 → Mel-spectrogram → Vocoder → 음성
→ 자연스럽지만 두 모델이 따로 필요

VITS (2021)
→ End-to-End TTS
→ 텍스트 → 음성
→ 더 자연스럽고 빠름
→ Duration Predictor로 발음 길이 제어

StyleTTS2 (2023)
→ VITS의 발전 버전
→ 화자의 스타일(억양, 감정, 속도)까지 학습

---

## 3. Tacotron 2 구조

텍스트 입력
↓
Encoder → 텍스트를 압축된 특징으로 변환
↓
Attention → 어느 텍스트에 집중할지 결정
↓
Decoder → Mel-spectrogram 생성
↓
WaveNet Vocoder → Mel-spectrogram → 실제 음성 파형

---

## 4. VITS가 Tacotron 2보다 나은 이유

| 구분 | Tacotron 2 | VITS |
|---|---|---|
| 구조 | 2단계 (TTS + Vocoder) | 1단계 (End-to-End) |
| 음질 | 좋음 | 더 좋음 |
| 속도 | 느림 | 빠름 |
| 커스터마이징 | 어려움 | 쉬움 |
| 보이스 클로닝 | 불가 | 가능 |

2단계 구조에서는 각 단계의 오류가 누적되지만
VITS는 한 번에 처리하므로 오류 누적이 없다.

---

## 보이스 클로닝?

보이스 클로닝 (Voice Cloning)은 
특정 사람의 목소리를 AI가 학습해서 똑같이 흉내내는 기술이다.

일반 TTS
→ "안녕하세요"를 입력하면
→ 기계적인 표준 목소리로 출력

보이스 클로닝 TTS
→ 특정 사람 목소리 샘플 몇 초 입력
→ "안녕하세요"를 그 사람 목소리로 출력

**관련 모델들**
SV2TTS (2018)   →  보이스 클로닝 개념의 시작
YourTTS (2021)  →  다국어 보이스 클로닝
XTTS (2023)     →  3초 샘플로도 클로닝 가능
                    현재 가장 많이 쓰이는 클로닝 모델

---

## 5. VITS의 핵심 — Duration Predictor가 무엇인가?

텍스트: "안녕"
Duration Predictor: "이 문장은 반가운 감정이니
첫 음절 0.9초, 둘째 음절 0.9초로 늘려야지"
→ 자연스러운 감정이 담긴 음성 출력

---

## 6. gTTS 실습 결과

Colab 환경 제약으로 VITS 직접 실행은 어려웠지만
gTTS로 TTS 결과물이 어떤 것인지 감을 잡을 수 있었다.

```python
from gtts import gTTS
from IPython.display import Audio

text = "She sells seashells by the seashore"
tts = gTTS(text=text, lang='en', slow=False)
tts.save("output.mp3")
Audio("output.mp3")
```

속도(slow=True/False), 언어(lang) 파라미터로
기본적인 조절이 가능하다.

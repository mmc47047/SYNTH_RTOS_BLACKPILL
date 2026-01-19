🎹 RTOS 기반 디지털 신디사이저
<table> <tr> <td width="45%" valign="top"> <img src="https://github.com/user-attachments/assets/4f4dce32-1f35-45e3-bfb7-9151e01d296c" width="100%" alt="Project Photo"> </td> <td width="55%" valign="top"> <h3>RTOS 기반 디지털 신디사이저</h3> <p> STM32 BlackPill(STM32F411)에서 <b>44.1kHz 실시간 오디오</b>를 생성하고,<br/> <b>키패드/로터리 입력</b>을 통해 <b>Pitch(옥타브)</b>, <b>파형</b>, <b>ADSR</b>, <b>LPF(Cutoff/Resonance)</b>, <b>Volume</b>을 제어하는 <b>RTOS 기반 디지털 신디사이저</b> 프로젝트입니다. </p> <p> 본 프로젝트는 <b>오디오 처리, 입력 처리, UI 시각화</b>를 RTOS 기반으로 분리하여 설계하였으며,<br/> 저는 그중 <b>ILI9341 TFT LCD 기반 UI 구현 및 입력–UI 연동 로직</b>을 담당했습니다. </p> <ul> <li><b>Audio</b>: I2S + DMA(Circular), DDS + ADSR + IIR LPF</li> <li><b>UI</b>: ILI9341 TFT LCD (SPI), RTOS LCD Task, 부분 갱신 최적화</li> <li><b>Input</b>: 4×4 Keypad + Rotary Encoder ×2</li> </ul> </td> </tr> </table>
✨ 1. 주요 기능 (Features)
🎼 (1) Keypad 기능

음도 입력 (도, 레, 미, 파, 솔, 라, 시)

파형 선택 (Sine / Square / Saw)

옥타브 Up / Down

🎚️ (2) Rotary 기능

Rotary #1: ADSR/Filter 파라미터 중 “선택된 항목” 값 변경

Rotary #1 버튼: ADSR(A/D/S/R) ↔ Filter(Cutoff/Q) 항목 선택 및 모드 이동

Rotary #2: Volume 조절

입력 이벤트는 RTOS 환경에서 UI Task와 연동되어,
파라미터 변경 시 화면 그래프와 표시가 즉시 반영되도록 구현하였습니다.

🎧 (3) DSP 기능

ADSR Envelope (A/D/S/R 상태 머신)

2차 IIR LPF (Biquad): Cutoff / Resonance(Q)

Polyphony (멀티 보이스) + Voice Stealing

TFT UI 시각화: ADSR/파형 그래프, 파라미터 표시

🧩 2. 구성도 (Hardware / System)
<img width="1080" height="522" alt="image" src="https://github.com/user-attachments/assets/3e9757d5-a652-40d4-9eae-89c765891a52" />

키패드: 노트 / 파형 / 옥타브 입력

로터리1: ADSR/Filter 편집 값 변경 + 버튼으로 항목 선택/모드 이동

로터리2: 볼륨 조절

TFT-LCD: 현재 파라미터 및 ADSR/파형 시각화

I2S DAC: 스피커/이어폰 출력

🎛️ 3. 파형 생성 (DDS/NCO 방식)

(원문 그대로 유지 — 오디오 알고리즘 이해도는 팀 역량으로 충분히 어필됨)

tuning_word = f_target * 2^32 / F_sample

phase_acc += tuning_word

index = phase_acc >> SHIFT 로 LUT 접근

샘플레이트: F_sample = 44.1kHz

🔊 4. 오디오 신호 처리 흐름
4.1 신호 처리 개념 (ADSR / IIR LPF)

(이미지 그대로 유지)

ADSR Envelope: NoteOn/NoteOff에 따른 진폭 변화

IIR Low-Pass Filter (2nd Order Biquad)

Cutoff(Fc): 음색 밝기/고조파 조절

Resonance(Q): 컷오프 근처 강조

해당 파라미터들은 UI에서 실시간으로 조정되며,
변경 내용은 TFT LCD에 그래프 및 수치로 즉시 시각화됩니다.

⏱️ 5. I2S DMA Circular Buffer (Half/Full Ping-Pong)

(원문 유지 — RTOS·DMA 설계 이해도 어필)

UI Task와 Audio Task를 분리하여,
UI 갱신으로 인한 부하가 오디오 스트림에 영향을 주지 않도록 설계했습니다.

🖥️ 6. RTOS 기반 UI 설계 (LCD Task)
핵심 설계 포인트

LCD 처리를 별도의 LCD Task로 분리

UI 상태 변경 시 Dirty Flag 구조를 사용해 필요한 영역만 부분 갱신

전체 리드로우를 최소화하여 깜빡임 및 지연 현상 개선

부분 갱신 예시

ADSR 값 변경 → ADSR 그래프 영역만 갱신

선택 이동 → 라벨/선택 박스 영역만 갱신

볼륨 변경 → 하단 볼륨 바만 갱신

🎥 7. 데모 영상 (Demo Videos)

(기존 데모 유지)

🧯 8. 트러블슈팅 (Troubleshooting)
🖥️ LCD SPI 통신 오류

현상: LCD 화면이 간헐적으로 꺼짐/깨짐

원인: LCD 라이브러리의 SPI 타임아웃 설정(1ms)

해결: 타임아웃을 HAL_MAX_DELAY로 수정 → 화면 안정화

🖥️ UI 성능 문제

현상: 전체 화면 리드로우로 인한 지연 가능성

해결: Dirty Flag 기반 부분 갱신 구조로 변경 → UI 응답성 향상

📌 9. 배운 점 (What I Learned)

RTOS 환경에서 오디오 처리와 UI 처리를 분리하는 구조의 중요성

임베디드 UI에서도 부분 갱신 최적화가 체감 성능에 큰 영향을 미침

입력 → 상태 → 시각화까지의 흐름을 명확히 분리하면 유지보수성이 크게 향상됨

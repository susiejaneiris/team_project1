# 마실 · Vue 3 + Vite

로그인 없는 지역 나들이 지도 SPA. 기존 단일 `masil.html`(CDN 방식)을 **Vue 3 + Vite** 표준 프로젝트 구조로 변환한 버전입니다.

## 실행

> Node.js 18+ 필요 (https://nodejs.org 에서 LTS 설치)

```bash
cd masil-vite
npm install      # 최초 1회 (vue, vite, leaflet 등 설치)
npm run dev      # 개발 서버 (http://localhost:5173)
npm run build    # dist/ 로 정적 빌드
npm run preview  # 빌드 결과 미리보기
```

정적 배포(GitHub Pages 등)는 `npm run build` 결과물인 `dist/` 폴더를 올리면 됩니다.

## 구조

```
masil-vite/
├─ index.html                 # 진입점 (#app 마운트 + 폰트 CDN)
├─ vite.config.js             # Vite 설정 (base:'./' 정적배포 대비)
├─ package.json
└─ src/
   ├─ main.js                 # createApp + leaflet CSS + 전역 CSS
   ├─ App.vue                 # 메인 앱 (홈/지도/캘린더/커뮤니티)
   ├─ style.css               # 전역 스타일 (기존 <style> 이전)
   ├─ components/
   │  └─ ChatWidget.vue       # 챗봇 (OpenAI 직접 호출)
   ├─ lib.js                  # 공용 상수/헬퍼 (CAT_COLOR, HERO, avatars 등)
   └─ data/
      └─ masil-data.js        # 장소 데이터 (TourAPI · export DATA, STATS)
```


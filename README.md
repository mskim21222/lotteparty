# 롯데캐피탈 입사 축하 파티 초대 게임

이 프로젝트는 롯데캐피탈 입사자들을 위한 축하 파티 초대 게임입니다. React로 제작되었으며, GitHub Pages로 배포할 수 있습니다.

## 주요 기능
- 방향키로 캐릭터 이동
- 엔터키로 NPC(부모님, 친구, 선생님, 동기)에게 초대장 전달
- 모든 NPC 초대 시 파티 화면 및 다짐 입력
- 다짐 제출 후 환영 메시지 및 게임 재시작 가능

## 실행 방법
1. Node.js와 npm이 설치되어 있어야 합니다.
2. 프로젝트 디렉토리에서 아래 명령어 실행:
   ```bash
   npm install
   npm run dev
   ```
3. 브라우저에서 `http://localhost:5173` 접속

## GitHub Pages 배포 방법
1. `package.json`의 `homepage` 필드에 배포 주소를 입력합니다.
2. 아래 명령어로 빌드 및 배포:
   ```bash
   npm run build
   npm run deploy
   ```

---

본 프로젝트는 Vite + React 기반으로 제작되었습니다. 
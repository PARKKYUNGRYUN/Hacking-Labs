# [TryHackMe] RootMe 취약점 분석 보고서

## 1. 대상 정보 및 요약
* **대상 환경:** Linux (Apache 2.4.29)
* **주요 취약점:** 
  1. 웹 파일 업로드 확장자 검증 미흡 (`.php5` 우회)
  2. 비정상 SUID 권한 설정 (`/usr/bin/python`)
* **결과:** 일반 웹 권한(`www-data`) 획득 후 `root` 최고 관리자 권한 탈취 성공

---

## 2. 정보 수집 (Reconnaissance)
### ① 포트 스캐닝 (`Nmap`)
* `nmap -sC -sV [Target_IP]` 실행 결과 `22(SSH)`와 `80(HTTP)` 포트가 열려 있음을 확인했습니다.

### ② 숨겨진 디렉터리 탐색 (`Gobuster`)
* 웹 디렉터리 스캔 결과 관리자 패널로 추정되는 `/panel/` 경로와 업로드된 파일이 저장되는 `/uploads/` 경로를 발견했습니다.

---

## 3. 취약점 공격 (Exploitation)
### ① 파일 업로드 필터링 우회 및 웹셸(Web Shell) 실행
* `/panel/` 페이지에서 `.php` 확장자 업로드가 차단되어 있었으나, 서버 설정상 실행 가능한 대체 확장자인 **`.phtml`**로 변경하여 리버스 셸(Reverse Shell) 파일을 업로드하는 데 성공했습니다.
* `/uploads/` 경로에서 해당 파일을 호출하여 Netcat 리스너로 `www-data` 셸을 획득했습니다.

---

## 4. 권한 상승 (Privilege Escalation)
### ① SUID 바이너리 탐색
* `find / -user root -perm /4000 2>/dev/null` 명령어로 SUID 비트가 설정된 파일을 검색한 결과, 일반적이지 않은 **`/usr/bin/python`**을 발견했습니다.

### ② Root 권한 탈취
* GTFOBins의 Python SUID 익스플로잇 구문을 활용해 `root` 권한 셸을 획득했습니다.
* 실행 명령어: `python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

---

## 5. 보안 대응 방안 (Remediation)
* **파일 업로드 취약점 대응:** 업로드 가능한 파일 확장자를 화이트리스트(이미지 등 허용된 확장자만 통과) 방식으로 엄격히 검증하고, 업로드 폴더(`/uploads/`)에 스크립트 실행 권한을 제거해야 합니다.
* **권한 상승 취약점 대응:** `/usr/bin/python` 바이너리에 불필요하게 부여된 SUID 권한을 `chmod u-s /usr/bin/python` 명령어로 즉시 제거해야 합니다.

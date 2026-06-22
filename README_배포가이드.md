# ATLAS 정적 사이트 — 전화 전용 배포 가이드

이 폴더는 ATLAS가 내보내는 마크다운(`content/`에 push됨)을 그대로 웹사이트로 만들어 주는
**Hugo** 프로젝트입니다. 흐름: ATLAS가 글 push → GitHub → Cloudflare Pages 자동 빌드·배포.

────────────────────────────────────────
## 0. 준비물 (둘 다 무료)
- GitHub 계정
- Cloudflare 계정
둘 다 폰 브라우저로 가입 가능.

────────────────────────────────────────
## 1. GitHub 빈 저장소 만들기
1) github.com → 가입/로그인
2) 우상단 + → New repository
3) 이름: `atlas-site` / **Public** / (README 체크 해제) → Create
4) 만든 뒤 주소 기억: `https://github.com/<아이디>/atlas-site`
5) Personal Access Token 발급(서버가 push할 때 필요):
   Settings → Developer settings → Personal access tokens → Tokens(classic)
   → Generate new token → scope `repo` 체크 → 생성 → **토큰 문자열 복사**(한 번만 보임)

────────────────────────────────────────
## 2. 이 스캐폴드를 서버에 올리고 GitHub에 push
SFTP로 `atlas-site.zip`을 서버 `~`에 업로드한 뒤:

```bash
cd ~
unzip -o atlas-site.zip          # → ~/atlas-site/ 생성
cd ~/atlas-site

# 이미 만든 글을 사이트에 한 번 복사(첫 글 표시용)
cp ~/atlas/sites/site-01/content/*.md content/ 2>/dev/null || true

git init -b main
git config user.email "bot@atlas.local"
git config user.name "atlas-bot"
git add -A
git commit -m "init: atlas static site"

# 토큰을 URL에 넣어 remote 등록 (TOKEN/아이디 교체)
git remote add origin https://<TOKEN>@github.com/<아이디>/atlas-site.git
git push -u origin main
```
push 성공하면 GitHub 저장소에 파일이 올라옵니다.

> 보안: 토큰이 `.git/config`에 저장됩니다. `~/atlas-site`는 본인 서버이니 괜찮지만,
> 더 안전하게 하려면 SSH 배포키 방식을 쓰세요(선택).

────────────────────────────────────────
## 3. Cloudflare Pages 연결 (자동 배포)
1) dash.cloudflare.com → 가입/로그인
2) 좌측 **Workers & Pages** → Create → **Pages** → Connect to Git
3) GitHub 연동 허용 → `atlas-site` 저장소 선택
4) 빌드 설정:
   - Framework preset: **Hugo**
   - Build command: `hugo`
   - Output directory: `public`
   - 환경변수 추가: `HUGO_VERSION` = `0.128.0` (또는 더 최신)
5) Save and Deploy → 1~2분 뒤 `https://atlas-site-xxxx.pages.dev` 주소 생성
6) 그 주소로 접속해 글이 보이면 성공

────────────────────────────────────────
## 4. ATLAS가 앞으로 자동 게시하도록 연결
`~/atlas/.env`에서(영문 키보드):
```
PUBLISH_MODE=git
GIT_REPO_DIR=/home/ubuntu/atlas-site
GIT_BRANCH=main
SITE_BASE_URL=https://atlas-site-xxxx.pages.dev   # 3번에서 받은 실제 주소(끝 / 없이)
ATLAS_AUTHOR_NAME=원하는 저자명
```
그리고 `~/atlas-site/hugo.toml`의 `baseURL`도 같은 주소로 변경(끝에 / 유지).

이제 ATLAS가 글을 발행(approve)할 때마다 → `content/`에 push → Cloudflare가 자동 빌드 →
몇 분 뒤 사이트에 반영됩니다. 더 손댈 것 없음.

────────────────────────────────────────
## 5. ★ 실제 게시 전 필수
`~/atlas/data/products.csv`의 가격을 **쿠팡 실제가**로, url을 **본인 쿠팡 파트너스 제휴링크**로
교체하세요. 지금 값은 테스트용 placeholder/가짜 링크입니다.

────────────────────────────────────────
## 사이트 이름·디자인 바꾸기
- 이름: `hugo.toml`의 `title`
- 색/폰트/여백: `layouts/_default/baseof.html` 안 `<style>` 블록의 `--accent` 등
변경 후 `~/atlas-site`에서 `git add -A && git commit -m "edit" && git push` 하면 자동 반영.

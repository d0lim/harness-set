# 검증 체크리스트 참조

Step별 검증 시 사용할 구체적 검증 방법 가이드.

## 자동 검증 방법

### 파일 존재 확인
```bash
# 특정 파일 존재 확인
test -f {path} && echo "PASS" || echo "FAIL: {path} not found"

# 디렉토리 존재 확인
test -d {path} && echo "PASS" || echo "FAIL: {path} not found"
```

### 의존성 설치 확인
```bash
# Node.js
cd {project-root} && npm install 2>&1
# 종료 코드 0 = PASS

# Python
cd {project-root} && pip install -r requirements.txt 2>&1

# JVM
cd {project-root} && ./gradlew dependencies 2>&1
```

### 빌드 확인
```bash
# Node.js
cd {project-root} && npm run build 2>&1

# JVM
cd {project-root} && ./gradlew build 2>&1

# Python
cd {project-root} && python -m py_compile {main-file} 2>&1
```

### 테스트 확인
```bash
# Node.js
cd {project-root} && npm test 2>&1

# JVM
cd {project-root} && ./gradlew test 2>&1

# Python
cd {project-root} && pytest 2>&1
```

### 린트 확인
```bash
# ESLint
cd {project-root} && npx eslint . 2>&1

# Prettier
cd {project-root} && npx prettier --check . 2>&1
```

## 수동 검증 방법

### 원본 보존 확인
원본 repo 디렉토리에서 git status를 실행한다:
```bash
cd {original-repo} && git status
```
"nothing to commit, working tree clean"이 출력되면 PASS.

### 컨벤션 일관성 확인
소스 파일을 샘플링하여 네이밍 규칙, import 스타일, 포맷팅이 설계서의 컨벤션 규칙과 일치하는지 확인한다.

### CLAUDE.md 정합성 확인
CLAUDE.md에 기술된 빌드/테스트 명령어를 실제로 실행하여 동작하는지 확인한다.

## 통합 전후 비교 데이터 수집

```bash
# 파일 수
find {original-repos} -type f -not -path '*/.git/*' -not -path '*/node_modules/*' | wc -l
find {target} -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/_workspace/*' | wc -l

# 의존성 수 (Node.js 예시)
cat {repo}/package.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d.get('dependencies',{}))+len(d.get('devDependencies',{})))"
```

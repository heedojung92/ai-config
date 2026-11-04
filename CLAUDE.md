@AGENTS.md

## Claude Code 전용

- 저장소 루트에서 시작한다. 하위 디렉터리에서 시작하면 첫 줄의 import가 외부 import로 처리돼 빠질 수 있고 루트 `.claude/settings.json`도 적용되지 않는다. 저장소 루트 `AGENTS.md`의 내용이 컨텍스트에 없으면 프로젝트 `AGENTS.md`가 로드돼 있어도 작업 전에 루트 파일을 읽는다.
- 루트 `.claude/settings.json`에는 비밀 파일 읽기 deny와 commit·PR attribution 끄기만 둔다. deny는 목록에 없는 파일, 디렉터리 단위 재귀 검색(`grep -r` 등), 스크립트 안의 파일 읽기를 막지 못하므로 `AGENTS.md`의 비밀값 규칙을 그대로 따른다. hook·응답 재생성·전역 설정 변경은 전제하지 않는다.

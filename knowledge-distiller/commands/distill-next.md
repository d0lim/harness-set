기존 _workspace/가 있는 상태에서 다음 라운드를 진행하라.

## 사전 확인

1. `_workspace/pending/` 디렉토리에 `.md` 파일이 있는지 확인한다.
2. pending 파일이 있으면 사용자에게 먼저 안내한다:
   ```
   📋 이전 라운드에서 미반영된 발견사항이 {N}건 있습니다:
   {각 파일의 제목과 핵심 내용 1줄 요약}

   이번 라운드에서 자동으로 참조 노트에 포함하여 재증류합니다.
   ```

## 주제 제안

이전 라운드의 distiller 관계 맵(_workspace/permanents/_relation-map.md)과 composer의 flow-map에서 제안된 추가 리서치 주제를 확인하고, pending 발견사항의 내용도 반영하여 사용자에게 다음 라운드 주제를 제안한 뒤 승인받아 진행한다.

사용자 입력: $ARGUMENTS

$ARGUMENTS에 추가 주제가 있으면 제안 주제에 병합한다.

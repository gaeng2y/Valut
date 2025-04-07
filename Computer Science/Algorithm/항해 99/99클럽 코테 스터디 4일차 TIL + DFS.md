오늘의 문제: [2468번 안전 영역](https://www.acmicpc.net/problem/2468)
- 오늘의 학습 키워드
  - DFS (Deepth First Search)
    - 탐색 노드의 인접 노드의 자식 노드들을 모두 탐색하고,
    - 깊이 우선 탐색(DFS)의 시간 복잡도 일반적으로 노드 수(V), 간선 수(E)라 했을 때 시간 복잡도는 **O(V+E)** 입력 자체를 노드와 간선으로 받기 때문

다시 돌아가서 탐색 노드의 다른 인접 노드 자식들을 모두 탐색하는 방법
- 공부한 내용 본인의 언어로 정리하기
```swift
func dfs(_ x :Int, _ y :Int, _ height:Int, _ visited : inout [[Bool]]){
    for i in direction {
        let nx = x+i.0
        let ny = y+i.1
        if ny < 0 || ny > h - 1 || nx < 0 || nx > h - 1 {
            continue
        }
        if !visited[ny][nx] && land[y][x] > height {
            visited[ny][nx] = true;
            dfs(nx,ny,height, &visited)
        }
    }
}
```
- 오늘의 회고
  - 지역의 높이 정보 파악

#99클럽 #코딩테스트준비 #개발자취업 #항해99 #TIL
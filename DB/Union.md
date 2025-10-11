# Union
- 유니온은 여러 Select 문을 결과를 합쳐서 보여주고자 할 때 사용하는 연산자이다.

## Union All vs Union
- 유니온 올은 중복 제거 없이 모든 값을 보여주고 유니온은 중복을 제거한 값만 보여준다.
- 중복을 제거하는 과정으로 인해 유니온은 유니온 올 보다 더 느리다.

### 사용법
- 월별 판매량 테이블을 통해 비교

# 온라인

| 월 | 제품 |
| :--- | :--- |
| 1월 | 노트북 |
| 2월 | 모니터 |
| 3월 | 키보드 |

# 오프라인

| 월 | 제품 |
| :--- | :--- |
| 3월 | 키보드 |
| 4월 | 마우스 |
| 5월 | 웹캠 |


- 중복 값 키보드가 있다.
```sql
SELECT 월, 제품 FROM Online_Sales
UNION
SELECT 월, 제품 FROM Offline_Sales;

SELECT 월, 제품 FROM Online_Sales
UNION ALL
SELECT 월, 제품 FROM Offline_Sales;
```
의 각각 값은 유니온의 경우 3월 키보드 컬럼이 하나, 유니온 올의 경우 둘이 나타난다. 
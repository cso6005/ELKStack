# fuzzy matches
- 허용한 만큼 오차를 허용한다.

~~~
{
    "query": {
        "fuzzy": {
            "title": {
                "value": "intrsteller",
                "fuzziness":2
            }
        }
    }
}
~~~

### levenshtenin edit distance 
- fuzzy matches 는 레벤쉬테인 편비 거리로부터 오차를 허용한다.
  - fuzziness parameter로 조정
- 얼마나 많은 오차를 정량화하는 것을 말한다.

<br>

1. substitutions
   - 대체 가능
   - 잘못된 문자를 입력했을 때 허용
2. insertions
   - 있으면 안되는 문자가 추가되었을 때 허용 
3. deletion
    - 문자가 누락되었을 때 허용


### auto fuzziness
문자열 길이에 따라 퍼지를 자동으로 설정하게 함.

문자열이 한 두 개뿐이라면 오타를 허용하지 않을 수 있고, 또 일정 비율 이상의 문자열이 잘못되면 오타가 더 이상 의미가 없어지기에 es 에서 내장된 fuzziness auto 설정이 있다.

- 1-2 character strings -> 편집거리 0
- 3-5 character strings -> 편집거리 1
- 다른 어떤 상황 -> 2 이하로 허용

# Item 45
## 스트림은 주의해서 사용하라

- 스트림 API를 잘활용하면 다재다능하여 사실상 어떠한 계산이라도 해낼 수 있음
- 하지만 할 수 있다는 뜻이지 그렇게 하라는 것은 아님
  - 스트림을 제대로 사용하면 프로그램이 짧고 깔끔해짐
  - 잘못 사용하면 읽기 어렵고 유지보수가 더 힘들어짐
 
### 왜 주의해서 사용해야 할까?

1. 스트림 없이 구현한 코드
```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		File dictionary = new File(args[0]); // 사전 파일
		int minGroupSize = Integer.parseInt(args[1]); // 사용자가 지정한 원소 수 문턱값

		// key : 알파벳 순으로 정렬한 값, value : 같은 키를 공유한 단어들을 담은 집합
		Map<String, Set<String>> groups = new HashMap<>();

		try (Scanner s = new Scanner(dictionary)) { //사전 파일에서 단어 읽음
			while (s.hasNext()) {
				String word = s.next();
				groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
			}
		}

		//minGroupSize 보다 원소 수가 많은 아나그램 그룹 출력
		for (Set<String> group : groups.values()) {
			if (group.size() >= minGroupSize) {
				System.out.println(group.size() + " : " + group);
			}
		}
	}

	// 아나그램 그룹의 key를 만들어줌
	private static String alphabetize(String s) {
		char[] a = s.toCharArray();
		Arrays.sort(a);
		return new String(a);
	}
}
```

2. 스트림을 과하게 사용한 코드
```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]); // 사전 파일 경로
		int minGroupSize = Integer.parseInt(args[1]); // 사용자가 지정한 원소 수 문턱값

		try (Stream<String> words = Files.lines(dictionary)) { // try-with-resources 문을 사용해 파일을 닫음
			// 사전을 여는 부분을 제외하고 프로그램 전체가 단 하나의 표현식으로 처리
			words.collect(
					groupingBy(word -> word.chars().sorted()
							.collect(StringBuilder::new,
									(sb, c) -> sb.append((char) c),
									StringBuilder::append).toString()))
					.values().stream()
					.filter(group -> group.size() >= minGroupSize) //
					.map(g -> g.size() + " : " + g)
					.forEach(System.out::println);
		}
	}
}

// 확실히 코드라인수가 짧지만 읽기가 어려움
// 스트림에 익숙하지 않다면 더더욱 어려움
```

3. 스트림을 적절하게 사용한 코드
```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]); // 사전 파일 경로
		int minGroupSize = Integer.parseInt(args[1]); // 사용자가 지정한 원소 수 문턱값

		try (Stream<String> words = Files.lines(dictionary)) { // try-with-resources 문을 사용해 파일을 닫음
			words.collect(groupingBy(Anagrams::alphabetize)) // alphabetize 메서드로 단어들을 그룹화함
					.values().stream()
					.filter(group -> group.size() >= minGroupSize) // 문턱값보다 작은 것을 걸러냄
					.forEach(group -> System.out.println(group.size() + " : " + group)); // 필터링이 끝난 리스트 출력
		}
	}

	// 아나그램 그룹의 key를 만들어줌
	private static String alphabetize(String s) {
		char[] a = s.toCharArray();
		Arrays.sort(a);
		return new String(a);
	}
}
```
- 스트림을 적절히 활용하면 깔끔하고 명료한 코드를 작성 가능
- 람다의 매개변수 이름을 잘 지어야 파이프라인의 가독성이 유지됨

#### 기존의 코드를 스트림을 사용해 리팩토링을 하되, 새 코드가 더 나아 보일때만 반영하자

### 코드블럭 vs 람다블럭
1. 코드블럭으로만 할 수 있는 일
   - 범위 안의 지역 변수를 읽고, 수정할 수 있음
     - 람다에서는 지역변수를 수정하는건 불가능함
   - 코드 블록에서는
     - return을 사용해 메서드에서 빠져나갈 수 있음
     - break나 continue를 사용하여 반복문을 종료하거나 건너뛸 수 있음
     - 메서드 선언에 명시된 검사 예외를 던질 수 있음
     - 람다로는 이 중 아무것도 할 수 없음
    
2. 스트림으로 처리하기에 맞춤인 일
   - 원소들을 일관되게 변환하기
   - 원소들을 필터링하기
   - 원소들을 하나의 연산을 사용해 결합하기
   - 원소들을 컬렉션에 모으기
   - 원소들중에 특정 조건을 만족하는 원소 찾기
  
3. 스트림으로 처리하기 어려운 일
   - 파이프라인 여러 단계에서 값들에 동시에 접근하기
     - 스트림 파이프라인은 일단 한 값을 매핑하고나면 원래의 값은 잃는 구조


# 핵심!

 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다.
그리고 수많은 작업들은 이 둘을 조합했을 때 가장 멋지게 해결된다.
어느쪽이 더 나은지 확실한 규칙은 없지만 참고할만한 지침들은 존재할 것이다. 하지만
스트림과 반복중 어느쪽이 더 나은지 확신하기 어렵다면 둘 다 해보고 더 나은쪽을 택하라!
```

## 회원서비스 개발 및 테스트

### 회원서비스

```java
package hello.hellospring.service;
import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import java.util.List;
import java.util.Optional;
public class MemberService {
private final MemberRepository memberRepository = new
MemoryMemberRepository();
/**
* 회원가입
*/
public Long join(Member member) {
validateDuplicateMember(member); //중복 회원 검증
memberRepository.save(member);
return member.getId();
}
private void validateDuplicateMember(Member member) {
memberRepository.findByName(member.getName())
.ifPresent(m -> {
throw new IllegalStateException("이미 존재하는 회원입니다.");
});
}
/**
* 전체 회원 조회
*/
public List<Member> findMembers() {
return memberRepository.findAll();
}
public Optional<Member> findOne(Long memberId) {
return memberRepository.findById(memberId);
}
}
```

### 회원 서비스 테스트

기존에는 회원 서비스가 메모리 회원 리포지토리를 직접 생성하게 했다.**
```java
public class MemberService {
private final MemberRepository memberRepository =
new MemoryMemberRepository();
}
```

*회원 리포지토리의 코드가
**회원 서비스 코드를 DI 가능하게 변경한다.**
```java
public class MemberService {
private final MemberRepository memberRepository;
public MemberService(MemberRepository memberRepository) {
this.memberRepository = memberRepository;
}
...
}
```
**회원 서비스 테스트**
```java
package hello.hellospring.service;
import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;
class MemberServiceTest {
MemberService memberService;
MemoryMemberRepository memberRepository;
@BeforeEach
public void beforeEach() {
memberRepository = new MemoryMemberRepository();
memberService = new MemberService(memberRepository);
}
@AfterEach
public void afterEach() {
memberRepository.clearStore();
}
@Test
public void 회원가입() throws Exception {
//Given
Member member = new Member();
member.setName("hello");
//When
Long saveId = memberService.join(member);
//Then
Member findMember = memberRepository.findById(saveId).get();
assertEquals(member.getName(), findMember.getName());
}
@Test
public void 중복_회원_예외() throws Exception {
//Given
Member member1 = new Member();
member1.setName("spring");
Member member2 = new Member();
member2.setName("spring");
//When
memberService.join(member1);
IllegalStateException e = assertThrows(IllegalStateException.class,
() -> memberService.join(member2));//예외가 발생해야 한다.
assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
}
}
```

`@BeforeEach` : 각 테스트 실행 전에 호출된다. 테스트가 서로 영향이 없도록 항상 새로운 객체를 생성하
고, 의존관계도 새로 맺어준다.

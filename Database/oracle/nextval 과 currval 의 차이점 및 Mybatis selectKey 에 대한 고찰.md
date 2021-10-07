# nextval 과 currval 의 차이점 및 Mybatis selectKey 에 대한 고찰

[OKKY ORACLE에서 시퀀스.currval 질문](https://okky.kr/article/348698)에 있는 예제를 살짝 바꿨다.

전제조건은 다음과 같다. 게시판에서 글쓰기와 첨부파일 등록을 하나의 서비스 메서드에서 처리중이고 트랜잭션이 걸려있는 상태이며, Mybatis 를 사용하고 있다.

이 예제에서 다음과 같은 고민을 할 수 있다.

- currval 을 쓰는게 맞는지, selectKey 를 쓰는게 맞는지에 대한 고민을 할 수 있다.

```java
@Transactional
public void create(Article article) {
  // 게시물 등록
  // 파일 등록
}
```
```xml
<insert id="create">
        insert into article
        values (article_seq.NEXTVAL, #{title}, #{content}, #{writer}, sysdate)
</insert>

<insert id="addAttach">    
        insert into file(fullname, article_seq) values (#{fullName}, article_seq.currval)
</insert>
```

파일을 등록할때 `currval`을 사용하게 되면 문제점은 없을까 ? nextval 과 currval 에 대해서 알아보자.

시퀀스는 `세션`의 영향을 받지 않는다. 하지만 currval 은 `세션`의 영향을 받는다. 이게 무슨 뜻이냐면, __currval 은 nextval 을 한 세션에서만 사용할 수 있다.__
연결이 서로 다른곳에서 currval 을 하더라도 세션이 다르기 때문에 같은 값을 갖지는 않는다.

즉, currval 은 __세션에 종속적__ 이라고 말할 수 있다.

시퀀스는 세션에 영향을 받지 않을 뿐더러, 트랜잭션에도 영향을 받지 않기 때문에 lock 을 걸 수 없다.

그러면 selecyKey 를 사용하는것보다 currval 을 사용하는게 더 낫지 않나? 라고 생각할 수도 있다. 혼자 개발한다면 selectKey 가 필요할 때만 사용하는게 더 나을 수 있다고 생각한다.
하지만 협업관점에서 볼 때는 조금 위험할 수 있다. 만약에 어떤 사람이 코드 수정을 잘못하여, aop 나 interceptor 안에 nextval 을 호출하는 코드를 또 집어넣어 코드 수정이 잘 못되었다면,
currval 을 사용하고 있던 경우라면 채번이 원하는대로 되지 않아 서비스에 문제가 생길 수 있다.

selectKey 에 대해서 잠시 살펴보겠다. 

selectKey 는 PK값을 미리(before) SQL을 통해서 처리해 두고 특정한 이름(keyProperty 에 지정한 이름)으로 보관을 하는데, 그 보관하는 과정에서 호출한 시퀀스 값을 Entity 객체에 저장시킨다.
그리고 insert 문을 실행할 때 객체에 저장된 시퀀스 값을 사용한다. 따라서 실제로 저장이 끝나고나서 생성된 시퀀스 값이 객체에 저장이 되었기 때문에 다음 로직에서 시퀀스가 필요한 경우 재사용 할 수 있다.

따라서, Mybatis 를 사용하고 Oracle 을 사용하고 있으면 selectKey 를 사용하여 시퀀스를 먼저 호출시킨다음 insert 구문을 한번 더 실행시키는 편이 더 낫다고 생각든다.

## References

- https://okky.kr/article/348698
- http://www.gurubee.net/article/61086
- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bluekisunny&logNo=120045145287
- https://developer-alle.tistory.com/203
- https://aljjabaegi.tistory.com/436
